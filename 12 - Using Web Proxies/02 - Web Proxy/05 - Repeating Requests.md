## Request History Analysis

Iterating on previous traffic or locating asynchronous updates; requires requests to have passed through the tool proxy.

Burp navigation for history review `Proxy > HTTP History`

ZAP navigation for history review `History tab`

- **Tool comparison**
    
    - Burp -> `Original Request / Edited Request` toggle -> prefer when verifying how proxy rules or manual intercepts altered the raw traffic.
    - ZAP -> `History` pane -> prefer for a unified view of final sent requests.
- **Edge cases**
    
    - **WebSockets history** must be checked specifically for asynchronous data fetching or background app updates.
- **Gotchas**
    
    - **Loss of original request data** in ZAP history as it only records the final modified version sent to the server.

## Manual Request Repeating

System enumeration or command injection testing; requires a baseline request to modify without re-intercepting traffic.

Send request to Burp Repeater `CTRL+R`

Move focus to Burp Repeater tab `CTRL+SHIFT+R`

Open ZAP Request Editor for manual modification `Right-click > Open/Resend with Request Editor`

Render the modified response directly in the ZAP HUD browser `Replay in Browser`

- **Tool comparison**
    
    - Burp -> `Right-click > Change Request Method` -> prefer for instantly pivoting between GET and POST without rewriting headers.
    - ZAP -> `Method drop-down` -> prefer when testing specific or non-standard HTTP methods via the UI.
- **Gotchas**
    
    - **Payload failure** if manual input is not **URL-encoded** when required by the application logic.
    - **Redundant steps** if failing to use **Request Repeating** for tasks requiring 5-6 steps per command execution.