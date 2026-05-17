## Fuzzer Configuration

Web discovery when Burp Community throttling is too slow or wordlists are unavailable. Requires a base request captured in proxy history.

Open the fuzzer from a proxy request `Right-click request > Attack > Fuzz`

Set the payload position `Fuzz Locations > Highlight <TARGET_STRING> > Add`

Select built-in wordlists `Payloads > Add > Type: File Fuzzers > dirbuster > directory-list-1.0.txt`

Prevent request breakage from special characters `Processors > Add > URL Encode`

Configure execution speed `Options > Concurrent Scanning Threads per Scan > 20`

Define payload rotation strategy `Options > Strategy > Depth First`

Tool comparison:

- ZAP Fuzzer -> `Attack > Fuzz` -> No speed throttling; prefer over Burp Intruder Community for large wordlists.
- Burp Intruder -> `Send to Intruder` -> Advanced payload logic; prefer when ZAP's 8 basic payload types are insufficient.

Gotchas:

- **High thread counts** can crash the service or trigger rate limits.
- **Missing URL encoding** causes 500 errors if the wordlist contains reserved characters.

---

## Result Analysis

Fuzzing run complete. Identifying successful hits from the results table.

Filter results by successful status code `Sort by Response > 200`

Edge cases:

- Use **RTT** to identify time-based vulnerabilities like SQL injection.
- Use **Size Resp. Body** to find valid pages that return 200 OK but differ from the baseline response.

Gotchas:

- **Filtering only for 200 OK** misses vulnerabilities detectable only via timing or response size shifts.
