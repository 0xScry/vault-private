## Web Proxy Execution

Intercepting and modifying HTTP/S traffic on ports 80/443 between application and back-end.

Launch Burp Suite directly from terminal

```
burpsuite
```

Launch OWASP ZAP directly from terminal

```
zaproxy
```

Execute either tool via JAR file

```
java -jar <FILE_PATH>
```

- Tool comparison
    
    - Burp Suite -> `burpsuite` -> Standard for corporate/advanced pentesting; includes built-in Chromium for testing.
    - OWASP ZAP -> `zaproxy` -> Open-source; use to bypass **throttling** or paid-tier limitations on automated scans.
- Gotchas **Temporary projects** in Burp Community Edition purge all data upon closing the application.
    

> ⚠️ Gap: Interception fails if the browser/application is not manually configured to route traffic through the proxy listener address.

---

## Project Persistence

Pentesting large-scale applications or running long-duration scans where session state and progress must be saved.

- Tool comparison
    
    - Burp Suite -> New project on disk -> Pro/Enterprise version only; required for saving project state to disk.
    - OWASP ZAP -> Persist session -> Available in free version; use when session data must survive restarts or persist for days.
- Gotchas **Disk-based projects** are unsupported in Burp Community, forcing a "Temporary project" selection that loses all configuration and captured data.