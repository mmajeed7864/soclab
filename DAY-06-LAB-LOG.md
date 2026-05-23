# Day 06 Lab Log - Phase 6 Complete: SOC Ticket Writing, Escalation, and Shift Handoff

**Date:** May 23, 2026  
**Phase Completed:** Phase 6 - SOC Ticket Writing, Escalation, and Shift Handoff  
**Focus:** SOC ticket templates, severity reasoning, escalation summaries, shift handoff notes, and analyst documentation discipline

---

## Phase 6 Title

**Phase 6 - SOC Ticket Writing, Escalation, and Shift Handoff**

## Phase 6 Status

**Completed for website / portfolio documentation**

This phase focused on taking alerts and events from Phase 5 and turning them into SOC-style tickets, escalation summaries, severity decisions, and shift handoff notes. The goal was to practice writing like a SOC Analyst Level 1, not just reviewing logs.

Phase 5 answered:

```text
What happened?
Who ran it?
What command/process executed?
What parent process launched it?
Was it expected?
Should I close, investigate, or escalate?
```

Phase 6 answered:

```text
How do I document this clearly in a ticket?
How do I explain severity?
How do I justify the verdict?
How do I recommend next action?
How do I hand this off to another analyst?
```

---

# 1. Phase Overview

Phase 6 built on the high-volume triage work completed in Phase 5. Instead of only reviewing alerts in Kibana, this phase focused on writing clean SOC documentation.

The major skill practiced was converting raw evidence into professional analyst notes.

Example raw evidence:

```text
Sysmon Event ID 1
PowerShell launched whoami.exe
User: DESKTOP-3JKM5O9\mmajeed
Parent process: powershell.exe
```

SOC ticket version:

```text
A PowerShell session executed whoami.exe to identify the current user context. This activity is suspicious by technique because whoami is commonly used during discovery, but benign by context because it was generated during the lab activity loop.
```

This phase practiced:

- Alert review
- SOC ticket writing
- Escalation summary writing
- Severity selection
- Recommended action writing
- Shift handoff writing

---

# 2. Lab Environment Used

## Ubuntu SIEM VM

Role:

- Elasticsearch
- Kibana
- Fleet Server
- Elastic Agent management
- Main SIEM/search interface

Known host-only IP:

```text
192.168.56.101
```

## Windows 10 Victim VM

Role:

- Endpoint being monitored
- Generated Phase 5/Phase 6 events
- Sent Sysmon and Windows logs into Elastic

Known host-only IP:

```text
192.168.56.104
```

Hostname:

```text
DESKTOP-3JKM5O9
```

Known lab user:

```text
DESKTOP-3JKM5O9\mmajeed
```

## Tools Used

- Elastic / Kibana
- Kibana Discover
- Sysmon
- Windows Security logs
- Windows System logs
- Elastic Agent / Fleet
- Phase 5 activity loop events as ticket source material

---

# 3. Phase 6 Goal

The purpose of this phase was to create professional SOC-style documentation from the alerts/events reviewed during Phase 5.

Required deliverables completed for this log:

- 5 SOC tickets
- 3 escalation summaries
- 2 shift handoff examples
- Severity reasoning notes
- Ticket writing template
- Escalation writing template
- Lessons learned
- Website/GitHub-ready documentation

---

# 4. Key Concept: Triage vs Ticket vs Escalation

## Triage

Triage is first-look analysis.

It answers:

```text
What happened?
Who was involved?
What evidence exists?
Is this expected or suspicious?
```

Example:

```text
PowerShell launched whoami.exe under user DESKTOP-3JKM5O9\mmajeed. This is user discovery behavior. Expected in lab, suspicious if unexpected in production.
```

## SOC Ticket

A SOC ticket is a formal written record.

It answers:

```text
What happened?
What host/user was affected?
What evidence supports the analysis?
What was the verdict?
What should be done next?
```

## Escalation Summary

An escalation summary is shorter and written for L2 / IR / security engineering.

It answers:

```text
Here is the suspicious activity.
Here is why it matters.
Here is what I checked.
Here is what I recommend next.
```

## Shift Handoff

A shift handoff summarizes what happened during the analyst's shift so the next analyst can continue without losing context.

It answers:

```text
What did I review?
What did I close?
What is still open?
What should the next analyst watch?
```

