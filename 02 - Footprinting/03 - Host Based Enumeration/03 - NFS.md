# NFS (Network File System)

**NFS** is used to access file systems over a network as if they were local, primarily between Linux and Unix systems. It relies on the **ONC-RPC/SUN-RPC** protocol (port 111) for system-independent data exchange.

### NFS Versions

|Version|Key Features|
|:--|:--|
|**NFSv2**|Older; operates entirely over UDP.|
|**NFSv3**|Supports variable file sizes and better error reporting; authenticates the client computer but not the user.|
|**NFSv4**|Stateful protocol; includes Kerberos authentication, supports ACLs, and works through firewalls using only port **2049** (TCP/UDP).|

---

## Footprinting and Enumeration

### 1. Identify Running RPC Services

Use **Nmap** to identify if NFS and the RPC portmapper are active.

```
sudo nmap <TARGET_IP> -p111,2049 -sV -sC
```

- **Goal:** Confirm the presence of `rpcbind` (port 111) and `nfs` (port 2049).
- **Why it matters:** The `rpcinfo` NSE script will list all running RPC services, their ports, and descriptions, confirming if the target share is reachable.

### 2. Enumerate Shares and Stats

Execute NFS-specific NSE scripts to retrieve volume information and file listings.

```
sudo nmap --script nfs* <TARGET_IP> -sV -p111,2049
```

- **Goal:** View the contents of the share, permissions, UIDs/GIDs, and filesystem statistics.
- **Attack Implication:** This may reveal sensitive files (e.g., `id_rsa`) and the specific subnets allowed to access the share.

### 3. List Exported File Systems

Query the mount daemon directly for a list of exports.

```
showmount -e <TARGET_IP>
```

- **Goal:** Identify which directories are exported and which hosts/subnets have access.

---

## Operational Workflow: Access and Manipulation

### 1. Mount the Remote Share

Create a local directory and mount the target NFS share to interact with it like a local filesystem.

```
mkdir <LOCAL_MOUNT_DIR>
sudo mount -t nfs <TARGET_IP>:/ <LOCAL_MOUNT_DIR> -o nolock
```

- **Scenario:** Use when you have identified an accessible share via `showmount` or Nmap.
- **Note:** The `-o nolock` option is often used to disable file locking.

### 2. Analyze File Ownership and Permissions

Determine the UIDs and GIDs required to access or modify files.

```
# List contents with usernames/groups
ls -l <LOCAL_MOUNT_DIR>

# List contents with numeric UIDs/GIDs
ls -n <LOCAL_MOUNT_DIR>
```

- **Why it matters:** NFS authorization is derived from file system information (UID/GID). The server often trusts the client's provided UID/GID.
- **Technique:** If a file is owned by UID `1000`, you can create a local user with UID `1000` on your attack machine to gain that user's permissions on the share.

### 3. Privilege Escalation via SUID

If you have SSH access to the target and write access to an NFS share:

1. Upload a shell/binary to the NFS share.
2. Set the **SUID** bit for a specific user.
3. Execute the binary via your SSH session to escalate privileges to that user.

### 4. Unmount the Share

Clean up the environment once tasks are complete.

```
sudo umount <LOCAL_MOUNT_DIR>
```

---

## Configuration Reference

### Dangerous Settings in `/etc/exports`

These settings significantly increase the attack surface or allow for administrative compromise.

|Option|Security Implication|
|:--|:--|
|**`rw`**|Allows **Read and Write** permissions on the share.|
|**`insecure`**|Allows clients to use ports above 1024. Only root can use ports below 1024, so this permits non-root users to interact with the service.|
|**`no_root_squash`**|**Critical:** Files created by root on the client retain UID/GID 0 on the server. This allows a root user on the attack machine to have root privileges on the share.|
|**`nohide`**|Automatically exports directories mounted below an exported directory.|

### Standard Configuration Options

|Option|Description|
|:--|:--|
|**`ro`**|Read-only permissions.|
|**`sync`**|Synchronous data transfer; slower but more reliable.|
|**`async`**|Asynchronous data transfer; faster performance.|
|**`root_squash`**|**Default/Secure:** Maps root UID/GID (0) to an anonymous ID, preventing root access from the client to the share.|