# Phase 7 Detailed Lab Log — Phishing Investigation, IOC Extraction, Email Security Triage, and Incident Reporting

## Phase 7 Title
**Phase 7 — Phishing and Email Security Investigation**

## Phase 7 Status
**Completed**

Phase 7 focused on phishing investigation and email security analysis from a SOC Analyst Level 1 perspective. Earlier phases built the virtual lab, SIEM visibility, endpoint telemetry, Wazuh visibility, alert triage workflow, and SOC ticket-writing skills. Phase 7 shifted into one of the most common real-world SOC queues: suspicious email reports.

This writeup is intentionally detailed so aspiring SOC analysts can use it as a repeatable phishing triage guide.

---

# 1. Purpose of Phase 7

The goal of Phase 7 was to learn how SOC analysts investigate suspicious emails.

In a real SOC, phishing reports are common. Users may report emails that look suspicious, or an email security gateway may generate alerts for suspicious messages. The analyst’s job is not just to say “this looks fake.” The analyst must collect evidence and determine:

- Is this email malicious?
- What is the attacker trying to get?
- Did the user interact with it?
- Who else received it?
- What should be blocked, removed, or escalated?

Phase 7 focused on:

- Understanding common phishing types
- Reviewing sender domains
- Reviewing URLs
- Reviewing attachments
- Identifying brand/person impersonation
- Identifying social engineering techniques
- Extracting IOCs
- Determining likely attack goals
- Writing phishing investigation tickets
- Writing a final phishing incident report
- Practicing SOC response recommendations

---

# 2. VMs Needed for Phase 7

Phase 7 is less VM-heavy than earlier phases.

Recommended VM usage:

- Ubuntu SIEM VM: Optional / recommended if reviewing logs or documenting in the lab
- Windows 10 Victim VM: Optional if generating browser/endpoint artifacts
- Wazuh Server VM: Not required
- Kali Linux VM: Not required

For the first part of Phase 7, browser-based investigation and safe simulated phishing samples were enough.

Professor decision:

- Start Ubuntu SIEM VM if needed for documentation or later log correlation.
- Keep Wazuh and Kali powered off to conserve resources.
- Windows victim is optional unless endpoint artifacts are needed.

Reason: phishing triage mostly happens through email content, headers, URL/domain analysis, attachment review, and SOC documentation.

---

# 3. SOC-Relevant Phishing Lesson

## What phishing is

Phishing is a social engineering attack where an attacker tries to trick a user into taking an unsafe action.

Common unsafe actions:

- Clicking a link
- Entering credentials
- Opening an attachment
- Sending money/gift cards
- Approving MFA
- Downloading malware
- Replying with sensitive information

For a SOC analyst, phishing investigation is evidence-based. The analyst should answer:

- Is this email malicious?
- What is the attacker trying to get?
- Did the user interact with it?
- Who else received it?
- What should be blocked, removed, or escalated?

---

# 4. Major Phishing Categories Covered

## Credential Harvesting

The attacker sends the user to a fake login page.

Example: “Your Microsoft password expires today. Click here to verify.”

Goal: steal username/password, MFA codes, MFA tokens, or session tokens.

Common clues:

- Fake Microsoft/Google/Okta/DocuSign domain
- Urgency
- Login page link
- Lookalike domain
- Account suspension threat

## Malware Delivery

The attacker sends an attachment or download link.

Example: “Invoice attached. Please review.”

Goal: trick the user into opening a malicious file or running malware.

Common clues:

- .zip
- .iso
- .exe
- .scr
- .js
- .vbs
- .docm
- .xlsm
- password-protected archive
- OneDrive/Dropbox download link

## Business Email Compromise

The attacker impersonates an executive, vendor, payroll team, or finance contact.

Example: “Can you update our banking details?”

Goal: gift cards, wire transfer, payroll redirect, vendor payment fraud, or sensitive information.

Common clues:

- No malware
- No link
- Impersonation
- Urgency
- Payment/bank/vendor language
- Reply-to mismatch
- External sender pretending to be internal leadership

