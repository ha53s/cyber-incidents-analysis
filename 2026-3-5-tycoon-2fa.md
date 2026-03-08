# Tycoon 2FA Phishing-As-A-Service Takedown

---

## Incident Overview

| Field | Details |
|------|--------|
| Incident Name |Tycoon 2FA PhaaS |
| Industry Targeted |Multi-sector |
| Date Reported | 2023–2026|
| Attack Type |Adversary-in-the-Middle Phishing |
| Severity | High  |
| Primary Source | https://thehackernews.com/2026/03/europol-led-operation-takes-down-tycoon.html|

---

## Table of Contents

- [Incident Overview](#incident-overview)
- [Executive Summary](#executive-summary)
- [Attack Timeline](#attack-timeline)
- [Initial Access](#initial-access)
- [Attack Chain Analysis](#attack-chain-analysis)
- [Indicators of Compromise](#indicators-of-compromise)
- [Detection Opportunities](#detection-opportunities)
- [Detection Logic](#detection-logic)
- [MITRE ATT&CK Mapping](#mitre-attck-mapping)
- [Impact Assessment](#impact-assessment)
- [Defensive Recommendations](#defensive-recommendations)
- [References](#references)

---

#  Executive Summary

Tycoon 2FA was a phishing-as-a-service platform used by hackers to steal login credentials and bypass multi-factor authentication. It worked as a reverse proxy, acting as a middleman between victims and websites to capture credentials and session cookies. While Europol and other international partners have taken the platform down, understanding its methods provides valuable insights into evolving phishing threats.

Key observations:

- Multiple industries everywhere were targeted.
- MFA can still be bypassed by phishing

---

# Attack Timeline

| Time | Event |
|------|------|
| T+0 min | Victim receives phishing email with a Tycoon phishing link  |
| T+2 min | Victim visits phishing page  |
| T+3 min | Reverse proxy captures credentials and MFA session |
| T+5 min | Attacker logs into victim account using captured session cookies bypassing MFA |
| T+10 min | Attacker takes over the account and begins further activity (data theft, ransomware staging, etc) |

---

# Initial Access

**Attack Vector**

- Reverse proxy phishing emails
- Typosquatted domains


**Details**

Initial access is gained by a phishing campaign that redirected victims to a Tycoon 2FA phishing page. The phishing infrastructure functioned as a middleman (AiTM) proxy that relays traffic between the victim and the service(Microsoft ,Gmail, etc). When a victim logs in, the phishing proxy captures username and password,MFA codes and authenticated session cookies.Since the session token was captured after successful authentication, attackers could log in to the service without needing the MFA code again ,bypassing MFA.

---

# Attack Chain Analysis

| Stage | Description |
|------|------------|
| Reconnaissance | Attacker gathers information about the targeted organization and crafts phishing campaigns impersonating services like Microsoft , Gmail, etc. |
| Initial Access | Victim clicks the phishing link leading to Tycoon 2FA reverse proxy login pages. |
| Execution | Malicious JavaScript captures credentials, MFA tokens, and session cookies. |
| Persistence | The stolen sessions gives the attacker access without needing the password |
| Lateral Movement | Attacker performs ATO jumping and uses the compromised account to compromise more accounts |
| Impact | Attackers may conduct data exfiltration, BEC , or stage ransomware. |

---

#  Indicators of Compromise

## Suspicious Domains
Tycoon 2FA used hundreds of temporary domains that are typosquatted and protected by Cloudflare, a larger list of identified domains is available in the linked GitHub repository:
https://github.com/NoMorePhish/Tycoon2FADomains

## Behavioral Indicators
- Login sessions from unusual ips and locations
- Suspicious JavaScript
- Reverse proxy traffic patterns


---

# Detection Opportunities

Possible monitoring points defenders could use:

| Detection Area | Example |
|---------------|--------|
| Email Security | Detect phishing emails with suspicious login domains |
| DNS Monitoring | Identify newly registered domains impersonating login services |
| Session Monitoring | Detect session reuse and token activity  |


---
# Detection Logic 

**Rule:** Suspicious Login Following Phishing Domain Visit

```text
Detection Logic: Potential AiTM Phishing Compromise

IF
  User login occurs from a new IP address
AND
  User-Agent differs from the user's normal login behavior
AND
  Login occurs shortly after visiting a suspicious domain

THEN
  Generate alert: Possible Adversary-in-the-Middle phishing attack
```



# MITRE ATT&CK Mapping

Mapped using the  
MITRE ATT&CK framework.

| Tactic | Technique | ID |
|------|----------|----|
| Initial Access | Phishing | T1566 |
| Credential Access | Adversary-in-the-Middle | T1557 |
| Lateral Movement | Valid Accounts | T1078 |



---

#  Impact Assessment

| Category | Impact |
|--------|-------|
| Data Exposure | Possible |
| Operational Disruption | Yes |
| Financial Damage | Possible |
| Reputation Damage | High |

---

#  Defensive Recommendations

- Use **FIDO2 , Passkeys** or **hardware security keys**
- Apply **conditional access policies** (location/device checks)
- **Revoke active sessions** after password changes
- **Block suspicious domains** via DNS web filters


# References
- https://thehackernews.com/2026/03/europol-led-operation-takes-down-tycoon.html
- https://github.com/NoMorePhish/Tycoon2FADomains
- https://attack.mitre.org/
- https://www.europol.europa.eu/media-press/newsroom/news/global-phishing-service-platform-taken-down-in-coordinated-public-private-action?/
- https://www.microsoft.com/en-us/security/blog/2022/07/12/from-cookie-theft-to-bec-attackers-use-aitm-phishing-sites-as-entry-point-to-further-financial-fraud/?/


# Disclaimer

This analysis is based on publicly available information and is intended for educational and research purposes only.