---

# 5. SOC Ticket Template Used

```markdown
# Ticket X - <Alert Name>

## Severity
Low / Medium / High / Critical

## Status
Open / Closed as benign / False Positive / Escalated

## Alert Summary
<One to three sentences explaining what happened.>

## Affected Asset
Host:
User:
IP / Endpoint Role:

## Detection Source
Tool:
Log Source:
Event ID:

## Evidence
Command/process:
Parent process:
Parent user:
Process ID:
Parent Process ID:
Timestamp:
Relevant fields:

## Analysis
<Explain what the event means, why SOC cares, and whether the activity is expected or suspicious.>

## Verdict
Benign by context / Suspicious by technique / False positive / Escalate

## Recommended Action
<Close, monitor, tune, investigate further, or escalate.>
```

---

# 6. Severity Reasoning Model

A major part of Phase 6 was learning that severity is not based only on the command name. Severity comes from:

```text
Command + user + parent process + destination + frequency + impact + follow-on activity
```

## Simple Severity Model

```text
Low:
Normal/noisy activity, expected behavior, little risk.

Medium:
Suspicious technique, but no proof of compromise or system change.

High:
Risky action with possible compromise, privilege change, persistence, suspicious source, or malicious pattern.

Critical:
Confirmed compromise, active malware, ransomware, data theft, domain-wide impact, or production outage.
```

## Severity Questions

For each event, ask:

```text
1. What did the command/event do?
2. Who ran it?
3. Was the user expected to run it?
4. What parent process launched it?
5. Did anything dangerous happen after it?
6. Was the destination/domain suspicious?
7. Were there repeated attempts?
8. Was a privileged account involved?
9. Did the activity create persistence, modify privileges, or touch credentials?
```

## Severity Examples from Phase 6

| Event | Suggested Severity | Reason |
|---|---:|---|
| notepad.exe / calc.exe | Low | Expected GUI process activity |
| nslookup github.com | Low | Expected DNS lookup to normal domain |
| whoami from PowerShell | Medium | User discovery via PowerShell |
| curl.exe to GitHub | Medium | Command-line web request could be abused |
| net localgroup administrators | Medium | Privilege discovery |
| One failed logon from known user | Low | Likely user error |
| Multiple failed logons from unknown source | Medium/High | Possible brute force/password spraying |
| New user created | High | Possible persistence |
| User added to Administrators | High | Privilege escalation |
| New service installed | High | Possible persistence |
| LSASS access | High/Critical | Possible credential dumping |

## Key Lesson

Severity and verdict are different.

Example:

```text
Severity: Medium
Verdict: Benign by context
```

Meaning:

```text
The behavior is important enough to review, but after analysis it was authorized or expected.
```

---

# 7. SOC Ticket 1 - PowerShell User Discovery Command

## Severity

Medium

## Status

Closed as benign lab activity

## Alert Summary

A PowerShell session executed the `whoami.exe` command on the Windows 10 victim endpoint. The command was used to identify the current user context.

## Affected Asset

```text
Host: DESKTOP-3JKM5O9
User: DESKTOP-3JKM5O9\mmajeed
Endpoint Role: Windows 10 victim VM
```

## Detection Source

```text
Tool: Elastic / Kibana
Log Source: Microsoft-Windows-Sysmon/Operational
Event ID: Sysmon Event ID 1 - Process Creation
```

## Evidence

```text
Command/process: "C:\Windows\system32\whoami.exe"
Parent process: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -Command whoami
Parent user: DESKTOP-3JKM5O9\mmajeed
Process ID: 7980
Parent Process ID: 8272
Event type: Process creation
```

## Analysis

Sysmon Event ID 1 indicates that a process was created on the endpoint. In this event, `whoami.exe` was launched from a PowerShell session under the user `DESKTOP-3JKM5O9\mmajeed`.

The `whoami` command is commonly used to identify the current user context. This can be normal administrative or troubleshooting activity, but it is also commonly seen during attacker discovery after initial access. An attacker may use this command to confirm which account they are running as before checking privileges, enumerating the environment, or attempting privilege escalation.

In this lab, the activity was expected because it was generated during the Phase 5 scripted activity loop. No additional suspicious context was observed for this event.

## Verdict

```text
Benign by context / suspicious by technique
```

