## DNS Enumeration

Port 53 is open and initial service versions or default configurations are required.

Initial service and script scan

```
nmap -p53 -Pn -sV -sC <TARGET_IP>
```

Identify CNAME records for potential takeover

```
host <DOMAIN>
```

> ⚠️ Gap: Nmap scripts may fail to provide actionable data if **UDP/53** is filtered, requiring a forced TCP scan.

## DNS Zone Transfer

The DNS server lacks IP-based access controls on zone transfers, allowing a full dump of the namespace via TCP/53.

Standard query for full zone dump

```
dig AXFR @<TARGET_IP> <DOMAIN>
```

Automated nameserver discovery and transfer attempt

```
fierce --domain <DOMAIN>
```

- **Tool comparison**
    - **dig** -> `dig AXFR @<TARGET_IP> <DOMAIN>` -> prefer for manual, targeted verification of a known vulnerable server.
    - **fierce** -> `fierce --domain <DOMAIN>` -> prefer for automated enumeration of all subdomains and nameservers in a root domain.

**Misconfigured DNS servers** often allow **unauthenticated AXFR requests** from any IP address.

**UDP communication fails** when the DNS response packet is too large, forcing a fallback to **TCP/53**.

## Subdomain Enumeration

The attack surface needs expansion to identify hidden services or targets for takeover.

Passive scraping from open sources

```
./subfinder -d <DOMAIN> -v
```

Pure DNS brute-forcing using specific resolvers

```
./subbrute.py <DOMAIN> -s <FILE_PATH> -r <FILE_PATH>
```

- **Tool comparison**
    - **Subfinder** -> `./subfinder -d <DOMAIN> -v` -> prefer for external reconnaissance using public APIs.
    - **Subbrute** -> `./subbrute.py <DOMAIN> -s <FILE_PATH> -r <FILE_PATH>` -> prefer for **internal penetration tests** on hosts without internet access.

Internal pivots often require uploading tools via USB or working from a compromised host to reach **internal DNS configurations**.

## Subdomain Takeover

A CNAME record points to an expired or non-existent third-party service, such as an AWS S3 bucket.

Verification of AWS S3 bucket vacancy

```
host <DOMAIN>
```

**CNAME records remain active** even after the destination domain expires, allowing an attacker to claim the external resource.

A **NoSuchBucket** error on a live subdomain indicates the resource is claimable.

## Local DNS Spoofing

Positioned on a local network with the ability to perform MITM and redirect traffic to a malicious host.

1. Map the target domain to the attack machine

```
echo "<DOMAIN> A <ATTACK_IP>" >> /etc/ettercap/etter.dns
```

2. Start the spoofing plugin after setting Target1 (victim) and Target2 (gateway) in the interface.

**Active MITM** is a prerequisite; the `dns_spoof` plugin will only respond to intercepted queries.