1. Scan TCP 111 and 2049 to identify the NFS version and active RPC services.
2. Enumerate available shares (exports) to identify targetable paths and subnet-based access controls.
3. Analyze share permissions and file lists using NSE scripts to find sensitive files like SSH keys before mounting.
4. Mount target exports locally to inspect UIDs/GIDs and content.
5. Match local UIDs to server-side UIDs to bypass permission sets on files not owned by root.
6. Check for `no_root_squash` to enable root-level writes and SUID-based escalation.

---

## Service Discovery

Identify if NFS is active and which version (v2, v3, or v4) is running. v2 and v3 rely on portmapper (111) while v4 can run solely on 2049.

Identify RPC services and their associated ports

```
sudo nmap <TARGET_IP> -p111,2049 -sV -sC
```

Detailed enumeration of share contents, statistics, and export lists

```
sudo nmap --script nfs* <TARGET_IP> -sV -p111,2049
```

- **Dangerous / misconfigured settings**
    
    - `insecure`: Allows clients to use ports above 1024, bypassing the requirement for root-only source ports.
- **Gotchas**
    
    - **Firewalls** may block the various ports used by NFSv3 (mountd, nlockmgr) even if 111 and 2049 are open; NFSv4 is more firewall-friendly using only 2049.

## Export Enumeration

Identify which physical filesystems are exposed and which subnets are allowed to access them.

List exported shares from the target

```
showmount -e <TARGET_IP>
```

- **Tool comparison**
    
    - `showmount` -> `showmount -e <TARGET_IP>` -> Quick check for export list.
    - Nmap (`nfs-showmount`) -> `nmap --script nfs-showmount <TARGET_IP>` -> Use when standard RPC calls are restricted or as part of a broader scan.
- **Dangerous / misconfigured settings**
    
    - Subnet masks in `/etc/exports` that are too broad, allowing access from untrusted segments.

## Share Access and Manipulation

Access the filesystem locally to interact with files using spoofed identities.

Mount an export to a local directory

```
sudo mount -t nfs <TARGET_IP>:<FILE_PATH> ./<SHARE_NAME> -o nolock
```

List files with numeric UIDs/GIDs to identify which local IDs need to be created/spoofed

```
ls -n <FILE_PATH>
```

Unmount the share after data collection

```
sudo umount ./<SHARE_NAME>
```

> ⚠️ Gap: The source mentions "adapting" local UIDs/GIDs to match the server but does not provide the `usermod` or `groupmod` commands required to perform this identity spoofing locally.

- **Edge cases**
    
    - If the server and client have different UID/GID mappings, the server does not perform extra validation; it trusts the UID provided by the client (NFSv2/v3).
- **Gotchas**
    
    - **root_squash** prevents the local root user from accessing or editing files owned by the remote root by mapping them to an anonymous user.

## Privilege Escalation via NFS

Leverage write access and misconfigurations to elevate privileges on the target system.

Check for dangerous settings in the exported options

```
cat /etc/exports
```

- **Dangerous / misconfigured settings**
    
    - `rw`: Allows modification of files; critical for uploading shells or modifying configs.
    - `no_root_squash`: Files created by the local root user retain root ownership (UID 0) on the server, allowing the creation of SUID binaries.
    - `nohide`: Automatically exports nested file systems if the parent is exported.
- **Gotchas**
    
    - **SUID shells** uploaded to an NFS share must be executed by a user with access to the target system (e.g., via SSH) to trigger the escalation.