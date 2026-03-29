### **SMTP Protocol Overview**

The **Simple Mail Transfer Protocol (SMTP)** is used for sending emails across IP networks, either between a client and a server or between two servers. By default, it operates on **TCP port 25**. Newer implementations often use **TCP port 587** for authenticated submissions using **STARTTLS** to upgrade a plaintext connection to an encrypted one.

|Component|Description|
|:--|:--|
|**Mail User Agent (MUA)**|The client/program used to send and receive emails.|
|**Mail Submission Agent (MSA)**|A relay server that checks the validity and origin of the email to relieve the MTA.|
|**Mail Transfer Agent (MTA)**|Software responsible for sending, receiving, and storing emails; it searches DNS for recipient IP addresses.|
|**Mail Delivery Agent (MDA)**|Transfers the email from the destination SMTP server to the recipient's mailbox.|

---

### **Service Footprinting**

**Footprinting** is used to identify the version of the SMTP service and determine which commands the server supports, which informs further enumeration or exploitation steps.

#### **1. Automated Enumeration**

Use **Nmap** to identify the service version and supported SMTP extensions.

```
sudo nmap <TARGET_IP> -sC -sV -p25
```

- **Why it matters:** The `smtp-commands` script uses `EHLO` to list capabilities like `VRFY`, `STARTTLS`, or `AUTH PLAIN`.

#### **2. Open Relay Testing**

An **Open Relay** occurs when a server is misconfigured to allow any IP address to send mail through it, facilitating spam or email spoofing.

```
sudo nmap <TARGET_IP> -p25 --script smtp-open-relay -v
```

- **Attack Implication:** If identified as an open relay, an attacker can **spoof emails** or potentially read communications between parties.

---

### **User Enumeration**

**Enumeration** is performed to identify valid system users through the SMTP service.

#### **Technique: VRFY Command**

The `VRFY` command checks if a specific mailbox exists.

```
telnet <TARGET_IP> 25
VRFY <USERNAME>
```

|Response|Implication|
|:--|:--|
|**250**|User exists/is valid.|
|**252**|The server confirms existence but may be a **false positive** due to configuration.|

- **Edge Case:** Do not rely solely on automated tools for user enumeration; administrators often configure servers to issue code **252** for all queries, regardless of whether the user exists.

---

### **Operational Workflow: Sending a Manual Email**

Use this workflow to manually interact with the SMTP server via **Telnet** to test for delivery or spoofing capabilities.

1. **Connect** to the target SMTP service.
    
    ```
    telnet <TARGET_IP> 25
    ```
    
2. **Initiate the session** using `HELO` or `EHLO` (Extended SMTP).
    
    ```
    EHLO <DOMAIN>
    ```
    
3. **Specify the sender** address.
    
    ```
    MAIL FROM: <USERNAME>@<DOMAIN>
    ```
    
4. **Specify the recipient** address.
    
    ```
    RCPT TO: <USERNAME>@<DOMAIN>
    ```
    
5. **Initiate data transmission** for the email body.
    
    ```
    DATA
    ```
    
6. **Input email content** (Header + Body). End the message with a single dot `.` on its own line.
    
    ```
    From: <USERNAME>@<DOMAIN>
    To: <USERNAME>@<DOMAIN>
    Subject: <SUBJECT>
    
    <MESSAGE_BODY>
    .
    ```
    
7. **Terminate** the session.
    
    ```
    QUIT
    ```
    

---

### **SMTP Command Reference**

|Command|Goal|
|:--|:--|
|**AUTH PLAIN**|Authenticates the client using a service extension.|
|**HELO / EHLO**|Identifies the client computer name and starts the session.|
|**MAIL FROM**|Identifies the sender of the email.|
|**RCPT TO**|Identifies the recipient of the email.|
|**DATA**|Starts the transmission of the email content.|
|**RSET**|Aborts current email transmission but keeps the connection open.|
|**EXPN**|Checks if a mailbox is available for messaging.|
|**NOOP**|Prevents connection timeout by requesting a server response.|

---

### **Dangerous Configurations**

Misconfigurations in the Postfix configuration file (`/etc/postfix/main.cf`) lead to security vulnerabilities.

|Setting|Risk|Attack Path|
|:--|:--|:--|
|`mynetworks = 0.0.0.0/0`|**Open Relay**|Allows any external user to send/spoof emails through the server.|
|**Unencrypted Connections**|**Credential Theft**|Default SMTP transmits authentication (`AUTH PLAIN`) in plaintext unless `STARTTLS` is enforced.|

**Scenario Context:** If inbound connections to a pivot are blocked, you can use a **Web Proxy** to connect to the SMTP server using the `CONNECT <TARGET_IP>:25 HTTP/1.0` command.