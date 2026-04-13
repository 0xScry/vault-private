# LLMNR/NBT-NS Poisoning (Linux)

## Overview

**LLMNR** (Link-Local Multicast Name Resolution) and **NBT-NS** (NetBIOS Name Service) are Windows components used for host identification when **DNS resolution fails**. In a **Man-in-the-Middle (MITM)** attack, an attacker spoofs an authoritative source by responding to these broadcast requests, tricking victims into authenticating with the attack machine.

### Why Use This Technique

- **Goal:** To capture **NTLMv1 or NTLMv2 password hashes** sent over the network.
- **Foothold:** Successful cracking of these hashes provides **cleartext credentials**, granting a credentialed standpoint for further domain enumeration,.
- **Scenario:** Effective when starting from an **anonymous position** on an internal network using a Linux attack host.

### Protocol Reference

|Protocol|Port|Description|
|:--|:--|:--|
|**LLMNR**|UDP 5355|Based on DNS format; allows local link name resolution.|
|**NBT-NS**|UDP 137|Identifies systems on a local network by NetBIOS name.|

---

## Tooling: Responder

**Responder** is a purpose-built Python tool used to poison LLMNR, NBT-NS, and MDNS requests. It includes rogue authentication servers (e.g., SMB, HTTP, WPAD) to capture hashes,.

### Operational Requirements

- **Privileges:** Must run with **sudo** or as **root**.
- **Port Availability:** The following ports must be open on the attack host for full functionality: UDP 137, 138, 53, 5355, 5353; TCP 389, 1433, 80, 135, 139, 445, 21, 3141, 25, 110, 587, 3128.

### Parameter Reference

|Flag|Description|Use Case|
|:--|:--|:--|
|`-I <INTERFACE>`|Specify interface|**Mandatory**; sets the network interface to listen on,.|
|`-A`|Analyze mode|**Passive** recon; listens for requests without poisoning,.|
|`-w`|Start WPAD proxy|Captures HTTP requests from browsers with "Auto-detect settings",.|
|`-f`|Fingerprint|Attempts to identify the OS and version of the requesting host,.|
|`-v`|Verbose|Increases output; prints every hash to the console,.|
|`-F` / `-P`|Force Auth|Forces NTLM/Basic prompts; use sparingly as it may alert users,.|

---

## Operational Workflow

### 1. Passive Enumeration

The goal is to identify active LLMNR/NBT-NS traffic in the environment without sending poisoned packets ("fly on the wall" approach),.

```
sudo responder -I <INTERFACE> -A
```

### 2. Active Poisoning and Capture

Switch to active mode to answer requests. It is recommended to run this in a **tmux** window for an extended period to maximize hash collection.

```
sudo responder -I <INTERFACE> -wf
```

### 3. Hash Management

Responder automatically saves captured hashes to: `/usr/share/responder/logs`.

- **File Format:** `(MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt`.
- **Reviewing Logs:**

```
ls /usr/share/responder/logs
```

### 4. Offline Password Cracking

NetNTLMv2 hashes **cannot** be used for Pass-the-Hash; they must be cracked offline. Success depends on the target's password complexity policy.

```
hashcat -m 5600 <HASH_FILE> /usr/share/wordlists/rockyou.txt
```

- **Note:** Mode `5600` is for **NetNTLMv2**. If an NTLMv1 hash is captured, consult Hashcat documentation for the correct mode.

---

## Attack Implications

- **Attack Unlocks:** Access to domain user accounts for further enumeration or lateral movement,.
- **Lateral Movement:** Captured hashes can potentially be used for **SMB Relay** attacks (if SMB signing is disabled) to gain administrative access without cracking,.

## Critical Misconfigurations

|Misconfiguration|Impact|
|:--|:--|
|**Lack of SMB Signing**|Allows captured authentication requests to be **relayed** to other hosts.|
|**Weak Password Policy**|Enables rapid offline cracking of captured NetNTLMv2 hashes (e.g., 8-character passwords).|
|**WPAD Auto-detect**|Allows Responder to capture HTTP/browser traffic via the rogue WPAD proxy.|