## Recommended Action

Close as authorized lab activity. In a production environment, review surrounding PowerShell activity, logon events, privilege-related events, and additional discovery commands before closing.

---

# 8. SOC Ticket 2 - Local Administrators Group Enumeration

## Severity

Medium

## Status

Closed as benign lab activity

## Alert Summary

A Windows command was executed to enumerate members of the local Administrators group on the Windows 10 victim endpoint. The activity was generated during the Phase 5 lab workflow.

## Affected Asset

```text
Host: DESKTOP-3JKM5O9
User: DESKTOP-3JKM5O9\mmajeed
Endpoint Role: Windows 10 victim VM
```

## Detection Source

```text
Tool: Elastic / Kibana
Log Source: Microsoft-Windows-Sysmon/Operational
Event ID: Sysmon Event ID 1 - Process Creation
```

## Evidence

```text
Command/process: "C:\Windows\system32\net1.exe" localgroup administrators
Parent process: "C:\Windows\system32\net.exe" localgroup administrators
Parent user: DESKTOP-3JKM5O9\mmajeed
Process ID: 9992
Parent Process ID: 1804
Event type: Process creation
```

## Analysis

Sysmon Event ID 1 indicates that a process was created on the endpoint. In this event, `net.exe` was used to run `localgroup administrators`, which enumerates members of the local Administrators group. Windows commonly invokes `net1.exe` as part of `net.exe` command execution, so seeing both `net.exe` and `net1.exe` is expected behavior.

This command is security-relevant because attackers may use it during discovery to identify which accounts have local administrator privileges. Knowing local admin membership can help an attacker determine whether they already have elevated access or which accounts may be valuable for privilege escalation or lateral movement.

In this lab, the activity was expected because it was generated during the Phase 5 scripted activity loop. No additional suspicious account creation, privilege escalation, or unauthorized access was observed with this event.

## Verdict

```text
Benign by context / suspicious by technique
```

## Recommended Action

Close as authorized lab activity. In a production environment, review whether the user was authorized to enumerate local administrators, check for nearby account creation or group membership changes, and correlate with logon events and other discovery commands.

---

# 9. SOC Ticket 3 - Command-Line Web Request to GitHub

## Severity

Medium

## Status

Closed as benign lab activity

## Alert Summary

A PowerShell session launched `curl.exe` to make an outbound HTTPS request to `github.com` from the Windows 10 victim endpoint. The activity occurred during the Phase 5 lab activity window.

## Affected Asset

```text
Host: DESKTOP-3JKM5O9
User: DESKTOP-3JKM5O9\mmajeed
Endpoint Role: Windows 10 victim VM
```

## Detection Source

```text
Tool: Elastic / Kibana
Log Source: Microsoft-Windows-Sysmon/Operational
Event ID: Sysmon Event ID 1 - Process Creation
```

## Evidence

```text
Command/process: "C:\Windows\system32\curl.exe" https://github.com
Parent process: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
Parent user: DESKTOP-3JKM5O9\mmajeed
Process ID: 9900
Parent Process ID: 8272
Event type: Process creation
```

## Analysis

Sysmon Event ID 1 indicates that a process was created on the endpoint. In this event, `curl.exe` was launched from PowerShell to make an outbound HTTPS request to GitHub.

Command-line web requests can be legitimate, especially for testing, troubleshooting, or downloading tools. However, this behavior is security-relevant because attackers may use `curl.exe`, PowerShell, or similar utilities to download payloads, contact external infrastructure, or transfer data.

In this lab, the activity was expected because it was generated during the Phase 5 scripted activity loop. The destination was GitHub, and no suspicious follow-on execution was observed in this ticket.

## Verdict

```text
Benign by context / suspicious by technique
```

## Recommended Action

Close as authorized lab activity. In a production environment, review the destination reputation, downloaded content if any, parent process, user context, and follow-on process execution before closing.

---

# 10. SOC Ticket 4 - DNS Lookup to GitHub Using nslookup

## Severity

Low

## Status

Closed as benign lab activity

## Alert Summary

A PowerShell session launched `nslookup.exe` to query DNS resolution for `github.com` from the Windows 10 victim endpoint. The activity occurred during the Phase 5 lab activity window and was expected.

## Affected Asset

