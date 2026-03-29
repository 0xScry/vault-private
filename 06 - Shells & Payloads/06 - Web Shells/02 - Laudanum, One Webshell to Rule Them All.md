### Laudanum Web Shell Methodology

**Laudanum** is a repository of pre-made injectable files designed to provide command execution or reverse shells across various web application languages, including **ASP, ASPX, JSP, and PHP**. It is utilized when a penetration tester identifies a file upload vulnerability and needs a reliable, language-specific shell to establish a foothold.

#### Repository Reference

|Location|Purpose|
|:--|:--|
|`/usr/share/laudanum`|Default directory for Laudanum files on Parrot OS and Kali Linux.|
|`/usr/share/laudanum/aspx/`|Directory for ASPX-specific shells.|

---

#### Operational Workflow: Deploying a Web Shell

1. **Stage the Payload**: Copy the desired shell from the read-only repository to a local working directory for modification.
    - `cp /usr/share/laudanum/<LANGUAGE>/<FILENAME> /home/<USERNAME>/<NEW_NAME>.aspx`
2. **Configure Access Control**: Edit the shell file to include your `<ATTACK_IP>` in the `allowedIps` variable. This ensures only the authorized attack machine can interact with the shell once it is live.
3. **Evade Detection**: Remove **ASCII art** and **comments** from the source code. These elements are frequently **signatured** by antivirus (AV) and can alert defenders to the presence of a payload.
4. **Upload to Target**: Utilize an identified **upload function** on the target web application to move the modified shell onto the server.
5. **Verify Execution Path**: Identify the directory where the file was saved. Some implementations may **randomize filenames** or restrict access to public directories, requiring careful observation of the server's response.
6. **Navigate to Shell**: Access the shell via a web browser. In certain Windows-based web environments, you may need to use a backslash (`\`) in the initial path (e.g., `<TARGET_IP>\<DIR>\<SHELL_NAME>`), which the browser will then normalize.
7. **Execute Commands**: Use the shell interface to issue system commands (e.g., `systeminfo`) to the host.

---

#### Attack Implications

- **Persistence/Access**: Successful deployment provides a persistent interface for command execution without requiring a constant reverse shell connection.
- **Information Gathering**: Allows for the execution of discovery commands to map the internal system and network.
- **Language Versatility**: The repository provides ready-made options for almost any common web environment, reducing the need for manual shell coding.

---

#### Security Misconfigurations

|Misconfiguration|Attack Impact|
|:--|:--|
|**Unrestricted File Upload**|Allows the attacker to upload and execute arbitrary code on the server.|
|**Publicly Accessible `/files/` Directory**|Simplifies the attack by providing a predictable path to execute uploaded payloads.|
|**Insecure Path Handling**|Allows access to uploaded files through non-standard URL formatting.|

---

#### Command Summary

| Action              | Command                                                                              |
| :------------------ | :----------------------------------------------------------------------------------- |
| **Copy Shell**      | `cp /usr/share/laudanum/<LANGUAGE>/<SHELL_FILE> /home/<USERNAME>/<TARGET_NAME>`      |
| **Edit Shell**      | `nano /home/<USERNAME>/<TARGET_NAME>` (To update `allowedIps` and remove signatures) |
| **Execute Command** | Enter command (e.g., `systeminfo`) into the shell's web UI                           |