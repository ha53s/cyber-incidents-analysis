# Cisco Catalyst SD-WAN Authentication Bypass

---

## Incident Overview

| Field | Details |
|------|--------|
| Vulnerability Name | Cisco Catalyst SD-WAN Controller Authentication Bypass |
| CVE | CVE-2026-20182 |
| Industry Targeted | Enterprise / Network Infrastructure |
| Date Reported | 2026-05-14 |
| Attack Type | Authentication Bypass |
| Severity | Critical |
| Primary Source | Cisco Security Advisory |
---


## Table of Contents

- [Executive Summary](#executive-summary)
- [Initial Access](#initial-access)
- [Attack Chain Analysis](#attack-chain-analysis)
- [Public Exploitation](#public-exploitation)
- [Indicators of Compromise](#indicators-of-compromise)
- [Detection Opportunities](#detection-opportunities)
- [MITRE ATT&CK Mapping](#mitre-attck-mapping)
- [Impact Assessment](#impact-assessment)
- [References](#references)


## Executive Summary

CVE-2026-20182 is a serious authentication bypass vulnerability that affects Cisco Catalyst SD-WAN controllers. It allows a remote attacker to impersonate a trusted SD-WAN peer, insert a malicious SSH public key into the vmanage-admin account, and get complete configuration control over NETCONF.
It has a CVSS score of 10 requiring no authentication or user interaction.

Key observations:
- DTLS peer authentication bypass
- SSH public key injection into privileged account
- NETCONF access enabling full SD-WAN control


---

## Initial Access

**Attack Vector**
- SD-WAN control plane DTLS interface
- vSmart / vManage peer authentication bypass

**Details**

The attacker acts as a vHub device and uses a self-signed certificate to connect to the SD-WAN DTLS service. vSmart, vManage, vBond, and vEdge devices are subjected to strict certificate and identity validation checks during authentication. However, vHub devices follow a less strict verification path, allowing important validation steps to be skipped. Due to this logic flaw in the `CHALLENGE / CHALLENGE_ACK` authentication flow, the attacker can be mistakenly trusted as a SD-WAN peer without proper  validation.
For more details refer to the Rapid7 research:
https://www.rapid7.com/blog/post/ve-cve-2026-20182-critical-authentication-bypass-cisco-catalyst-sd-wan-controller-fixed/

---

## Attack Chain Analysis

| Stage | Description |
|------|------------|
| Reconnaissance | Attacker scans and identifies exposed SD-WAN DTLS interfaces |
| Initial Access | DTLS session initiated using a self-signed certificate and impersonating a vHub device |
| Execution | Authentication bypass triggered during the CHALLENGE / CHALLENGE_ACK flow |
| Defense Evasion | Controller accepts attacker as a trusted SD-WAN peer |
| Persistence | Malicious SSH public key injected into `vmanage-admin` authorized keys |
| Privilege Escalation | SSH access gained using injected key |
| Lateral Movement | NETCONF access enables configuration manipulation across SD-WAN infrastructure |
| Impact | Full SD-WAN control plane compromise |

---
## Public Exploitation

Public exploitation for CVE-2026-20182 is available through a Metasploit module:

`admin/networking/cisco_sdwan_vhub_auth_bypass`

The module exploits the authentication bypass by injecting a malicious SSH public key into the `vmanage-admin` account.
After successful exploitation the attacker can authenticate through SSH and gain privileged NETCONF access to the SD-WAN controller.

Example:

```bash
ssh -i <generated_key>.pem vmanage-admin@<sd-wan-controller-ip> -p 830
```
## Indicators of Compromise
---
### Authentication Log Indicators

Review `/var/log/auth.log` for unexpected SSH key authentication to the `vmanage-admin` account from unknown IP addresses.

Example:

```bash
2026-02-10T22:51:36+00:00 vm sshd[804]: Accepted publickey for vmanage-admin from UNKNOWN_IP ssh2: RSA SHA256:REDACTED_KEY
```
---

### Behavioral Indicators
- Unauthorized DTLS peering events
- Rogue SD-WAN peer devices
- NETCONF access initiated from unauthorized hosts
- Unknown System IPs appearing in SD-WAN logs

---
# Detection Opportunities

Possible monitoring points defenders could use

- SSH logs to detect unexpected vmanage admin key logins from unknown IPs  
- SD WAN control plane logs to identify unauthorized DTLS peering events  
- Certificate validation to detect self signed or invalid peer certificates  
- NETCONF monitoring to detect unauthorized configuration sessions 

---
# MITRE ATT&CK Mapping

Mapped using the MITRE ATT&CK framework

| Tactic | Technique | ID |
|------|----------|----|
| Initial Access | Exploit Public-Facing Application | T1190 |
| Persistence | Account Manipulation | T1098 |
| Execution | Remote Services | T1021 |
| Impact | Network Data Manipulation | T1565 |

---

# Impact Assessment

| Category | Impact |
|--------|-------|
| Data Exposure | High |
| Operational Disruption | High |
| Financial Damage | High |
| Reputation Damage | High |

---

# References 
- https://www.rapid7.com/blog/post/ve-cve-2026-20182-critical-authentication-bypass-cisco-catalyst-sd-wan-controller-fixed/  
- https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-20182  