```text
Host: DESKTOP-3JKM5O9
User: DESKTOP-3JKM5O9\mmajeed
Endpoint Role: Windows 10 victim VM
```

## Detection Source

```text
Tool: Elastic / Kibana
Log Source: Microsoft-Windows-Sysmon/Operational
Event ID: Sysmon Event ID 1 - Process Creation
```

## Evidence

```text
Command/process: "C:\Windows\system32\nslookup.exe" github.com
Parent process: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
Parent user: DESKTOP-3JKM5O9\mmajeed
Process ID: 1616
Parent Process ID: 8272
Event type: Process creation
```

## Analysis

Sysmon Event ID 1 indicates that a process was created on the endpoint. In this event, `nslookup.exe` was launched from PowerShell to query DNS resolution for `github.com`.

DNS lookup activity is commonly used for troubleshooting and connectivity testing. However, it can also be security-relevant because attackers may perform DNS lookups during reconnaissance, domain testing, or command-and-control validation. In this case, the queried domain was GitHub and the activity was expected as part of the Phase 5 scripted lab loop.

No suspicious domain, unusual user, abnormal parent process, or malicious follow-on activity was observed.

## Verdict

```text
Benign by context
```

## Recommended Action

Close as authorized lab activity. In a production environment, investigate further if the queried domain is suspicious, newly registered, associated with malware/phishing, or appears alongside other discovery or download activity.

---

# 11. SOC Ticket 5 - Local User Failed Logon

## Severity

Low

## Status

Closed as benign user activity

## Alert Summary

A failed logon event was observed for local user `DESKTOP-3JKM5O9\mmajeed` on the Windows 10 victim endpoint. Only one failed attempt was observed during the reviewed time window.

## Affected Asset

```text
Host: DESKTOP-3JKM5O9
User: DESKTOP-3JKM5O9\mmajeed
Endpoint Role: Windows 10 victim VM
```

## Detection Source

```text
Tool: Elastic / Kibana
Log Source: Windows Security
Event ID: Windows Security Event ID 4625 - Failed Logon
```

## Evidence

```text
Command/process: N/A
Parent process: N/A
Timestamp: May 22, 2026 @ 21:00:17.106
Relevant fields to review:
- TargetUserName
- TargetDomainName
- LogonType
- IpAddress
- WorkstationName
- Status
- SubStatus
- FailureReason
```

## Analysis

Windows Security Event ID 4625 indicates a failed logon attempt. In this event, the failed logon involved a known local user account on the Windows victim endpoint.

A single failed logon attempt from a known user is commonly caused by a mistyped password, expired credentials, or normal user error. No evidence of brute force activity was observed because there was only one failed attempt in the reviewed context.

In a production environment, this event would become more concerning if there were repeated failed attempts, multiple targeted users, an unknown source IP, a privileged account, failed logons followed by a successful logon, or activity occurring outside normal hours.

## Verdict

```text
Benign by context
```

## Recommended Action

Close as likely user error. Continue monitoring for repeated failed logons, unusual source IPs, privileged account targeting, or failed logons followed by successful authentication.

---

# 12. Example "Unknown / Production" SOC Ticket - Multiple Failed Logons

This ticket represents what the same 4625 event would look like if the context were worse.

## Severity

High if multiple attempts were observed or the source/user was unknown.

Medium if only a small number of attempts occurred without follow-on success.

## Status

Escalated for L2 review

## Alert Summary

Multiple failed logon attempts were observed on host `DESKTOP-3JKM5O9` during business hours. The attempted username was unknown or unexpected, which may indicate password guessing, account enumeration, or unauthorized access attempts.

## Detection Source

```text
Tool: Elastic / Kibana
Log Source: Windows Security
Event ID: Windows Security Event ID 4625 - Failed Logon
```

## Analysis

Windows Security Event ID 4625 indicates a failed logon attempt. Multiple failed logons from an unknown or unexpected user are security-relevant because they may indicate brute force activity, password spraying, account enumeration, or unauthorized login attempts.

This should be escalated if the attempts are repeated, originate from an unusual source IP, target privileged accounts, occur across multiple users, or are followed by a successful logon.

## Recommended Action

- Verify whether the attempted username is valid or expected.
- Review the source IP, workstation name, and logon type.
- Check whether the same source attempted other usernames.
- Check for a successful 4624 logon after the failed attempts.
- Review whether the targeted account is privileged.
- Consider blocking the source or locking the account if brute force behavior is confirmed.

