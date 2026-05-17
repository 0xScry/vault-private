## Intercepting Command-Line Traffic

### Proxychains

Visibility into web requests from CLI tools or thick clients is required for manual manipulation or debugging.

Modify the configuration file to route traffic to the local listener port:

```
# Edit /etc/proxychains.conf
# Comment out default socks4 127.0.0.1 9050
http 127.0.0.1 8080
```

Execute the target command in quiet mode to suppress connection metadata and reduce terminal clutter:

```
proxychains -q curl http://<TARGET_IP>:<PORT>
```

**Gotchas** **Performance degradation** — Proxying traffic significantly slows tool execution; utilize only for investigation and not for standard tool usage.

---

## Intercepting Metasploit Traffic

Metasploit module behavior or scanner output requires inspection through proxy history or interception.

Define the proxy variable for the active auxiliary or exploit module:

```
set PROXIES HTTP:127.0.0.1:8080
```

**Gotchas** **Manual investigation required** — Every tool utilizes different methods for proxy configuration; verify individual tool settings if a global proxy wrapper is not utilized.