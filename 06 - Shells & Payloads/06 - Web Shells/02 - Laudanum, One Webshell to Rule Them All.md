## Preparation and Modification

Identified file upload vulnerability and need language-specific webshells for ASP, ASPX, JSP, or PHP.

Copy template to working directory to avoid modifying original source

```
cp /usr/share/laudanum/<SERVICE_NAME>/shell.<SERVICE_NAME> <FILE_PATH>
```

**Dangerous / misconfigured settings**

- **allowedIps** variable: Must be updated with `<ATTACK_IP>` (e.g., line 59 in ASPX) or access to the shell will be blocked.

**Gotchas**

- **ASCII art and comments** within the shell files are frequently **signatured by AV** and should be stripped before upload.

---

## Execution and Access

Web application returns a successful upload path or directory for the injected file.

Execute commands through the browser interface

```
http://<TARGET_IP>/<FILE_PATH>
```

**Edge cases**

- Web applications using backslashes in paths (e.g., `\files\shell.aspx`) may require manual entry in the URL bar, though browsers typically normalize this to forward slashes.

**Gotchas**

- **Randomized filenames** or **hidden directories** can prevent navigation to the shell if the upload response does not explicitly reveal the final destination.

> ⚠️ Gap: Laudanum shells will fail to execute if the server-side environment lacks the specific language handler (e.g., uploading ASPX to a Linux/Apache server without Mono or similar).r