---

# 13. Escalation Summary Template

```markdown
# Escalation Summary - <Alert Name>

## Summary
<What happened in 1-3 sentences.>

## Evidence Reviewed
- Host:
- User:
- Command:
- Parent process:
- Log source:
- Event ID:
- Related fields:

## Analyst Assessment
<Why this matters. Explain whether it is suspicious by technique, benign by context, or escalation-worthy.>

## Risk
Low / Medium / High / Critical

## Recommended L2 Actions
-
-
-
```

---

# 14. Escalation Summary 1 - Multiple Failed Logons From Unknown User

## Summary

Multiple failed logon attempts were observed on host `DESKTOP-3JKM5O9` during business hours. The attempted username was unknown or unexpected, which may indicate password guessing, account enumeration, or unauthorized access attempts.

## Evidence Reviewed

```text
Host: DESKTOP-3JKM5O9
User: Unknown / unexpected username
Command: N/A - authentication event
Parent process: N/A - authentication event
Log source: Windows Security
Event ID: 4625 - Failed Logon
Timestamp: May 22, 2026 @ 21:00:17.106
Related fields:
- TargetUserName
- LogonType
- IpAddress
- WorkstationName
- Status
- SubStatus
- FailureReason
```

## Analyst Assessment

Windows Security Event ID 4625 indicates a failed logon attempt. Multiple failed logons from an unknown or unexpected user are security-relevant because they may indicate brute force activity, password spraying, account enumeration, or unauthorized login attempts.

This should be escalated if the attempts are repeated, originate from an unusual source IP, target privileged accounts, occur across multiple users, or are followed by a successful logon.

## Risk

High if multiple attempts were observed or if the source/user is unknown.

Medium if only a small number of attempts occurred without follow-on success.

## Recommended L2 Actions

- Verify whether the attempted username is valid or expected.
- Review the source IP, workstation name, and logon type.
- Check whether the same source attempted other usernames.
- Check for a successful 4624 logon after the failed attempts.
- Review whether the targeted account is privileged.
- Consider blocking the source or locking the account if brute force behavior is confirmed.

---

# 15. Escalation Summary 2 - curl.exe Launched from PowerShell

## Summary

A local user executed `curl.exe` from PowerShell to connect to `https://github.com` on host `DESKTOP-3JKM5O9`. The activity should be reviewed because command-line web requests can be used for legitimate administration or suspicious activity such as downloading files, contacting external infrastructure, or transferring data.

## Evidence Reviewed

```text
Host: DESKTOP-3JKM5O9
User: DESKTOP-3JKM5O9\mmajeed
Command: "C:\Windows\system32\curl.exe" https://github.com
Parent process: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
Log source: Microsoft-Windows-Sysmon/Operational
Event ID: Sysmon Event ID 1 - Process Creation
Timestamp: May 22, 2026 @ 21:00:17.106
```

## Analyst Assessment

Sysmon Event ID 1 indicates process creation. In this event, `curl.exe` was launched from PowerShell to make an outbound HTTPS request to GitHub.

This activity is security-relevant because command-line web tools such as `curl.exe` can be used to download scripts, payloads, or tools, contact external infrastructure, or transfer data. The activity is not automatically malicious, but it should be reviewed if unexpected, repeated, associated with unknown domains, or followed by file creation or execution.

## Risk

Medium if the activity was unexpected, repeated, or involved downloads.

High if the command downloaded suspicious files, contacted a malicious domain, or was followed by execution.

## Recommended L2 Actions

- Verify whether the user was authorized to use `curl.exe`.
- Review how many times `curl.exe` was executed.
- Check the destination domain and reputation.
- Review whether any files were downloaded.
- Check for follow-on process execution after the curl command.
- Correlate with Sysmon Event ID 3 network connections and Sysmon Event ID 22 DNS queries.

---

# 16. Escalation Summary 3 - Local Administrators Group Enumeration

## Summary

A local user executed a command to enumerate members of the local Administrators group on host `DESKTOP-3JKM5O9`. This activity shows which accounts have local administrator privileges on the endpoint.

## Evidence Reviewed