## MFA / Security Alert Phishing

The attacker impersonates Microsoft, Okta, Duo, Google, or another identity/security platform.

Example: “Unusual sign-in attempt blocked. Verify your account now.”

Goal: steal credentials, MFA codes, session tokens, or push users into fake verification flows.

Common clues:

- Fake security alert
- Urgency/fear
- Account lockout threat
- Identity verification link
- Lookalike identity provider domain

---

# 5. What a SOC Analyst Checks First

## Sender

Check:

- Display name
- Sender email
- Sender domain
- Reply-To address
- Return-Path

Important lesson: display names can lie. The actual sender address and domain matter more.

Example:

Display name: Microsoft Support  
Real sender: support@micros0ft-security[.]com

This is suspicious because `micros0ft` uses a zero instead of the letter “o”.

## Links

Check:

- Visible link text
- Actual destination URL
- Root domain
- Subdomain
- URL path
- URL shorteners
- Lookalike spelling
- HTTPS usage

Important lesson: HTTPS does not mean safe. A malicious site can still use HTTPS.

The real question is: is the domain legitimate, is the link destination trusted, and does the email pressure the user to click?

## Attachments

Check:

- File name
- File extension
- File type
- Compressed archives
- Macro-enabled documents
- Executables/scripts
- Password-protected files
- Attachment hash

Risky attachment examples:

- .zip
- .iso
- .exe
- .scr
- .js
- .vbs
- .docm
- .xlsm
- .lnk

A PDF is not automatically safe, and a ZIP is not automatically malicious. The analyst must review sender, context, file type, and behavior.

## Email Body

Look for social engineering:

- Urgency
- Fear
- Threat of account suspension
- Financial request
- Too-good-to-be-true offer
- Generic greeting
- Brand impersonation
- Executive impersonation
- Pressure to reply quickly

## Authentication Results

For real email headers, check:

- SPF
- DKIM
- DMARC
- Received path
- Sending IP
- Return-Path
- Reply-To
- Message-ID domain

Simple explanation:

- SPF = was the sending server allowed to send for the domain?
- DKIM = was the message cryptographically signed?
- DMARC = does the domain enforce SPF/DKIM alignment?

SPF/DKIM/DMARC failures do not automatically prove phishing, but they increase suspicion and help validate spoofing.

---

# 6. IOC Extraction

IOC means Indicator of Compromise.

For phishing, extract:

- Sender email
- Sender domain
- Reply-to email
- Return-path domain
- URL
- Root domain
- IP address
- Attachment filename
- Attachment hash
- Subject line
- Message-ID
- Sending IP
- Impersonated brand/person
- Email theme

IOCs are important because SOC teams use them to:

- Block domains/URLs
- Search for other recipients
- Remove matching messages
- Correlate with endpoint logs
- Correlate with identity sign-in logs
- Escalate to IR if users interacted

---

# 7. Phishing Verdict Types

Common verdicts:

- Benign: expected sender, expected content, safe links, no suspicious indicators.
- Spam: unwanted bulk email, annoying but not clearly malicious.
- Suspicious: some red flags, but not enough evidence to confirm malicious.
- Phishing: social engineering attempt to steal credentials, money, access, or sensitive info.
- Malware: attachment or link likely delivers malicious payload.
- BEC: Business Email Compromise / impersonation for financial or business fraud.

---

# 8. General SOC Response Actions

For phishing investigations, recommended actions may include:

- Block sender/domain/URL
- Search for other recipients
- Remove/quarantine matching emails
- Warn affected users
- Ask whether the reporting user clicked/replied/opened attachment
- Check if credentials were entered
- Reset password if credentials were entered
- Revoke active sessions if credentials were entered
- Review identity provider sign-in logs
- Submit URL/domain/file to reputation or sandbox tools
- Escalate to incident response if malware execution or account compromise is suspected

---

# 9. Phase 7 Investigation Workflow

Every case followed this workflow:

