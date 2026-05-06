# [Snapped Phish-ing Line] - TryHackMe Writeup
**Date:** 05/06/2026  
**Room URL:** https://tryhackme.com/room/snappedphishingline
**Difficulty:** Easy
**Category:** Phishing Analysis / Email Forensics  

---

## Objective
Apply knowledge of phishing attack techniques, as well as CTI tools to investigate malicious e-mails and attachments/URL's as part of a larger phishing campaign.

---

## Tools Used
- SHA256
- VirusTotal

---

## Methodology

### Step 1: Initial Triage
I opened the folder containing 5 different e-mails to analyze, and quickly gave each e-mail file a quick lookover.

- All e-mails in the folder are sent from an external e-mail address.
    - 4 of the e-mail subjects are "Group Marketing Online Direct Credit Advice" while the last is "Quote for Services Rendered: processed on June 29, 2020, 10:01:32 AM"
    - They all contain a .html file except for the last, which contains a .pdf file.


### Step 2: Identifying the Malicious Email
- Downloading one of the Group Marketing attachments title "Direct Credit Advice.html", I was redirected to a page that appeared to be a Microsoft login page, however the url was not legitimate.
    - Seems to be a spoofed login page to extract login info from victims.

HTML File URL:
```
http://kennaroads.buzz/data/Update365/office365/40e7baa2f826a57fcf04e5202526f8bd/enterpassword.php?87iJ021778092380439bc96a537ec710b26bbda0c09fa637439bc96a537ec710b26bbda0c09fa637439bc96a537ec710b26bbda0c09fa637439bc96a537ec710b26bbda0c09fa637439bc96a537ec710b26bbda0c09fa637&email=zoe.duncan@swiftspend.finance&error=
```

- By backtracking to the /data directory, we can see the attacker left incriminating files on the system
    - Phishing kit
    - Text file containing extracted login info from successful phishing attempts

![.txt file containing extracted login info](/phishing/phishing-images/snapped-phishing-login-info.png)


### Step 3: Attachment Analysis

- **Attachment name:** Update365.zip
- **Attachment type:** .zip
- **Notes:** Phishing kit (trojan)

### Step 4: Additional Findings
- Used sha256sum to generate a hash for the phishing kit .zip file, then uploaded the hash onto VirusTotal
    - 28/63 security vendors flagged this file as malicious
    - Besides phishing, the .zip file was categorized as a trojan

Bash & VirusTotal
```
sha256sum value = ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
```
![VirusTotal Results](/phishing/phishing-images/snapped-attachment-hash.png)

- Also found a "submit.php" file that contains the e-mail used to receive all extracted login info

.php File
```
$send = "m3npat@yandex.com";
```

---

## Findings Summary

| # | Finding | Significance |
|---|---|---|
| 1 | 5 emails sent from external address targeting swiftspend.finance employees | Coordinated phishing campaign against a single organization |
| 2 | HTML attachment redirects to spoofed Microsoft 365 login page | Credential harvesting attack designed to steal Office 365 logins |
| 3 | Login page hosted on kennaroads.buzz, not a legitimate Microsoft domain | Fake login page designed to appear legitimate to victims |
| 4 | Attacker left phishing kit and extracted credentials exposed in /data directory | |
| 5 | Text file found containing successfully harvested login credentials | Confirms active victims' real credentials were captured |
| 6 | submit.php reveals attacker email m3npat@yandex.com | Attacker using Yandex free email to receive stolen credentials |
| 7 | Update365.zip identified as phishing kit and trojan | 28/63 VirusTotal vendors confirmed malicious |
| 8 | SHA256 hash ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686 flagged as malicious | Confirmed known malicious file |

---

## IOCs (Indicators of Compromise)
- **Phishing domain:** kennaroads.buzz
- **Phishing URL:** http://kennaroads.buzz/data/Update365/office365/40e7baa2f826a57fcf04e5202526f8bd/enterpassword.php
- **Attachment name:** Update365.zip
- **SHA256 hash:** ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
- **VirusTotal detections:** 28/63 vendors flagged as malicious
- **Attacker email:** m3npat@yandex.com
- **Target organization:** swiftspend.finance
- **File type:** Phishing kit / Trojan

---

## MITRE ATT&CK Mapping

| Technique | ID | Tactic |
|---|---|---|
| Spearphishing attachment | T1566.001 | Initial Access |
| Input Capture: GUI Input Capture  | T1056.002  | Collection, Credential Access |

---

## Key Takeaways
- Attackers will spoof login pages of legitimate services to extract user info
- Some attackers' operational infrastructure can expose their tactics and tools
- VirusTotal hash lookups are an efficient way to determine the safety of an attachment

---

## References
- [MITRE ATT&CK T1566.001](https://attack.mitre.org/techniques/T1566/001/)
- [MITRE ATT&CK T1056.002](https://attack.mitre.org/techniques/T1056/002/)