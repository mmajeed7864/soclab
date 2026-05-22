# Day 05 Lab Log - Phase 5 Complete

## Scripted Alert Loop, High-Volume SOC Triage, Escalation Notes, and Shift Handoff

**Status:** Phase 5 Complete  
**Date:** May 22, 2026  
**Focus:** SOC Analyst L1 first-look triage, alert classification, false-positive decisions, escalation notes, and shift handoff documentation.

> Roadmap note: the roadmap references VMware because the planned repeatable version of this lab will run as a VMware attacker/admin loop. This completed run was performed in Oracle VirtualBox using the same SOC workflow: generate repeatable endpoint telemetry, review it in Elastic/Kibana, classify events, and document the analyst decision process.

---

## 1. Phase Overview

Phase 5 moved the lab from tool setup into analyst workflow practice.

Previous phases built the monitoring environment:

- Phase 1: virtual lab foundation
- Phase 2: Elastic SIEM, Kibana, Fleet, Sysmon, Suricata, and Elastic Agent
- Phase 3: Wazuh XDR, Windows agent, FIM, SCA, and vulnerability detection
- Phase 4: Windows, Linux, Sysmon, and network log fundamentals

Phase 5 used that infrastructure to simulate a SOC alert queue. A scripted activity loop was executed on the Windows 10 victim endpoint to generate endpoint telemetry. The resulting events were reviewed in Kibana Discover using a SOC-style triage process.

Main workflow practiced:

```text
Alert/Event appears -> Identify source -> Review user/process/parent/destination -> Determine context -> Decide benign/suspicious/escalate -> Document clearly
```

This phase was not about proving every event was malicious. It was about practicing how an analyst separates normal activity, false positives, suspicious techniques, and escalation-worthy activity.

---

## 2. Lab Environment

### Ubuntu SIEM VM

Role:

- Elastic/Kibana SIEM server
- Fleet Server
- Elasticsearch data store
- Kibana analyst interface
- Suricata IDS from previous phase

Known host-only IP:

```text
192.168.56.101
```

Services involved:

```text
elasticsearch
kibana
elastic-agent
```

### Windows 10 Victim VM

Role:

- Endpoint being monitored
- Generated the Phase 5 activity loop
- Sent logs to Elastic through Elastic Agent

Known host-only IP:

```text
192.168.56.104
```

Hostname:

```text
DESKTOP-3JKM5O9
```

User:

```text
DESKTOP-3JKM5O9\mmajeed
```

Telemetry sources:

- Windows Security logs
- Windows System logs
- Sysmon logs
- Elastic Agent / Fleet managed log shipping

### Wazuh Server VM

The Wazuh VM was intentionally powered off for most of Phase 5 to conserve laptop resources.

Reason:

```text
Phase 5 focused on Elastic/Kibana high-volume triage, not Wazuh alerting.
```

---

## 3. Learning Objectives

By completing this phase, I practiced:

- Generating repeatable alert/event activity in a lab
- Reviewing high-volume Sysmon logs in Kibana
- Searching for specific commands and processes
- Identifying user context
- Distinguishing user activity from SYSTEM/background service activity
- Reading parent process fields
- Determining whether activity was expected or suspicious
- Classifying alerts as benign, false positive, suspicious, or escalation-worthy
- Writing SOC-style triage notes
- Writing escalation-style notes
- Writing a shift handoff summary
- Troubleshooting SIEM service issues
- Building documentation suitable for GitHub/portfolio review

---

## 4. Key SOC Mindset

### Suspicious by technique, benign by context

This became the main phrase of the phase.

Some commands are commonly used by both administrators and attackers:

- `whoami`
- `hostname`
- `ipconfig`
- `net user`
- `net localgroup administrators`
- `nslookup`
- `curl`
- `powershell.exe`
- `cmd.exe`
- outbound HTTPS connections
- Windows service logons

These are not automatically malicious. They matter because attackers use them for discovery, privilege checks, environment awareness, connectivity testing, payload download, persistence checks, and lateral movement preparation.

Correct analyst mindset:

```text
The behavior may be suspicious by technique, but the final verdict depends on context.
```

Context includes:

- user
- host
- command line
- parent process
- process path
- destination
- timing
- surrounding events
- whether the action was expected

---

## 5. Pre-Phase Issue: Kibana Would Not Open

### Problem

At the start of Phase 5, Kibana would not open from the browser.

Kibana and Elastic Agent appeared to be running, but the web interface would not load properly.

### Troubleshooting

Elasticsearch was checked:

```bash
sudo systemctl status elasticsearch
```

Elasticsearch was failing, so logs were reviewed:

```bash
sudo tail -n 120 /var/log/elasticsearch/elasticsearch.log
sudo grep -i "error\|exception\|fatal\|failed" /var/log/elasticsearch/elasticsearch.log | tail -n 50
```

