# [The Greenholt Phish] - TryHackMe Writeup
**Date:** 05/05/2026  
**Room URL:** https://tryhackme.com/room/phishingemails5fgjlzxc  
**Difficulty:** Easy
**Category:** Phishing Analysis / Email Forensics  

---

## Objective
Use my knowledge of phishing techniques, e-mail header and body analysis, SPF/DKIM/DMARC, as well as tools such as IP/domain reputation checkers to analyze an e-mail and attachment(s) and determine whether it is legitimate or a phishing attempt.

---

## Tools Used
- Talos Intelligence
- MXToolBox
- SHA256
- VirusTotal

---

## Methodology

![Email to analyze](/phishing/phishing-images/greenholt-phishing-email.png)

### Step 1: Initial Triage
I opened the .eml file and viewed the message source under View -> Message Source.

### Step 2: Identifying the Malicious Email
A large portion of the information in the message source seemed to be obfuscated, SPF (Sender Policy Framework) check returned a fail, and DMARC check returned "unknown".

Message Source:
```
X-Originating-Ip: [x.x.x.x]
Received-SPF: fail (domain of mutawamarine.com does not designate x.x.x.x as permitted sender)
dmarc=unknown
```

### Step 3: Email Header Analysis

| Header | Value | Significance |
|---|---|---|
| From | info@mutawamarine.com |  |
| To | webmaster@redacted.org | |
| Subject | your: Transfer Reference Number:(09674321) | Unusual subject regarding reference number; social engineering |
| X-Originating-IP | [x.x.x.x] | Sender IP obfuscated |
| Reply-To | info.mutawamarine@mail.com | Mismatch |

- Unusual subject line with a reference number (could be social engineering to imply urgency)
- Obfuscated sender IP
- Mismatched domains between From and Reply-To addresses

### Step 4: Attachment Analysis

- **Attachment name:** SWT_#09674321___PDF__.CAB
- **Attachment type:** .CAB
- **Notes:** Unusual/suspicious file type .CAB

### Step 5: SPF/DMARC Analysis

Message Source
```
Received-SPF: fail (domain of mutawamarine.com does not designate x.x.x.x as permitted sender)
Authentication-Results: atlas125.free.mail.bf1.yahoo.com;
 spf=fail smtp.mailfrom=mutawamarine.com;
 dmarc=unknown
```

MXToolbox SPF/DMARC Record Checker
```
v=spf1 include:spf.protection.outlook.com -all
v=DMARC1; p=quarantine; fo=1
```

- Domain does not designate the obfuscated IP to send mail on it's behalf. 
- "dmarc=unknown" shows there is no policy configured to reject or quarantine emails that fail SPF or DKIM checks, yet DMARC check shows fo=1, meaning it should generate a report when SPF/DKIM fails. These findings contradict each other.

### Step 6: Additional Findings
- Source IP shown further down in message source
    - talosintelligence.com (IP/Domain Reputation Checker) returned HOSTWINDS LLC as the network owner.
- Used sha256sum to generate a hash for the .CAB file, then uploaded the hash onto VirusTotal
    - 49/63 security vendors flagged this file as malicious
    - The actual file type is a .rar

Message Source
```
Received: from hwsrv-737338.hostwindsdns.com ([192.119.71.157]:51810 helo=mutawamarine.com)
```

Bash & VirusTotal
```
sha256sum value = 2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f
```
![VirusTotal Results](/phishing/phishing-images/greenholt-attachment-hash.png)

---

## Findings Summary

| # | Finding | Significance |
|---|---|---|
| 1 | SPF returned fail — sending IP 192.119.71.157 not authorized by mutawamarine.com | Confirms email was not sent from legitimate infrastructure |
| 2 | X-Originating-IP obfuscated as [x.x.x.x] | Sender deliberately concealed true origin to avoid detection |
| 3 | Reply-To mismatch — from mutawamarine.com but replies go to mail.com | Classic phishing technique to intercept victim replies |
| 4 | Sending server hwsrv-737338.hostwindsdns.com owned by Hostwinds LLC | Third party hosting provider unrelated to mutawamarine.com infrastructure |
| 5 | .CAB attachment disguised as PDF | Unusual file type used to bypass email filters that block .exe or .zip |
| 6 | Actual file type is .RAR despite .CAB extension | File extension spoofing to further conceal malicious payload |
| 7 | 49/63 VirusTotal vendors flagged attachment as malicious | Confirms attachment is known malware |
| 8 | DMARC returned unknown despite quarantine policy existing | Inconsistency suggests policy was bypassed or not enforced at time of sending |

---

## IOCs (Indicators of Compromise)
- **Sender IP:** 192.119.71.157
- **Sender domain:** mutawamarine.com
- **Sending server:** hwsrv-737338.hostwindsdns.com
- **Network owner:** Hostwinds LLC
- **Attachment name:** SWT_#09674321___PDF__.CAB
- **True file type:** .RAR
- **SHA256 hash:** 2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f
- **VirusTotal detections:** 49/63 vendors flagged as malicious
- **Reply-To address:** info.mutawamarine@mail.com
- **From address:** info@mutawamarine.com

---

## MITRE ATT&CK Mapping

| Technique | ID | Tactic |
|---|---|---|
| Phishing attachment | T1566.001 | Initial Access |

---

## Key Takeaways
- Files extensions can be spoofed to disguise malicious payloads; always verify true file type
- DMARC/SPF does not guarantee e-mail will not be delivered; bad actors can still find ways to circumvent this
- Reply-To mismatches are a reliable indicator of a phishing attempt
- Free email providers (such as mail.com) are a red flag as legitimate organizations will reply from their own domains
- VirusTotal hash lookups are an efficient way to determine the safety of an attachment

---

## References
- [MITRE ATT&CK T1566.001](https://attack.mitre.org/techniques/T1566/001/)