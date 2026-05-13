
## Intercepting Responses

Client-side restrictions like **numeric-only** input or **maxlength** limits prevent payload delivery or hidden fields need to be exposed.

Enable response interception to catch the server's reply before the browser renders it

```
Proxy > Proxy settings > Response interception rules > Intercept Response
```

Catch the immediate next response after a request

```
Step
```

- Burp Suite
    - `Proxy > Proxy settings > Intercept Response`
    - Prefer for granular control over which responses are caught based on rules.
- ZAP
    - **Step** button
    - Prefer for quick, one-off response modification during an active intercept.

**Gotchas** **Browser caching** prevents the proxy from seeing the response; force a full refresh using `CTRL+SHIFT+R` to ensure the server sends a new response.

---

## Automated Response Modification

Repeated manual modification of HTML elements is required across multiple pages or hidden fields must be visible globally.

Unhide all hidden form fields automatically

```
Proxy > Proxy settings > Response modification rules > Unhide hidden form fields
```

Enable the feature to show HTML comments as UI indicators

```
+ button > Comments
```

- Burp Suite
    - `Response modification rules`
    - Prefer for persistent, background modification of all incoming traffic.
- ZAP HUD
    - **Show/Enable** (light bulb icon)
    - Prefer for instant UI updates without requiring a page refresh or manual rule configuration.

**Gotchas** **Manual intercept** is still required for custom logic changes; automated rules only handle pre-defined patterns like unhiding or enabling fields.