Important error:

```text
AccessDeniedException: /etc/elasticsearch/service_tokens
Failed to load service_tokens file
```

### Root Cause

Elasticsearch could not read its own `service_tokens` file due to incorrect ownership/permissions. Kibana could be running, but Kibana could not function because Elasticsearch was down.

### Fix Applied

```bash
sudo chown root:elasticsearch /etc/elasticsearch/service_tokens
sudo chmod 660 /etc/elasticsearch/service_tokens

sudo chown -R root:elasticsearch /etc/elasticsearch
sudo find /etc/elasticsearch -type d -exec chmod 750 {} \;
sudo find /etc/elasticsearch -type f -exec chmod 660 {} \;

sudo systemctl restart elasticsearch
sudo systemctl status elasticsearch --no-pager

sudo systemctl restart kibana
sudo systemctl status kibana --no-pager
```

### Result

Elasticsearch came back online and Kibana became accessible again.

### Lesson Learned

This was a realistic SOC/NOC troubleshooting scenario. A service appeared to be up, but the backend dependency was down. The fix required checking service status, reading logs, identifying an access denied error, correcting Linux file permissions, and restarting affected services.

---

## 6. Scripted Activity Loop

### Purpose

The Phase 5 loop generated many safe but security-relevant events on the Windows 10 victim endpoint.

The goal was to simulate a SOC alert queue with enough activity to practice filtering, searching, classifying, and documenting events.

### Script Folder

```text
C:\SOC-Lab\Phase5
```

### Script File

```text
phase5-loop.ps1
```

### Script Creation

```powershell
mkdir C:\SOC-Lab\Phase5
notepad C:\SOC-Lab\Phase5\phase5-loop.ps1
```

### Script Content

```powershell
Write-Host "Starting Phase 5 SOC alert generation loop..."

for ($i = 1; $i -le 10; $i++) {

    Write-Host "Iteration $i"

    whoami
    hostname
    ipconfig

    nslookup google.com
    nslookup github.com
    nslookup microsoft.com

    curl.exe https://example.com
    curl.exe https://github.com

    Start-Process notepad.exe
    Start-Sleep -Seconds 2
    Start-Process calc.exe
    Start-Sleep -Seconds 2

    echo "Phase 5 test file $i" > "C:\Users\Public\phase5-test-$i.txt"
    Start-Sleep -Seconds 2
    Remove-Item "C:\Users\Public\phase5-test-$i.txt" -Force

    net user
    net localgroup administrators

    powershell.exe -Command "Get-Process | Select-Object -First 5"

    Start-Sleep -Seconds 10
}

Write-Host "Phase 5 loop complete."
```

### Execution

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
C:\SOC-Lab\Phase5\phase5-loop.ps1
```

### Activity Generated

The loop generated:

- process creation events
- PowerShell execution
- DNS lookups
- command-line web requests
- network connections
- GUI application launches
- file creation/deletion activity
- user/account enumeration
- local administrator group enumeration
- host/network discovery commands

---

## 7. Kibana Searches Used During Triage

### Broad Windows Host Search

```kql
agent.name : "DESKTOP-3JKM5O9"
```

### Sysmon Process Creation

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "1"
```

### Sysmon Network Connections

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "3"
```

### Sysmon DNS Queries

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "22"
```

### PowerShell Activity

```kql
winlog.event_data.CommandLine : *powershell*
```

### whoami Activity

```kql
winlog.event_data.CommandLine : *whoami*
```

### nslookup Activity

```kql
winlog.event_data.CommandLine : *nslookup*
```

### curl Activity

```kql
winlog.event_data.CommandLine : *curl*
```

### net Commands

```kql
winlog.event_data.CommandLine : *net*
```

### Local Administrator Group Enumeration

```kql
winlog.event_data.CommandLine : *localgroup*
```

### HTTPS Traffic

```kql
winlog.event_data.DestinationPort : 443
```

### Successful Windows Logons

```kql
winlog.channel : "Security" and event.code : "4624"
```

---

## 8. Triage Structure Used

For each event, I used this structure:

```text
Event:
User:
Command/process:
Parent:
Decision:
Reason:
```

Expanded triage questions:

1. What triggered?
2. Who ran it?
3. What process or command ran?
4. What parent process launched it?
5. Was the activity expected?
6. Is the activity suspicious by technique?
7. Is it benign by context?
8. Should it be closed, investigated, tuned, or escalated?

---

## 9. Important Fields Used

### User Field

```text
winlog.event_data.User
```

Example:

```text
DESKTOP-3JKM5O9\mmajeed
```

### Command Line Field

```text
winlog.event_data.CommandLine
```

Example:

```text
"C:\Windows\system32\whoami.exe"
```

### Image Field

```text
winlog.event_data.Image
```

Example:

