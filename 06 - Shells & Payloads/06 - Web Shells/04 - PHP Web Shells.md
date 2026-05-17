## Methodology

1. Identify PHP backend through file extensions or server headers.
2. Locate authenticated file upload fields in administrative menus.
3. Capture the upload request and test for **Content-Type** validation bypass.
4. Locate the uploaded payload in predictable directories to trigger execution.
5. Upgrade the non-interactive web shell to a reverse shell for stability.
6. Delete the uploaded payload immediately to avoid detection.

---

## Bypassing File Type Restrictions

Application allows image uploads but rejects `.php` files. Required when the server performs client-side or basic header-based validation.

Modify the request header to mimic a valid image type

```
Content-Type: image/gif
```

> ⚠️ Gap: The source assumes the upload directory is known or guessable; if the application randomizes filenames or hides the upload path, execution will fail.

**Failure to remove author comments** from the payload source code can trigger signature-based detection or provide attribution to the attacker.

## Web Shell Execution

File upload successful but code does not execute automatically. Requires navigating to the static resource path.

Access the uploaded shell via the browser or CLI

```
curl http://<TARGET_IP>/images/vendor/<FILE_NAME>.php
```

**Leaving the web shell on disk** after the engagement period or during a stealth assessment leads to discovery and potential attribution.

**Direct browser execution** often results in a non-interactive shell, limiting complex command execution.