1. Review subject, sender, body, URL, and attachment.
2. Identify the impersonated brand/person.
3. Extract IOCs.
4. Identify social engineering technique.
5. Determine likely attacker goal.
6. Decide verdict.
7. Recommend SOC actions.
8. Write phishing investigation ticket.

---

# 10. Phishing Investigation Ticket Template

# Phishing Investigation Ticket X — <Case Name>

## Severity
Low / Medium / High / Critical

## Status
Confirmed phishing / Suspected phishing / Benign / Escalated

## Alert Summary
Short summary of what was reported and why it is suspicious.

## Reported Email Details
Subject:
Sender:
URL(s):
Attachment(s):
Reported by:

## IOCs Extracted
Sender email:
Sender domain:
URL(s):
URL domain(s):
Attachment filename:
Attachment hash:
Subject:
Impersonated brand/person:
Email theme:

## Analysis
Explain sender, URL, attachment, social engineering technique, and likely attack goal.

## Verdict
Phishing / BEC / Malware delivery / Credential harvesting / Suspicious / Benign

## Recommended SOC Actions
- Action 1
- Action 2
- Action 3

---

# 11. Case 1 — Microsoft 365 Password Expiration Phishing

## Scenario

Subject: Urgent: Password Expiration Notice

Sender: IT Support <support@micros0ft-security[.]com>

Body:
Your Microsoft 365 password expires today. Failure to verify your account will result in access suspension. Click below to keep your account active.

URL: hxxps://micros0ft-security-login[.]com/verify

Attachment: None

Reported by: User reported suspicious email to SOC mailbox.

## Analysis Notes

The sender domain is suspicious because it uses typosquatting/lookalike spelling. The word `micros0ft` uses a zero instead of the letter “o”. The URL also uses a Microsoft-themed lookalike domain.

The message uses urgency and fear by claiming the password expires today and threatening account suspension. No attachment is present. The likely attack goal is credential harvesting and possible MFA/session token theft.

## IOCs Extracted

- Sender email: support@micros0ft-security[.]com
- Sender domain: micros0ft-security[.]com
- URL: hxxps://micros0ft-security-login[.]com/verify
- URL domain: micros0ft-security-login[.]com
- Subject: Urgent: Password Expiration Notice
- Impersonated brand/service: Microsoft / Microsoft 365
- Attachment: None

## Ticket 1 — Microsoft 365 Password Expiration Phishing

### Severity
Medium

### Status
Confirmed phishing / credential harvesting attempt

### Alert Summary
A user reported a suspicious email claiming their Microsoft 365 password would expire today. The message attempted to pressure the user into clicking a verification link to avoid account suspension.

### Reported Email Details
Subject: Urgent: Password Expiration Notice  
Sender: IT Support <support@micros0ft-security[.]com>  
URL: hxxps://micros0ft-security-login[.]com/verify  
Attachment: None  
Reported by: User reported suspicious email to SOC mailbox

### IOCs Extracted
Sender email: support@micros0ft-security[.]com  
Sender domain: micros0ft-security[.]com  
URL: hxxps://micros0ft-security-login[.]com/verify  
URL domain: micros0ft-security-login[.]com  
Attachment filename: N/A  
Subject: Urgent: Password Expiration Notice  
Impersonated brand/service: Microsoft / Microsoft 365

### Analysis
The sender domain is suspicious because it uses typosquatting/lookalike spelling. The domain `micros0ft-security[.]com` uses a zero instead of the letter “o” in Microsoft and is not a legitimate Microsoft domain.

The URL is also suspicious because it points to a Microsoft-themed lookalike domain instead of an official Microsoft-owned domain. The email uses urgency and fear by claiming the user’s password expires today and threatening account suspension if the user does not verify the account.

No attachment was included. The likely attack goal is credential harvesting, with possible theft of Microsoft 365 credentials, MFA tokens, or session information if the user enters credentials into the fake login page.

### Verdict
Phishing — credential harvesting attempt