```text
C:\Windows\System32\curl.exe
```

### Parent Image Field

```text
winlog.event_data.ParentImage
```

Example:

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

### Event Code

```text
event.code
```

Examples:

```text
Sysmon 1 = Process Creation
Sysmon 3 = Network Connection
Sysmon 22 = DNS Query
Windows Security 4624 = Successful Logon
```

---

## 10. User Context vs SYSTEM Context

A key lesson was understanding the difference between human user activity and Windows system activity.

Human/lab user:

```text
DESKTOP-3JKM5O9\mmajeed
```

Windows system account:

```text
NT AUTHORITY\SYSTEM
```

If an event shows:

```text
ParentUser: NT AUTHORITY\SYSTEM
ParentImage: C:\Windows\system32\services.exe
```

This usually means Windows service/background activity.

SOC lesson: SYSTEM activity is often normal, but attackers can also attempt to run malicious processes as SYSTEM. Analysts must check process name, command line, parent process, file path, user context, and surrounding events.

---

## 11. Parent Process vs Command Process

Example:

```text
Command/process: C:\Windows\system32\whoami.exe
Parent process: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

Meaning:

```text
PowerShell launched whoami.exe.
```

SOC lesson: the command/process tells what ran. The parent process tells where it came from.

Suspicious parent-child examples to remember:

```text
PowerShell -> whoami.exe
PowerShell -> curl.exe
cmd.exe -> net.exe
Office app -> PowerShell
browser -> unknown executable
services.exe -> unknown binary
```

---

## 12. Completed Triage Examples

### Example 1 - net localgroup administrators

- Event: Sysmon Event ID 1, Process Creation
- Command: `net localgroup administrators`
- Meaning: displays members of the local Administrators group
- Why SOC cares: attackers use it for privilege discovery
- Decision: benign by context / suspicious by technique
- Reason: expected lab activity; in production, review authorization and nearby account/group changes

### Example 2 - PowerShell whoami

- Event: Sysmon Event ID 1
- User: `DESKTOP-3JKM5O9\mmajeed`
- Command: `whoami.exe`
- Parent: PowerShell
- Why SOC cares: attackers use it after access to confirm context
- Decision: benign by context / suspicious by technique

### Example 3 - nslookup github.com

- Event: Sysmon Event ID 1
- Command: `nslookup github.com`
- Why SOC cares: DNS lookups can be troubleshooting or reconnaissance
- Decision: benign by context; review unusual domains in production

### Example 4 - curl.exe to github.com

- Event: Sysmon Event ID 1
- Command: `curl.exe https://github.com`
- Parent: PowerShell
- Why SOC cares: command-line web tools can download payloads or contact infrastructure
- Decision: benign in lab; investigate if unexpected or followed by execution

### Example 5 - HTTPS Network Connection on Port 443

- Event: Sysmon Event ID 3
- Destination port: 443
- Process example: OneDrive.exe or curl.exe depending on event
- Why SOC cares: malware and legitimate apps both use HTTPS
- Decision: close/tune if known process and destination; investigate unknown process/destination

### Example 6 - Windows Successful Logon 4624

- Event: Windows Security Event ID 4624
- Logon Type: 5
- Meaning: service logon
- Decision: close as benign unless account or surrounding activity is suspicious

### Additional Examples Reviewed

- PowerShell parent process launching child processes
- SYSTEM/services.exe activity
- nslookup google.com
- calc.exe process creation
- notepad.exe process creation
- curl.exe to example.com
- powershell.exe Get-Process
- ipconfig execution
- hostname execution

---

## 13. False Positive / Benign Activity List

Events documented as benign or false-positive-style in lab context:

1. PowerShell `whoami`
2. `net user`
3. `net localgroup administrators`
4. `curl.exe https://github.com`
5. `curl.exe https://example.com`
6. `nslookup google.com`
7. `nslookup github.com`
8. `nslookup microsoft.com`
9. `notepad.exe` process creation
10. `calc.exe` process creation
11. OneDrive HTTPS connection
12. Windows service logon type 5
13. SYSTEM/service process activity
14. `ipconfig` command
15. `hostname` command

---

## 14. Escalation-Style Notes

### Escalation Note 1 - PowerShell User Discovery

PowerShell executed `whoami` on DESKTOP-3JKM5O9. This can indicate post-compromise discovery if unexpected.

Evidence:

- User: `DESKTOP-3JKM5O9\mmajeed`
- Command: `whoami.exe`
- Parent: PowerShell
- Event ID: Sysmon 1

Decision: benign in lab / investigate in production if unexpected.

### Escalation Note 2 - Local Administrator Group Enumeration

`net localgroup administrators` reveals accounts with local administrator rights. Attackers may use this for privilege discovery.

Decision: benign by context / suspicious by technique.

### Escalation Note 3 - Command-Line Web Request

