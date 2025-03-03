# Provide your CLI command here:

```bash
grep -E '"symbol": "TSLA".*"side": "sell"' ./transaction-log.txt | grep -o '"order_id": "[^"]*"' | cut -d'"' -f4 | xargs -I{} sh -c 'curl -s "https://example.com/api/{}" >> ./output.txt'
```
## My asumptions
1. No batch request support for the API
## Explanation for No Batch Request Support

This command handles the case where the API does not support batch requests, requiring us to make separate HTTP calls for each order ID. Here's how it works:

1. `grep -E '"symbol": "TSLA".*"side": "sell"' ./transaction-log.txt`
   - Searches the transaction log file for lines containing both "TSLA" symbol and "sell" side
2. `grep -o '"order_id": "[^"]*"'`
   - Extracts just the order_id portion from each matching line
3. `cut -d'"' -f4`
   - Splits each string by the double quote character and takes the 4th field
   - This extracts just the numeric order IDs (12346, 12362)
4. `xargs -I{} sh -c 'curl -s "https://example.com/api/{}" >> ./output.txt'`
   - For each order ID, runs a shell command that:
     - Makes an individual HTTP GET request to the API with the order ID in the path
     - Uses the RESTful endpoint format: /api/{order_id}
     - The -s flag makes curl silent (no progress or error messages displayed)
     - Appends (>>) each response to output.txt
   - This results in two separate API calls:
     - curl -s "https://example.com/api/12346"
     - curl -s "https://example.com/api/12362"

This approach makes individual requests one after another and combines all responses into a single output file.