### Recommended SOC Actions
- Block the sender domain and URL.
- Search mailboxes for other recipients using the sender, subject, URL, and domain.
- Remove/quarantine matching emails from user mailboxes.
- Ask the reporting user whether they clicked the link or entered credentials.
- If credentials were entered, reset the user’s password and revoke active sessions.
- Review Microsoft 365 or identity provider sign-in logs for suspicious authentication activity.

---

# 12. Case 2 — Suspicious Invoice ZIP Attachment

## Scenario

Subject: Invoice INV-88421 Past Due

Sender: Billing Department <billing@secure-invoices-payments[.]com>

Body:
Hello,

Your invoice payment is overdue. Please review the attached invoice and submit payment today to avoid late fees.

Attachment: Invoice_INV-88421.zip

URL: None

Reported by: User reported suspicious attachment to SOC mailbox.

## Analysis Notes

The sender domain is suspicious because it uses a generic invoice/payment-themed domain rather than a trusted vendor domain. The email uses financial urgency by claiming that payment is overdue and must be submitted today.

There is no URL. The primary risk is the ZIP attachment. Compressed attachments are commonly used in phishing campaigns to hide scripts, executables, shortcut files, malicious documents, or payloads.

The likely attack goal is malware delivery or initial access.

## IOCs Extracted

- Sender email: billing@secure-invoices-payments[.]com
- Sender domain: secure-invoices-payments[.]com
- URL: None
- Attachment filename: Invoice_INV-88421.zip
- Subject: Invoice INV-88421 Past Due
- Theme: Invoice / overdue payment

## Ticket 2 — Suspicious Invoice Attachment

### Severity
High

### Status
Suspected phishing / possible malware delivery

### Alert Summary
A user reported a suspicious invoice-themed email claiming that an invoice payment was overdue. The email attempted to pressure the user into opening a ZIP attachment named `Invoice_INV-88421.zip`.

### Reported Email Details
Subject: Invoice INV-88421 Past Due  
Sender: Billing Department <billing@secure-invoices-payments[.]com>  
URL: None observed  
Attachment: Invoice_INV-88421.zip  
Reported by: User reported suspicious attachment to SOC mailbox

### IOCs Extracted
Sender email: billing@secure-invoices-payments[.]com  
Sender domain: secure-invoices-payments[.]com  
URL: N/A  
URL domain: N/A  
Attachment filename: Invoice_INV-88421.zip  
Attachment hash: Not available in scenario  
Subject: Invoice INV-88421 Past Due  
Impersonated brand/service: Billing department / invoice payment theme

### Analysis
The sender domain is suspicious because it uses a generic invoice/payment-themed domain rather than a known trusted vendor domain. The email uses financial urgency by claiming the invoice is overdue and that payment must be submitted today to avoid late fees.

No URL was observed in the email. The main suspicious indicator is the ZIP attachment. Compressed attachments are commonly used in phishing campaigns to hide malicious files, scripts, shortcut files, executables, or macro-enabled documents. The invoice theme is also commonly used to trick users into opening attachments.

The likely attack goal is malware delivery or initial access. If the user opens the ZIP file and runs the contents, the attacker may be attempting to execute malware, steal credentials, or gain access to the endpoint.

### Verdict
Suspicious phishing email / possible malware delivery attempt

### Recommended SOC Actions
- Instruct the user not to open the attachment.
- Quarantine or remove the email from the reported mailbox.
- Search for other recipients using the sender, subject, and attachment filename.
- Submit the attachment hash or file to sandbox/malware analysis if available.
- Block the sender domain if confirmed malicious.
- Check endpoint telemetry for evidence that the attachment was opened or executed.
- If execution occurred, escalate to incident response for endpoint investigation.

---

# 13. Case 3 — CEO / Business Email Compromise Attempt

## Scenario

Subject: Quick favor needed

Sender: CEO <ceo.company.office@gmail[.]com>

Body:
Hi,

Are you available? I need you to handle something for me quickly. I am tied up in a meeting and cannot talk right now. Please reply when you get this.

Thanks,
CEO

URL: None

Attachment: None

Reported by: Employee reported suspicious executive request to SOC mailbox.

## Analysis Notes