```text
Host: DESKTOP-3JKM5O9
User: DESKTOP-3JKM5O9\mmajeed
Command: "C:\Windows\system32\net1.exe" localgroup administrators
Parent process: "C:\Windows\system32\net.exe" localgroup administrators
Log source: Microsoft-Windows-Sysmon/Operational
Event ID: Sysmon Event ID 1 - Process Creation
Timestamp: May 22, 2026 @ 21:00:17.106
```

## Analyst Assessment

Sysmon Event ID 1 indicates process creation. In this event, a local user executed a command to enumerate the local Administrators group. This command is commonly used for privilege discovery because it reveals which accounts have local administrator rights on the system.

The command does not modify the system by itself, but it is security-relevant because attackers may use it after gaining access to determine whether they have elevated privileges or to identify accounts that may be useful for privilege escalation or lateral movement.

This should be escalated if the command was unexpected, executed by an unusual user, followed by account creation, followed by a user being added to Administrators, associated with suspicious remote access, or seen with other discovery commands.

## Risk

Medium by default because the command performs privilege discovery but does not modify the system.

High if followed by:

- New user creation
- User added to Administrators
- Suspicious successful logon
- Remote access activity
- Privilege escalation behavior

## Recommended L2 Actions

- Verify whether the user was authorized to enumerate local administrator membership.
- Review how many times the command was executed.
- Check Windows Security events for user creation or group membership changes.
- Look for Event ID 4720 for new user creation.
- Look for Event ID 4732 for a user added to a local group.
- Review nearby successful logons and PowerShell activity.

---

# 17. Shift Handoff Note 1 - End-of-Shift Summary

## Shift Summary

During this shift, multiple SOC tickets were created from Phase 5 lab events using Elastic/Kibana and Sysmon telemetry. The reviewed alerts included PowerShell user discovery, local administrator group enumeration, command-line web requests, DNS lookup activity, and a failed Windows logon event.

## Environment

```text
SIEM: Elastic / Kibana
Endpoint: DESKTOP-3JKM5O9
User: DESKTOP-3JKM5O9\mmajeed
Log Sources:
- Microsoft-Windows-Sysmon/Operational
- Windows Security
```

## Tickets Created

```text
Ticket 1: PowerShell whoami - Closed as benign lab activity
Ticket 2: Local Administrators Group Enumeration - Closed as benign lab activity
Ticket 3: curl.exe to GitHub - Closed as benign lab activity
Ticket 4: nslookup GitHub - Closed as benign lab activity
Ticket 5: Failed Logon 4625 - Closed as benign user activity
```

## Escalation Summaries Created

```text
Escalation 1: Multiple Failed Logons From Unknown User
Escalation 2: curl.exe Launched from PowerShell
Escalation 3: Local Administrators Group Enumeration
```

## Findings

No confirmed malicious activity was identified. Most reviewed activity was generated by the Phase 5 scripted lab loop or represented expected Windows behavior.

Several activities were suspicious by technique but benign by context:

- PowerShell executing whoami
- curl.exe launched from PowerShell
- net localgroup administrators
- nslookup GitHub
- Windows failed logon event

## Open Items

No open incidents remain from this lab session.

## Recommended Follow-Up

Continue practicing SOC ticket writing with higher-risk scenarios such as:

- New user creation
- User added to Administrators
- Multiple failed logons followed by success
- Suspicious PowerShell download command
- New service installation
- LSASS process access

---

# 18. Shift Handoff Note 2 - Next Analyst Handoff

## Summary for Next Analyst

The Phase 6 ticket-writing exercise was completed using events generated from the Phase 5 scripted activity loop. The reviewed events were primarily benign lab-generated activity, but several were useful for practicing SOC documentation because they map to common attacker techniques such as discovery, privilege discovery, and command-line web requests.

## Reviewed Activity

```text
PowerShell whoami:
Reviewed as user discovery. Closed as benign lab activity.

net localgroup administrators:
Reviewed as local admin enumeration. Closed as benign lab activity.

curl.exe to GitHub:
Reviewed as command-line web request. Closed as benign lab activity.

nslookup GitHub:
Reviewed as DNS lookup activity. Closed as benign lab activity.

4625 failed logon:
Reviewed as failed authentication. Closed as likely user error due to single attempt and known user context.
```

## Watch Items

If similar activity appears in a production environment, the following conditions should increase severity:

