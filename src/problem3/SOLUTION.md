# Memory Troubleshooting on NGINX Load Balancer VM

## Initial Investigation Steps

### 1. Verify Actual Memory Usage

First, I'd confirm the actual memory usage to ensure we're not looking at cached memory (which Linux can reclaim):

```bash
free -h
```

This will show us total memory, used memory, free memory, and cached memory. The key metric is "available" memory - if this is genuinely low, we have a real problem.

I'd also check memory usage trends:

```bash
vmstat 1 10
```

### 2. Identify Memory-Consuming Processes

Next, I'd determine which processes are using memory:

```bash
ps aux --sort=-%mem | head -15
```

Or using interactive tools:

```bash
htop
```

This will show if NGINX is the culprit or if there are unexpected processes.

## Likely Root Causes & Solutions

### 1. NGINX Configuration Issues

**Possible symptoms:**
- Multiple NGINX worker processes consuming large amounts of memory
- Memory usage that doesn't decrease after traffic drops

**Investigation:**
```bash
grep worker_processes /etc/nginx/nginx.conf
grep -r "buffer" /etc/nginx/
grep -r "worker_connections" /etc/nginx/
```

**Impact:** Excessive workers or oversized buffers can consume all available memory, leading to degraded performance or OOM kills.

**Recovery steps:**
- Reduce `worker_processes` to match CPU cores
- Adjust buffer sizes to reasonable values
- Reload NGINX: `systemctl reload nginx`

### 2. Memory Leak in NGINX

**Possible symptoms:**
- Memory usage gradually increasing over time without corresponding traffic increase
- Memory not released after NGINX service restart

**Investigation:**
```bash
dmesg | grep -i oom
journalctl -u nginx
```

**Impact:** Gradual memory exhaustion eventually leading to system instability.

**Recovery steps:**
- Temporary fix: `systemctl restart nginx`
- Long-term: Update NGINX to latest stable version
- Check for problematic modules or third-party add-ons

### 3. Excessive Connections

**Possible symptoms:**
- High number of established connections
- Normal per-connection memory usage, but multiplied by thousands

**Investigation:**
```bash
ss -s
netstat -tunapl | grep ESTABLISHED | wc -l
```

**Impact:** Even modest per-connection memory allocations can add up when connection counts are very high.

**Recovery steps:**
- Implement connection limits in NGINX
- Add request rate limiting
- Check for possible DDoS or misconfigured clients

### 4. Kernel Memory Issues

**Possible symptoms:**
- High memory usage not attributable to user processes

**Investigation:**
```bash
slabtop
cat /proc/meminfo | grep Slab
```

**Impact:** Kernel memory issues can be difficult to diagnose and may require system restart.

**Recovery steps:**
- Reboot VM (short-term)
- Update kernel packages
- Check for problematic kernel modules

### 5. Hidden/Malicious Processes

**Possible symptoms:**
- Unexpected processes using memory
- Processes restarting automatically when killed

**Investigation:**
```bash
lsof -i
ps auxf
```

**Impact:** Resource consumption and potential security breach.

**Recovery steps:**
- Identify and terminate unwanted processes
- Check startup services and cron jobs
- Perform security audit if compromise suspected

## Preventive Measures

1. **Implement Memory Limits**
   ```bash
   # In systemd service file
   MemoryMax=2G
   ```

2. **Optimize NGINX Configuration**
   ```nginx
   worker_processes auto;
   worker_connections 1024;
   
   # Keep connection buffer sizes reasonable
   proxy_buffer_size 4k;
   proxy_buffers 8 8k;
   ```

3. **Monitoring Improvements**
   - Set up alerts for memory trends, not just thresholds
   - Monitor connection counts and request rates
   - Track NGINX worker memory over time

4. **Regular Maintenance**
   - Schedule periodic NGINX restarts if memory leaks suspected
   - Keep system and NGINX updated
   - Perform regular config audits