This case demonstrated that phishing does not always include links or attachments. A CEO using a random Gmail address instead of the company’s legitimate domain is suspicious.

The attacker is using executive impersonation, urgency, and pressure to start a conversation. The likely goal is Business Email Compromise. If the user replies, the attacker may ask for gift cards, wire transfers, payroll changes, vendor payment changes, sensitive employee information, or other business actions.

## IOCs Extracted

- Sender email: ceo.company.office@gmail[.]com
- Sender domain: gmail[.]com
- Subject: Quick favor needed
- Impersonated person: CEO / executive leadership
- Theme: urgent executive request
- URL: None
- Attachment: None

## Ticket 3 — CEO / Executive Impersonation BEC Attempt

### Severity
Medium

### Status
Suspected phishing / BEC attempt

### Alert Summary
An employee reported a suspicious email claiming to be from the CEO. The message asked whether the employee was available and attempted to start a conversation while claiming the sender was busy in a meeting and could not talk.

### Reported Email Details
Subject: Quick favor needed  
Sender: CEO <ceo.company.office@gmail[.]com>  
URL: None observed  
Attachment: None  
Reported by: Employee reported suspicious executive request to SOC mailbox

### IOCs Extracted
Sender email: ceo.company.office@gmail[.]com  
Sender domain: gmail[.]com  
URL: N/A  
Attachment filename: N/A  
Subject: Quick favor needed  
Impersonated person: CEO / executive leadership  
Email theme: urgent executive request

### Analysis
The sender is suspicious because the email claims to come from the CEO but uses a Gmail address instead of an official company domain. The message does not contain a URL or attachment, but that does not make it safe.

The email uses executive impersonation, urgency, and pressure. The attacker attempts to start a conversation by asking the employee to reply quickly while claiming to be unavailable for a phone call. This is a common BEC pattern. After the victim replies, the attacker may request gift cards, wire transfers, vendor payment changes, payroll changes, sensitive employee data, or other financial actions.

### Verdict
Business Email Compromise / executive impersonation attempt

### Recommended SOC Actions
- Quarantine or remove the email.
- Search for other recipients using sender, subject, and similar body text.
- Block the sender address.
- Warn the targeted user not to reply.
- Ask whether the user replied or sent any information.
- Notify the real executive or internal security team if impersonation is confirmed.
- Monitor for follow-up emails from the same sender or similar addresses.

---

# 14. Case 4 — Fake Okta Security Alert

## Scenario

Subject: Unusual Sign-In Attempt Blocked

Sender: Security Alert <alerts@okta-verification[.]com>

Body:
We detected an unusual sign-in attempt from a new location. To prevent account lockout, confirm your identity immediately.

Review sign-in:
hxxps://okta-verification[.]com/security-check

If this was not you, verify your account now to avoid temporary suspension.

Attachment: None

Reported by: User reported suspicious security alert to SOC mailbox.

## Analysis Notes

The sender domain `okta-verification[.]com` is suspicious because it is an Okta-themed lookalike domain and not clearly an official Okta-owned domain. The URL uses the same lookalike domain.

The email uses fear and urgency by referencing an unusual sign-in attempt and threatening account lockout/suspension. The likely goal is credential harvesting and possible MFA/session token theft.

## IOCs Extracted

- Sender email: alerts@okta-verification[.]com
- Sender domain: okta-verification[.]com
- URL: hxxps://okta-verification[.]com/security-check
- URL domain: okta-verification[.]com
- Subject: Unusual Sign-In Attempt Blocked
- Impersonated service: Okta / identity security
- Theme: unusual sign-in / account verification
- Attachment: None

## Ticket 4 — Fake Okta Security Alert Phishing

### Severity
Medium

### Status
Confirmed phishing / MFA-themed credential harvesting attempt

### Alert Summary
A user reported a suspicious security alert claiming that an unusual sign-in attempt had been detected. The email instructed the user to verify their account using an Okta-themed link.