```text
Unknown user
Repeated attempts
Privileged account targeted
Suspicious source IP
Unknown domain
Payload download
Follow-on process execution
New account created
User added to Administrators
Remote login after failures
```

## Recommended Next Steps

Move into the next roadmap phase after documenting Phase 6. Future practice should include creating tickets for higher-risk alerts and writing escalation summaries where actual escalation is justified.

---

# 19. What Was Missed / What To Revisit Later

This phase was completed for website/portfolio documentation, but these are useful items to revisit for additional practice:

## 1. More Realistic High-Severity Tickets

Future tickets should include:

- User account created
- User added to Administrators
- New service installed
- Suspicious PowerShell download command
- Repeated failed logons followed by success
- LSASS access
- Suspicious outbound connection to unknown IP

## 2. Better Timestamp Collection

Some tickets used example timestamps from available events. Future practice should capture exact timestamps from each event.

## 3. More Source IP Review

For Windows Security events like 4625, future practice should include:

- Source IP
- Workstation name
- Logon type
- Status/SubStatus
- Failure reason

## 4. More Correlation

Future tickets should correlate:

- Sysmon Event ID 1 process creation
- Sysmon Event ID 3 network connection
- Sysmon Event ID 22 DNS query
- Windows Security 4624/4625 logons
- Windows Security 4720/4732 account/group changes

## 5. More Realistic Escalation Decisions

Most events in this lab were benign by context. Future practice should include scenarios where escalation is clearly required.

---

# 20. Lessons Learned

## Lesson 1 - Evidence must match the alert

One mistake during practice was accidentally copying curl evidence into a local administrators enumeration summary. This reinforced that every ticket must have evidence that matches the alert.

## Lesson 2 - Do not claim unauthorized unless proven

The log may show that a command executed, but it does not prove the user lacked permission. Better wording:

```text
Unexpected or not yet validated as authorized
```

Instead of:

```text
Unauthorized
```

## Lesson 3 - Do not overstate what the log proves

Example:

A log showing `nslookup github.com` proves the command executed. It does not prove the DNS lookup succeeded unless the result is reviewed.

## Lesson 4 - Severity needs evidence

High severity requires stronger evidence such as:

- Repeated failed logons
- Unknown source
- Privileged account targeted
- User added to admin group
- Payload downloaded
- Confirmed malware
- Follow-on execution

## Lesson 5 - Command/process and parent process are different

Example:

```text
Command/process: whoami.exe
Parent process: powershell.exe
```

The command/process tells what ran. The parent tells what launched it.

## Lesson 6 - Windows Security events are not Sysmon events

Example:

```text
4625 = Windows Security failed logon
```

Not:

```text
Microsoft-Windows-Sysmon/Operational
```

## Lesson 7 - Escalation summaries should be concise

A good escalation summary quickly explains:

```text
What happened
Why it matters
What was checked
What L2 should do next
```

---

# 21. Interview Translation

In Phase 6, I practiced converting raw SIEM events into SOC-style tickets, escalation summaries, and shift handoff notes. I used Elastic/Kibana to review Sysmon and Windows Security events from my Windows 10 victim endpoint, then documented the alerts using a structured ticket format.

I created tickets for PowerShell user discovery, local administrator group enumeration, command-line web requests using curl, DNS lookup activity using nslookup, and Windows failed logon events. For each ticket, I identified the affected host, user, event source, event ID, command line, parent process, severity, verdict, and recommended action.

I also practiced severity reasoning by distinguishing between low, medium, and high-risk activity based on context. For example, a single failed logon from a known user was low severity, while repeated failed logons from an unknown source would be medium or high. I learned to avoid overclaiming and to write what the evidence actually supports.

This phase helped me understand how SOC analysts communicate findings, justify decisions, document false positives, and escalate events that require deeper investigation.

---

# 22. Phase 6 Completion Checklist

Completed:

- SOC ticket template created
- 5 polished SOC tickets completed
- Severity reasoning practiced
- 3 escalation summaries completed
- 2 shift handoff notes completed
- False-positive/benign context documented
- Evidence matching reviewed
- Windows Security vs Sysmon distinction practiced
- Recommended actions written
- Website/GitHub-ready Phase 6 documentation created

## Final Status

**Phase 6 complete.**