`curl.exe` was used to connect to GitHub. Command-line web requests can be used to download payloads, scripts, or tools.

Decision: benign in lab / investigate if unexpected.

### Escalation Note 4 - HTTPS Network Connection

Port 443 is common, but malware often uses HTTPS for C2 or data transfer. Process and destination determine risk.

Decision: benign/tune if known process and destination; investigate unknown process or destination.

### Escalation Note 5 - Successful Service Logon

Windows Security 4624 with Logon Type 5 usually indicates service activity.

Decision: benign/tune unless unexpected account or suspicious surrounding activity exists.

---

## 15. Shift Handoff Summary

### Analyst

Mohammed Majeed

### Environment

Elastic/Kibana SIEM with Windows 10 victim endpoint running Sysmon and Elastic Agent.

### Shift Summary

A scripted Phase 5 activity loop was executed on the Windows 10 victim host to generate high-volume endpoint telemetry for SOC triage practice. The loop produced process creation events, PowerShell activity, DNS lookups, command-line web requests, local account/group enumeration, file activity, GUI process launches, and network connections.

Kibana initially failed to load because Elasticsearch was down. Elasticsearch logs showed an `AccessDeniedException` related to `/etc/elasticsearch/service_tokens`. The file ownership and permissions were corrected, Elasticsearch and Kibana were restarted, and SIEM access was restored.

Events reviewed:

- PowerShell whoami execution
- nslookup github.com DNS lookup
- curl.exe request to github.com
- net localgroup administrators enumeration
- HTTPS connection on port 443
- OneDrive outbound HTTPS activity
- Windows 4624 service logon
- SYSTEM/services.exe background activity
- calc.exe process creation
- notepad.exe process creation
- ipconfig execution
- hostname execution

Findings:

No confirmed malicious activity was identified. Most events were intentionally generated by the Phase 5 lab script or represented normal Windows background/service activity.

Security-relevant by technique:

- PowerShell execution
- whoami user discovery
- net localgroup administrators enumeration
- curl command-line web requests
- DNS lookup activity
- HTTPS outbound traffic

Escalation status:

```text
No events required escalation in the lab environment.
```

Recommended follow-up:

- Convert the best triage examples into formal SOC tickets in Phase 6
- Continue practicing event classification
- Continue reviewing parent-child process relationships
- Continue using suspicious-by-technique, benign-by-context decision language
- Practice identifying when similar activity would become escalation-worthy

---

## 16. Lessons Learned

1. Same event type, different risk. Sysmon Event ID 1 only means a process was created. Risk depends on user, command, parent, path, and context.
2. Context is everything. `whoami`, `curl`, and `net` are not automatically malicious.
3. User context matters. `DESKTOP-3JKM5O9\mmajeed` is the lab user. `NT AUTHORITY\SYSTEM` is system/service activity.
4. Parent process matters. A benign-looking command can become suspicious with an unusual parent process.
5. SOC analysts do not escalate everything. The job is to identify what is benign, noisy, tunable, suspicious, or escalation-worthy.
6. Troubleshooting is part of SOC work. A broken SIEM can stop investigations, so service health and log review matter.

---

## 17. Interview Translation

In Phase 5, I practiced SOC Analyst first-look triage using my Elastic/Kibana SIEM and Windows 10 victim endpoint. I generated a high volume of safe endpoint activity with a PowerShell loop and reviewed the resulting Sysmon and Windows events in Kibana. I practiced identifying the user, command line, parent process, host, event source, and context for each event.

I triaged PowerShell activity, whoami discovery commands, nslookup DNS lookups, curl web requests, local administrator group enumeration, HTTPS network connections, Windows service logons, GUI process launches, and SYSTEM background activity. I classified events as benign, false positive, suspicious, or escalation-worthy based on context.

I also troubleshot a SIEM outage when Elasticsearch failed due to permissions on the `service_tokens` file. I reviewed service status and Elasticsearch logs, identified the access denied error, corrected ownership and file permissions, and restored Kibana access.

This phase helped me practice the real SOC workflow of reviewing alerts, collecting evidence, understanding context, making first-look decisions, documenting false positives, and creating shift handoff notes.

---

## 18. Completion Checklist

Completed:

- Phase 5 activity loop created
- High-volume endpoint events generated
- Elastic/Kibana restored after service issue
- Sysmon process creation triage practiced
- PowerShell triage practiced
- curl/web request triage practiced
- nslookup/DNS triage practiced
- net/localgroup enumeration triage practiced
- Windows 4624 service logon reviewed
- SYSTEM vs user context understood
- Parent process vs command/process understood
- False positive/benign list created
- Escalation-style notes created
- Shift handoff summary created
- SOC-style decision language practiced
- Website/GitHub-ready writeup created

## Final Status

**Phase 5 complete.**