### Reported Email Details
Subject: Unusual Sign-In Attempt Blocked  
Sender: Security Alert <alerts@okta-verification[.]com>  
URL: hxxps://okta-verification[.]com/security-check  
Attachment: None  
Reported by: User reported suspicious security alert to SOC mailbox

### IOCs Extracted
Sender email: alerts@okta-verification[.]com  
Sender domain: okta-verification[.]com  
URL: hxxps://okta-verification[.]com/security-check  
URL domain: okta-verification[.]com  
Attachment filename: N/A  
Subject: Unusual Sign-In Attempt Blocked  
Impersonated service: Okta / identity security  
Email theme: unusual sign-in / account verification

### Analysis
The sender domain is suspicious because it uses an Okta-themed lookalike domain and is not clearly an official Okta-owned domain. The URL also points to the same lookalike domain and is designed to appear like a legitimate identity verification page.

The email uses fear and urgency by claiming there was an unusual sign-in attempt and pressuring the user to confirm their identity immediately to avoid account lockout or suspension.

No attachment was included. The likely attack goal is credential harvesting and possible MFA/session token theft. The attacker likely wants the user to enter login credentials or complete a fake verification flow.

### Verdict
Phishing — credential harvesting / MFA-themed phishing attempt

### Recommended SOC Actions
- Block the sender domain and URL.
- Search for other recipients using sender, subject, URL, and domain.
- Quarantine or remove matching emails.
- Ask whether the user clicked the link or entered credentials.
- If credentials were entered, reset password and revoke active sessions.
- Review identity provider sign-in logs for suspicious activity.
- Monitor for additional Okta-themed phishing messages.

---

# 15. Case 5 — Fake DocuSign Document Review

## Scenario

Subject: Document Ready for Signature

Sender: DocuSign Notification <notification@docusign-secure[.]net>

Body:
You have received a secure document that requires your signature. Please review and sign the document before it expires.

Review Document:
hxxps://docusign-secure[.]net/document/review?id=88421

Attachment: None

Reported by: User reported suspicious document signing email to SOC mailbox.

## Analysis Notes

The sender domain `docusign-secure[.]net` is suspicious because it is a DocuSign-themed lookalike domain and not the normal legitimate DocuSign domain.

The URL uses HTTPS, but HTTPS does not mean the site is safe. A malicious site can still have a valid certificate. The domain and context matter more.

The email uses trust abuse and urgency by impersonating a document-signing platform and telling the user the document requires signature before it expires.

The likely goal is credential harvesting.

## IOCs Extracted

- Sender email: notification@docusign-secure[.]net
- Sender domain: docusign-secure[.]net
- URL: hxxps://docusign-secure[.]net/document/review?id=88421
- URL domain: docusign-secure[.]net
- Subject: Document Ready for Signature
- Impersonated service: DocuSign
- Theme: document signing / secure document review
- Attachment: None

## Ticket 5 — Fake DocuSign Document Review Phishing

### Severity
Medium

### Status
Confirmed phishing / credential harvesting attempt

### Alert Summary
A user reported a suspicious DocuSign-themed email stating that a secure document was ready for signature. The message attempted to direct the user to a DocuSign-themed lookalike URL.

### Reported Email Details
Subject: Document Ready for Signature  
Sender: DocuSign Notification <notification@docusign-secure[.]net>  
URL: hxxps://docusign-secure[.]net/document/review?id=88421  
Attachment: None  
Reported by: User reported suspicious document signing email to SOC mailbox

### IOCs Extracted
Sender email: notification@docusign-secure[.]net  
Sender domain: docusign-secure[.]net  
URL: hxxps://docusign-secure[.]net/document/review?id=88421  
URL domain: docusign-secure[.]net  
Attachment filename: N/A  
Subject: Document Ready for Signature  
Impersonated service: DocuSign  
Email theme: document signing / secure document review

### Analysis
The sender domain is suspicious because it uses a DocuSign-themed lookalike domain and is not the normal legitimate DocuSign domain. Attackers often add words like `secure`, `login`, `verify`, `document`, or `support` to fake domains to make them look trustworthy.

