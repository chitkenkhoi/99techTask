# Provide your CLI command here:

```bash
grep -E '"symbol": "TSLA".*"side": "sell"' ./transaction-log.txt | grep -o '"order_id": "[^"]*"' | cut -d'"' -f4 | paste -sd "," | xargs -I{} curl -s "https://example.com/api?order_ids={}" > ./output.txt
```
## My asumptions
1. The API support batch request
2. The syntax for batch request is:
   - https://example.com/api?order_ids=123,456
## Explanation for Batch Request Support

This command handles the case where the API supports batch requests, allowing us to fetch multiple order IDs in a single HTTP call. Here's how it works:

1. `grep -E '"symbol": "TSLA".*"side": "sell"' ./transaction-log.txt`
   - Searches the transaction log file for lines containing both "TSLA" symbol and "sell" side
2. `grep -o '"order_id": "[^"]*"'`
   - Extracts just the order_id portion from each matching line
3. `cut -d'"' -f4`
   - Splits each string by the double quote character and takes the 4th field ()
4. `paste -sd ","`
   - Combines all order IDs into a single comma-separated string (12346,12362)
   - This creates the proper format for a batch API request
5. `xargs -I{} curl -s "https://example.com/api?order_ids={}" > ./output.txt`
   - Executes a single curl request with all order IDs as a comma-separated parameter
   - Uses the query parameter format: ?order_ids=12346,12362
   - The -s flag makes curl silent (no progress or error messages displayed)
   - Redirects the API response to output.txt
