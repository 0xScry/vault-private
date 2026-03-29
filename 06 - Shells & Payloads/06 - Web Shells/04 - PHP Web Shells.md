### **Exploiting PHP Web Shells via File Upload Bypass**

#### **Methodology Overview**

PHP is a server-side scripting language that processes code on the server before sending output to the browser. When a web application like **rConfig 3.9.6** allows file uploads but fails to properly validate the file content or extension, attackers can upload a **PHP web shell** to execute commands on the underlying Linux host.

**When to Use:**

- When a target web application is confirmed to use PHP (e.g., presence of `.php` files like `login.php`).
- When an authenticated or unauthenticated **File Upload** feature is discovered (e.g., adding vendor logos).
- When client-side or basic server-side filters (like extension or `Content-Type` checks) are in place but can be intercepted.

---

#### **Target Configuration & Vulnerabilities**

|Vulnerable Component|Setting/Condition|Attack Implication|
|:--|:--|:--|
|**Authentication**|Default Credentials (`admin:admin`)|Initial access to administrative panels with upload privileges.|
|**Vendor Management**|File Upload (Logo)|Entry point for uploading malicious PHP scripts.|
|**Validation Logic**|`Content-Type` Header Check|The server trusts the `Content-Type` header provided by the client, allowing for proxy interception and bypass.|

---

#### **Operational Workflow: Uploading and Executing a Web Shell**

1. **Access the Administrative Panel**
    
    - Log in to the **rConfig** instance using the default credentials discovered in the source.
    - Navigate to the **Vendor Management** section to locate the upload functionality.
2. **Configure Interception Proxy**
    
    - Route browser traffic through **Burp Suite** to capture and modify the upload request before it reaches the server.

|Parameter|Value|
|:--|:--|
|**Proxy IP**|`127.0.0.1`|
|**Proxy Port**|`8080`|

3. **Prepare the Payload**
    
    - Use a pre-written PHP shell (e.g., WhiteWinterWolf’s) or create a custom `.php` file.
    - **Note:** Remove author comments or kudos from the script to avoid detection by security monitoring.
4. **Bypass File Type Restrictions**
    
    - Select the `.php` shell in the browser's upload field and click Save.
    - Intercept the **POST request** in Burp Suite.
    - Locate the `Content-Type` header and modify it to trick the server into accepting the file as an image.

|Original Header|Modified Header (Bypass)|
|:--|:--|
|`Content-type: application/x-php`|`Content-type: image/gif`|

5. **Forward and Verify**
    
    - Forward the modified request through Burp Suite.
    - If successful, the application will confirm the vendor was added, often displaying a broken image or default icon if the file isn't a valid image.
6. **Execute the Shell**
    
    - Navigate to the direct path where the uploaded file is stored on the server to trigger execution.

|Action|Command/URL|
|:--|:--|
|**Access Web Shell**|`http://<TARGET_IP>/images/vendor/connect.php`|

---

#### **Attack Implications & Post-Exploitation**

- **Command Execution:** Successful access provides a non-interactive shell in the browser, allowing for OS command execution.
- **Persistence & Cleanup:** To maintain access and remain stealthy, use the web shell to initiate a **reverse shell** to an attack machine, then delete the original PHP payload from the server.
- **Documentation:** Record the `sha1sum` or `MD5` hash of the payloads and exact upload locations for reporting and attribution.

---

#### **Operational Considerations**

- **Stealth:** Large payloads with comments are more likely to be flagged.
- **Environment:** Firefox version 90+ removed FTP support, which may affect alternative file transfer methods if needed during exploitation.
- **Failure Conditions:** If the server does not recognize the file as an image, it may default to a placeholder image, but the PHP code may still be executable if it was successfully written to the directory.