The URL is suspicious because it uses the same lookalike domain. The use of HTTPS does not make the site safe, because malicious sites can also use HTTPS certificates.

No attachment was included. The email uses trust abuse and urgency by impersonating a trusted document-signing platform and pressuring the user to review and sign the document before it expires.

The likely attack goal is credential harvesting. The attacker likely wants the user to click the document review link and enter credentials, potentially exposing login credentials, MFA tokens, or session information.

### Verdict
Phishing — credential harvesting attempt using DocuSign impersonation

### Recommended SOC Actions
- Block the sender domain and URL.
- Search for other recipients using sender, subject, URL, and domain.
- Quarantine or remove matching emails.
- Ask whether the user clicked the link or entered credentials.
- If credentials were entered, reset password and revoke active sessions.
- Review sign-in logs for suspicious authentication activity.
- Monitor for additional DocuSign-themed phishing messages.

---

# 16. Final Phishing Incident Report

## Executive Summary

During Phase 7, five simulated phishing scenarios were investigated using SOC analyst methodology. The cases included Microsoft 365 credential phishing, invoice attachment phishing, executive impersonation/BEC, fake Okta security alert phishing, and fake DocuSign document review phishing.

The investigation focused on identifying suspicious sender domains, URLs, attachments, impersonated brands, social engineering techniques, IOCs, likely attacker goals, final verdicts, and recommended SOC response actions.

## Scope

This investigation covered five simulated phishing reports submitted to a SOC mailbox.

## Cases Reviewed

1. Microsoft 365 password expiration phishing
2. Suspicious invoice ZIP attachment
3. CEO / executive impersonation BEC attempt
4. Fake Okta unusual sign-in alert
5. Fake DocuSign document review link

## Key Findings

- Multiple emails used brand impersonation.
- Several messages used urgency or fear to pressure the user.
- Lookalike domains were used to appear legitimate.
- Some emails contained no attachments but were still malicious.
- Some emails contained no links but still represented BEC risk.
- ZIP attachments can indicate possible malware delivery.
- Credential harvesting was the most common suspected goal.
- HTTPS does not guarantee a website is safe.
- Display names can be spoofed.
- The actual sender domain and URL domain matter more than the display name.

## IOCs Identified

- support@micros0ft-security[.]com
- micros0ft-security[.]com
- micros0ft-security-login[.]com
- hxxps://micros0ft-security-login[.]com/verify
- billing@secure-invoices-payments[.]com
- secure-invoices-payments[.]com
- Invoice_INV-88421.zip
- ceo.company.office@gmail[.]com
- gmail[.]com
- Subject: Quick favor needed
- alerts@okta-verification[.]com
- okta-verification[.]com
- hxxps://okta-verification[.]com/security-check
- notification@docusign-secure[.]net
- docusign-secure[.]net
- hxxps://docusign-secure[.]net/document/review?id=88421

## Analysis

The reviewed phishing cases demonstrated several common phishing techniques, including typosquatting, brand impersonation, urgency, fear, attachment-based delivery, fake security alerts, document-signing impersonation, and business email compromise.

The Microsoft and Okta-themed emails were likely credential harvesting attempts. The invoice ZIP attachment case was assessed as possible malware delivery. The CEO email was assessed as a BEC attempt because it relied on executive impersonation and attempted to start a conversation without using a link or attachment. The DocuSign-themed email was assessed as credential phishing through document-signing impersonation.

## Impact Assessment

No real users or production systems were impacted because these were simulated lab scenarios.

In a production environment, possible impact could include:

- Credential theft
- MFA/session token theft
- Malware execution
- Initial access
- Account takeover
- Financial fraud
- Vendor/payment fraud
- Sensitive information disclosure

## Recommended SOC Actions

- Block confirmed malicious sender domains and URLs.
- Search for other recipients across the mail environment.
- Quarantine or remove matching emails.
- Ask reporting users whether they clicked, replied, opened attachments, or entered credentials.
- Reset passwords and revoke sessions if credentials were entered.
- Review identity provider sign-in logs for suspicious authentication activity.
- Submit attachments or URLs to sandbox/reputation tools when safe.
- Escalate to incident response if malware execution, account compromise, or financial fraud is suspected.

