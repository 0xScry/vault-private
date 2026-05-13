## Windows Netsh Port Forwarding

Compromised Windows host has reachability to internal targets and requires a local listener to tunnel traffic from the attack host.

Establish a v4 to v4 port proxy to map a local pivot port to an internal target service:

```
netsh.exe interface portproxy add v4tov4 listenport=<PORT> listenaddress=<PIVOT_IP> connectport=<PORT> connectaddress=<TARGET_IP>
```

Display the active port forwarding table to verify mapping:

```
netsh.exe interface portproxy show v4tov4
```

Connect to the internal target via the pivot listener from the attack host:

```
xfreerdp /v:<PIVOT_IP>:<PORT> /u:<USERNAME> /p:<PASSWORD>
```

> ⚠️ Gap: Configuring `portproxy` requires **administrative privileges** on the pivot host.

**Host-based firewall rules** on the pivot system may silently drop incoming traffic to the `<PORT>` even if the proxy is correctly configured.