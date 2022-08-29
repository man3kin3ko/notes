---
title: FOSS Network Infrastructure and Security
created: '2022-08-29T13:43:51.577Z'
modified: '2022-08-29T14:53:51.095Z'
---

# FOSS Network Infrastructure and Security
## DNS
Domain names are implementation of DNS **name space**. Each name space is reflected by its **zone file** or **administrative name space**. Servers that hosts zone files are authoritative that can be **master** or **slave** server. Each time the master server is updated, it can automatically send notification to slaves. Servers that doesn't handle their own zone file are non-authoritative and usually perform caching to increase the network performance. **Caching nameservers should never perform authoritative functions to prevent cache poisoning attacks**. 

Below the root server are the **Country Code Top Level Domain (ccTLD)** and **Global Top Level Domain (gTLD)**. 
> **ccTLD** domains defined in ISO 3166 standard. It always two letter codes like `.uk` and `.np`.

> **gTLD** are managed by ICANN. Examples of these domains are `.com`, `.gov`, `.biz` or `.org`.

DNS client is usually called **resolver**.

### Check-list
- Ensure that master nameservers don't perform **zone transfer** for the unauthorized systems.
```
dig axfr @dns-server domain.name
```
- Use several nameservers to provide aviability.
- Check both forward and reverse zone lookups.

Common FOSS realisations are BIND (named) and dnsmasq. 

## Email
SMTP defined in RFC2821 works in **plain-text** including the **authentication** so you should use lower layer protection - **TLS**. SMTP works between mail servers which is called **Mail Transfer Agents** or **MTAs**.
To prevent spamming the Apache SpamAssasin can be used. It implements anispam technique **Real Time Block Lists (RTBL)**.

Common FOSS realisation is sendmail.

## Security Policy
**In the absence of any policy, work is always done in ad-hoc manner**. You should prefer **Deny by Default**, **defence in depth** and use the **Last Priveledge Rule**. Remember that **prevention is ideal, but detection is a must**.

## Firewalls
Firewalls **enforces access control policy between networks**. Firewalls can be **statefull** or **stateless**. In statefull firewalls like `iptables` one can add the rule that allows traffic for the already opened connections:
```
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
```