## Final Verdict

The five reviewed cases represented simulated phishing activity, including credential harvesting, possible malware delivery, BEC, and brand impersonation. Phase 7 successfully demonstrated phishing triage, IOC extraction, verdict writing, and SOC response planning.

---

# 17. Lessons Learned

## Lesson 1 — Phishing does not always have a link

The CEO/BEC case had no URL and no attachment, but it was still malicious because it attempted to start a conversation for possible fraud.

## Lesson 2 — Phishing does not always have an attachment

The Microsoft, Okta, and DocuSign cases had no attachments, but they were still phishing attempts because they used fake login/security/document links.

## Lesson 3 — HTTPS does not mean safe

A malicious website can still use HTTPS. Analysts must inspect the domain, sender, URL path, and email context.

## Lesson 4 — Display names can lie

The displayed name may say “IT Support,” “Security Alert,” “CEO,” or “DocuSign Notification,” but the actual sender address and domain must be reviewed.

## Lesson 5 — Lookalike domains are a major indicator

Examples:

- micros0ft-security[.]com
- okta-verification[.]com
- docusign-secure[.]net

## Lesson 6 — Attachments require caution

ZIP attachments are commonly used to hide payloads, scripts, shortcuts, executables, or malicious documents.

## Lesson 7 — BEC attacks often start simple

The first email may only ask:

- Are you available?
- Can you handle something for me?
- Please reply when you get this.

The malicious request often comes after the user replies.

## Lesson 8 — SOC analysts document evidence, not assumptions

Use wording like:

- The email appears suspicious because...
- The likely goal is...
- If the user clicked...
- If credentials were entered...

Do not overstate what has not been proven.

---

# 18. Tools to Use in Future Real Phishing Investigations

For real phishing investigations, use safe and approved tools:

- Google Admin Toolbox Messageheader
- MXToolbox Email Header Analyzer
- VirusTotal
- URLScan.io
- AbuseIPDB
- WHOIS lookup
- Any.run / Hybrid Analysis for safe sandboxing if allowed
- Microsoft Defender portal if available
- Google Workspace Admin if available
- Email security gateway logs if available
- Identity provider sign-in logs

Important safety rule:

- Do not click suspicious links directly.
- Do not open suspicious attachments on your host machine.
- Use defanged URLs such as hxxps:// and [.] when documenting.
- Use sandbox/reputation tools when available.

---

# 19. Interview Translation

A strong way to explain Phase 7 in an interview:

In Phase 7 of my SOC lab, I practiced phishing investigation using simulated suspicious email reports. I reviewed five different phishing scenarios: Microsoft 365 credential phishing, invoice attachment malware delivery, CEO/BEC impersonation, fake Okta security alert phishing, and fake DocuSign document review phishing. For each case, I extracted IOCs, identified the impersonated brand or person, reviewed sender domains, URLs, attachments, social engineering techniques, likely attack goals, and wrote SOC-style verdicts and recommended actions. I also wrote a final phishing incident report summarizing key findings, IOCs, impact, and response actions.

Short version:

I investigated simulated phishing emails, extracted IOCs, identified credential harvesting, malware delivery, and BEC patterns, and wrote SOC-style phishing tickets and an incident report with recommended containment actions.

---

# 20. Phase 7 Completion Checklist

Completed:

- Phishing overview lesson completed
- IOC extraction practiced
- Sender domain analysis practiced
- URL analysis practiced
- Attachment risk analysis practiced
- Social engineering identification practiced
- Credential harvesting scenarios reviewed
- Malware delivery scenario reviewed
- BEC scenario reviewed
- Fake security alert scenario reviewed
- Fake document signing scenario reviewed
- 5 phishing investigation tickets completed
- Final phishing incident report completed
- Lessons learned documented
- Website/GitHub-ready log created

## Final Status

**Phase 7 complete.**
