## Request Interception

Traffic needs to be analyzed or modified mid-flight while the browser is already proxied to the tool.

Enable request interception in Burp Suite

```
Proxy > Intercept > Intercept is on
```

Forward the currently held request to the server in Burp

```
Forward
```

Toggle request interception in ZAP using global shortcut

```
CTRL+B
```

Forward a single request and intercept the corresponding response in ZAP

```
Step
```

Enable browser-integrated controls via ZAP HUD

```
HUD Button (End of top menu bar)
```

Toggle interception directly within the browser using ZAP HUD

```
Second button from top (Left pane)
```

- Tool comparison
    
    - Burp Suite -> `Proxy > Intercept` -> standard for manual request-by-request analysis.
    - ZAP -> `Step` button -> prefer when examining every individual step of page functionality.
    - ZAP -> `Continue` button -> prefer when only interested in a single target request while allowing others to pass.
- Edge cases
    
    - Use ZAP HUD when in-browser controls are preferred over switching windows, provided the browser version is compatible.
- Gotchas
    
    - **Firefox background traffic** may trigger interceptions for unrelated requests; forward until `<TARGET_IP>` is visible in the header.
    - **ZAP HUD instability** occurs in specific browser versions and may not function as intended.

## Request Manipulation

Front-end JavaScript or HTML attributes prevent input of specific characters or strings in the browser.

Inject command into an intercepted parameter to test for backend execution

```
ip=;ls;
```

- Gotchas
    - **Backend validation** may still exist; bypassing front-end restrictions only confirms a lack of client-side filtering, not a successful exploit.
    - **Request hanging** occurs if a request is intercepted and neither forwarded nor dropped, stalling the application flow.