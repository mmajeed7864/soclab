# Phase 9 Detailed Lab Log — Detection Engineering and Rule Tuning

## Status
**Completed — May 30, 2026**

## Completion Summary

Phase 9 moved my SOC home lab from manual investigations into detection
engineering. After building Splunk searches and dashboards in Phase 8, I
converted useful investigation ideas into scheduled alerts, safely generated
controlled endpoint behavior, confirmed which telemetry sources could support
each rule, documented severity and false positives, and tuned a noisy network
detection so normal OneDrive activity did not dominate the results.

### Final Deliverables

| Requirement | Result |
|---|---|
| Practical detections | 6 enabled Splunk alerts |
| Sigma-style rules | 3 documented rules |
| Tuning write-up | 1 completed before/after analysis |
| Detection platform | Splunk Enterprise |
| Endpoint telemetry used | Sysmon and Windows Security logs |
| Honest deferrals documented | LSASS access and Certutil remote download |

The central lesson was that a detection is not useful simply because it returns
events. A practical rule must be supported by available telemetry, validated
safely, interpreted with context, and tuned so verified benign behavior does
not create unnecessary analyst work.

---

# 1. Objective and Detection Workflow

Phase 8 focused on investigation:

```text
Event happens → Search evidence → Analyze context → Assign verdict
```

Phase 9 focused on repeatable detection logic:

```text
Select behavior → Confirm telemetry → Generate safe test activity
→ Detect evidence → Save alert → Tune noise → Document tradeoff
```

Roadmap requirement:

```text
6 detections
3 Sigma-style rules
1 tuning write-up with before/after logic
```

---

# 2. Platform and Telemetry Foundation

| Item | Configuration |
|---|---|
| SIEM | Splunk Enterprise 10.4.0 |
| Splunk server | `splunk-soc` |
| Splunk server host-only IP | `192.168.56.102` |
| Receiver port | TCP `9997` |
| Endpoint forwarder | Splunk Universal Forwarder |
| Windows endpoint | `DESKTOP-3JKM5O9` |

## Sysmon Availability Check

Before chasing planned rule ideas, I inventoried the Sysmon Event IDs already
visible in Splunk.

| Sysmon Event ID | Meaning | Available |
|---:|---|---|
| `1` | Process Creation | Yes |
| `3` | Network Connection | Yes |
| `11` | File Creation | Yes |
| `12` | Registry Object Create/Delete | Yes |
| `13` | Registry Value Set | Yes |
| `22` | DNS Query | Yes |
| `10` | Process Access | No |

Windows Security telemetry was also available, including Event ID `4732` for
members added to a security-enabled local group.

This inventory prevented a telemetry rabbit hole: LSASS process-access
detection would require Event ID `10`, which was not available in the current
dataset, so it was documented and replaced with a supported persistence rule.

---

# 3. Phase-Blocking Performance Troubleshooting

## Problem

At the beginning of Phase 9, the Windows Victim VM became extremely slow while
running with the Splunk server. CPU reached `100%`, making detection testing
painfully slow.

## Evidence

| Check | Result |
|---|---|
| Windows Victim RAM | 8 GB |
| Initial Windows Victim vCPU | 2 |
| C: drive free space | Approximately 17.26 GB |
| `Sysmon64` | Running / Automatic |
| `SplunkForwarder` | Running / Automatic |
| Host logical processors | 8 |
| Host physical RAM | Approximately 31.8 GB |

## Fix Applied

Phase 9 required Sysmon and the Splunk Forwarder, but did not require Elastic
Agent or Wazuh Agent to be running on the endpoint during Splunk detection
testing. I temporarily stopped those non-required services and set them to
Manual while preserving the required telemetry path.

```powershell
Stop-Service -Name "Elastic Agent" -Force -ErrorAction SilentlyContinue
Stop-Service -Name WazuhSvc -Force -ErrorAction SilentlyContinue
Set-Service -Name "Elastic Agent" -StartupType Manual -ErrorAction SilentlyContinue
Set-Service -Name WazuhSvc -StartupType Manual -ErrorAction SilentlyContinue
```

Required services remained active:

```text
Sysmon64           Running / Automatic
SplunkForwarder    Running / Automatic
```

I then powered down the Windows Victim cleanly and increased its allocation
from 2 to 4 vCPUs in VirtualBox.

## Result

| Metric | Before | After |
|---|---:|---:|
| Windows Victim vCPU | 2 | 4 |
| CPU behavior | Pinned at 100% / unusable | Became usable; approximately 41% during validation |
| Memory usage | Not blocking | Healthy, approximately 33% |
| Phase 9 work | Blocked | Resumed |

CPU still fluctuated during normal Windows background activity, but the issue
no longer blocked detection work. Additional optimization was intentionally
deferred as low ROI.

---

# 4. Alert Configuration Standard

Each validated detection was saved in Splunk as a private scheduled alert using
a consistent configuration:

| Alert Setting | Value |
|---|---|
| Permissions | Private |
| Alert Type | Scheduled |
| Schedule | Hourly |
| Lookback | Last 60 minutes |
| Trigger Condition | Number of Results is greater than 0 |
| Trigger Mode | Once |
| Trigger Action | Add to Triggered Alerts |

A final screenshot of the Splunk Alerts page showed all six alerts listed as
enabled.

---

# 5. DET-001 — Encoded PowerShell Execution

## Why It Matters

Encoded PowerShell can hide the readable intent of a command line. It is a
common behavior worth investigating, although administrators and automation
may also use it legitimately.

## Data Source

```text
Sysmon Event ID 1 — Process Creation
```

## Safe Controlled Test

I encoded a harmless command that printed a test message:

```powershell
$cmd = 'Write-Output "Phase9 Encoded PowerShell Detection Test"'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($cmd)
$encoded = [Convert]::ToBase64String($bytes)
powershell.exe -NoProfile -EncodedCommand $encoded
```

The test safely printed:

```text
Phase9 Encoded PowerShell Detection Test
```

## Detection SPL

```spl
index=main host=DESKTOP-3JKM5O9 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| search SysmonEventID=1
| rex field=_raw "<Data Name=[\"']User[\"']>(?<User>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']Image[\"']>(?<Image>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']CommandLine[\"']>(?<CommandLine>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']ParentImage[\"']>(?<ParentImage>[^<]+)</Data>"
| search Image="*powershell.exe" (CommandLine="*-EncodedCommand*" OR CommandLine="*-enc *")
| table _time, User, Image, ParentImage, CommandLine
| sort - _time
```

## Result and Verdict

Splunk matched one recent event from
`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` under the known
lab user.

| Item | Decision |
|---|---|
| Alert | `DET-001 Encoded PowerShell Execution` |
| Severity | Medium |
| Lab verdict | Benign by context — controlled harmless test |
| False positives | Admin scripts, automation, security tooling |
| MITRE ATT&CK | T1059.001 — PowerShell |

---

# 6. DET-002 — Registry Run Key Persistence

## Why It Matters

Run and RunOnce registry values may automatically launch software at logon.
Malware often abuses them for persistence, while legitimate software also uses
them, making contextual review important.

## Data Source

```text
Sysmon Event ID 13 — Registry Value Set
```

## Safe Controlled Test

I briefly created a harmless startup value pointing to Notepad, then deleted it
before it could execute at login:

```powershell
$runKey = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
New-Item -Path $runKey -Force | Out-Null
New-ItemProperty -Path $runKey -Name "Phase9Test" -Value "C:\Windows\System32\notepad.exe" -PropertyType String -Force
Start-Sleep -Seconds 5
Remove-ItemProperty -Path $runKey -Name "Phase9Test" -Force
```

## Detection SPL

The final rule was deliberately broadened beyond the test value name:

```spl
index=main host=DESKTOP-3JKM5O9 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| search SysmonEventID=13
| rex field=_raw "<Data Name=[\"']User[\"']>(?<User>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']Image[\"']>(?<Image>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']TargetObject[\"']>(?<TargetObject>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']Details[\"']>(?<Details>[^<]+)</Data>"
| search TargetObject="*\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\\*" OR TargetObject="*\\Software\\Microsoft\\Windows\\CurrentVersion\\RunOnce\\*"
| table _time, User, Image, TargetObject, Details
| sort - _time
```

## Result and Verdict

Splunk matched the temporary `Phase9Test` value, the Notepad path, and the
PowerShell process used to create it.

| Item | Decision |
|---|---|
| Alert | `DET-002 Registry Run Key Persistence` |
| Severity | Medium |
| Lab verdict | Benign by context — value created then removed |
| False positives | Approved startup apps, installers, update agents |
| MITRE ATT&CK | T1547.001 — Registry Run Keys / Startup Folder |

---

# 7. DET-003 — Scheduled Task Creation via Schtasks

## Why It Matters

Scheduled tasks are used legitimately, but attackers can create tasks for
persistence or repeated execution.

## Data Source

```text
Sysmon Event ID 1 — Process Creation
```

## Safe Controlled Test

I created a task scheduled for a future date that would launch Notepad, verified
it, and removed it immediately:

```powershell
schtasks.exe /Create /TN "Phase9TestTask" /TR "C:\Windows\System32\notepad.exe" /SC ONCE /ST 23:59 /SD 12/31/2026 /F
schtasks.exe /Query /TN "Phase9TestTask" /FO LIST
schtasks.exe /Delete /TN "Phase9TestTask" /F
```

## Ingestion Troubleshooting

The first Splunk search displayed zero results even though Windows confirmed
task creation and deletion. After allowing more time for ingestion, Splunk
displayed the matching `schtasks.exe /Create` process events. I treated this as
indexing delay and did not waste time changing telemetry.

## Detection SPL

```spl
index=main host=DESKTOP-3JKM5O9 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| search SysmonEventID=1
| rex field=_raw "<Data Name=[\"']User[\"']>(?<User>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']Image[\"']>(?<Image>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']CommandLine[\"']>(?<CommandLine>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']ParentImage[\"']>(?<ParentImage>[^<]+)</Data>"
| search Image="*\\schtasks.exe" CommandLine="*/Create*"
| table _time, User, Image, ParentImage, CommandLine
| sort - _time
```

## Result and Verdict

| Item | Decision |
|---|---|
| Alert | `DET-003 Scheduled Task Creation via Schtasks` |
| Severity | Medium |
| Lab verdict | Benign by context — future harmless task removed |
| False positives | IT automation, software updates, backup tools |
| MITRE ATT&CK | T1053.005 — Scheduled Task/Job: Scheduled Task |

---

# 8. DET-004 — New Local Administrator Activity

## Why It Matters

Adding a user to the local Administrators group grants privileged access. This
can indicate privilege escalation or persistence and is higher risk than a
generic suspicious technique.

## Data Source

```text
Windows Security Event ID 4732 — A member was added to a security-enabled local group
```

## Safe Controlled Test

I created a temporary local account with a randomly generated in-memory
password, added it to Administrators, then removed and deleted it:

```powershell
$testUser = "Phase9TempAdmin"
$testPassword = ConvertTo-SecureString ("P9!" + [Guid]::NewGuid().ToString("N") + "aA1") -AsPlainText -Force
New-LocalUser -Name $testUser -Password $testPassword -Description "Temporary Phase 9 detection test account" | Out-Null
Add-LocalGroupMember -Group "Administrators" -Member $testUser
Start-Sleep -Seconds 5
Remove-LocalGroupMember -Group "Administrators" -Member $testUser
Remove-LocalUser -Name $testUser
```

The temporary account and its administrator membership did not remain on the
endpoint after testing.

## Ingestion Troubleshooting

The narrow Event ID `4732` query initially returned zero events. A broader
Security-log search later returned the account-management sequence, including
`4726` user deletion, `4733` removal from Administrators, and `4732` addition
to Administrators. I then isolated the key behavior in the final detection.

## Detection SPL

```spl
index=main host=DESKTOP-3JKM5O9 sourcetype="WinEventLog:Security" EventCode=4732 Group_Name="Administrators"
| table _time, EventCode, SubjectUserName, Member_Name, Group_Name, _raw
| sort - _time
```

## Result and Verdict

The Splunk result showed Event ID `4732` with message text confirming that a
member was added to the local `Administrators` group. The raw evidence and
surrounding account-management events tied it to the controlled test sequence.

| Item | Decision |
|---|---|
| Alert | `DET-004 New Local Administrator Activity` |
| Severity | High |
| Lab verdict | Benign by context — temporary account cleaned up |
| False positives | Authorized admin changes, endpoint provisioning |
| MITRE ATT&CK | T1098 — Account Manipulation |

---

# 9. DET-005 — Unusual Outbound Network Connection

## Why It Matters

Outbound network connections can represent ordinary software behavior or
malicious communications. A broad rule can be too noisy to be useful, making
this detection ideal for a tuning exercise.

## Data Source

```text
Sysmon Event ID 3 — Network Connection
```

## Initial Controlled Test and Pivot

I successfully ran a controlled connection from the Windows Victim to the local
Splunk server:

```powershell
curl.exe -I http://192.168.56.102:8000
```

The endpoint received a Splunk Web HTTP redirect response, proving the
connection occurred. The targeted `curl.exe` Event ID 3 search did not match
that test. Rather than force a Sysmon configuration change, I used the live
Event ID 3 evidence already present: current OneDrive public HTTPS activity.

This pivot preserved honesty and directly created a useful false-positive
tuning exercise.

## Untuned Baseline SPL

```spl
index=main host=DESKTOP-3JKM5O9 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| search SysmonEventID=3
| rex field=_raw "<Data Name=[\"']Image[\"']>(?<Image>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']DestinationIp[\"']>(?<DestinationIp>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']DestinationPort[\"']>(?<DestinationPort>[^<]+)</Data>"
| where NOT cidrmatch("10.0.0.0/8",DestinationIp)
    AND NOT cidrmatch("172.16.0.0/12",DestinationIp)
    AND NOT cidrmatch("192.168.0.0/16",DestinationIp)
    AND DestinationIp!="127.0.0.1"
| stats count as Connections values(DestinationIp) as DestinationIPs values(DestinationPort) as DestinationPorts by Image
| sort - Connections
```

## Baseline Result

| Process | Count | Context |
|---|---:|---|
| `OneDrive.exe` | 19 | Expected Microsoft cloud sync traffic |
| `OneDriveStandaloneUpdater.exe` | 19 | Expected Microsoft update traffic |

```text
Untuned count: 38
```

## Tuning Pass 1

After excluding `OneDrive.exe`, 18 results remained from
`OneDriveStandaloneUpdater.exe`.

## Final Tuned SPL

```spl
index=main host=DESKTOP-3JKM5O9 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| search SysmonEventID=3
| rex field=_raw "<Data Name=[\"']User[\"']>(?<User>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']Image[\"']>(?<Image>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']DestinationIp[\"']>(?<DestinationIp>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']DestinationHostname[\"']>(?<DestinationHostname>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']DestinationPort[\"']>(?<DestinationPort>[^<]+)</Data>"
| where NOT cidrmatch("10.0.0.0/8",DestinationIp)
    AND NOT cidrmatch("172.16.0.0/12",DestinationIp)
    AND NOT cidrmatch("192.168.0.0/16",DestinationIp)
    AND DestinationIp!="127.0.0.1"
| where NOT like(Image,"%\\Microsoft\\OneDrive\\OneDrive.exe")
    AND NOT like(Image,"%\\Microsoft\\OneDrive\\OneDriveStandaloneUpdater.exe")
| eval TriageReason="Unexpected process communicating with public destination"
| table _time, User, Image, DestinationHostname, DestinationIp, DestinationPort, TriageReason
| sort - _time
```

## Tuning Outcome

| Stage | Results | Decision |
|---|---:|---|
| Broad public-IP connection rule | 38 | Too noisy; expected OneDrive activity |
| Exclude `OneDrive.exe` | 18 | Still noisy; updater remained |
| Exclude both verified OneDrive executables | 0 | Known benign noise suppressed |

## Result and Verdict

| Item | Decision |
|---|---|
| Alert | `DET-005 Unusual Outbound Network Connection` |
| Severity | Medium |
| Validation type | Before/after false-positive tuning |
| Known benign exclusions | Full-path OneDrive and OneDrive updater executables |
| Detection tradeoff | Better signal quality, but allowlists require future review |
| MITRE ATT&CK | T1071.001 — Web Protocols |

---

# 10. DET-006 — Command-Line DNS Query Activity

## Why It Matters

Command-line tools generating DNS requests can be normal, but may also occur
during scripting, discovery, download staging, or suspicious execution. This
rule is intended for contextual review rather than automatic high-severity
escalation.

## Data Source

```text
Sysmon Event ID 22 — DNS Query
```

## Detection SPL

```spl
index=main host=DESKTOP-3JKM5O9 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| search SysmonEventID=22
| rex field=_raw "<Data Name=[\"']User[\"']>(?<User>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']Image[\"']>(?<Image>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']QueryName[\"']>(?<QueryName>[^<]+)</Data>"
| eval Image=replace(Image,"&lt;","<")
| eval Image=replace(Image,"&gt;",">")
| search Image="*\\curl.exe" OR Image="*\\powershell.exe" OR Image="*\\nslookup.exe"
| eval TriageReason="Command-line process generated DNS query - review domain and surrounding execution context"
| table _time, User, Image, QueryName, TriageReason
| sort - _time
```

## Validation Result

Splunk matched existing controlled lab evidence:

```text
User:      DESKTOP-3JKM5O9\mmajeed
Image:     C:\Windows\System32\curl.exe
QueryName: github.com
```

The validation search was initially run across all time to prove the data
existed. The alert was saved with a last-60-minutes scheduled lookback so old
lab history would not repeatedly generate new alert results.

## Result and Verdict

| Item | Decision |
|---|---|
| Alert | `DET-006 Command-Line DNS Query Activity` |
| Severity | Low / Medium |
| Lab verdict | Benign by context — controlled GitHub lookup |
| False positives | Admin testing, scripts, troubleshooting |
| Analyst use | Correlate command-line process activity with domain context |

---

# 11. Deferred Original Detection Concepts

## LSASS Process Access — Deferred

### Planned Detection
Unexpected process access to `lsass.exe`, potentially indicating credential
dumping.

### Expected Data Source
```text
Sysmon Event ID 10 — Process Access
```

### Evidence and Decision
A Splunk query for Event ID `10` targeting LSASS returned no events. A Sysmon
inventory confirmed Event ID `10` was not present in the dataset.

```text
Status: Deferred — current telemetry cannot validate this detection honestly.
```

Rather than altering Sysmon solely for this test, I completed a supported
persistence detection using Event ID `13`.

## Certutil Remote Download Activity — Deferred

### Planned Detection
`certutil.exe` with remote URL-cache/download arguments.

### Evidence
`certutil.exe` itself was functional, confirmed by hashing a local Windows
file. However, a harmless retrieval attempt from `https://example.com/`
returned `Access is denied`; no test file was created, and no matching
remote-download event appeared in Splunk.

```text
Status: Deferred — endpoint blocked live validation and no searchable detection evidence existed.
```

Rather than start a policy/security-control troubleshooting detour, I completed
the required alert set with validated command-line DNS telemetry.

---

# 12. Three Sigma-Style Rules

## Rule 1 — Encoded PowerShell Execution

```yaml
title: Encoded PowerShell Execution
id: phase9-det-001-encoded-powershell
status: experimental
description: Detects PowerShell execution using encoded command parameters.
author: Mohammed H. Majeed
date: 2026-05-30
logsource:
  product: windows
  category: process_creation
detection:
  selection_image:
    Image|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
  selection_commandline:
    CommandLine|contains:
      - '-EncodedCommand'
      - '-enc '
  condition: selection_image and selection_commandline
falsepositives:
  - Legitimate administrative automation
  - Software deployment scripts
  - Security tooling or monitoring activity
level: medium
tags:
  - attack.execution
  - attack.t1059.001
```

## Rule 2 — Registry Run Key Persistence

```yaml
title: Registry Run Key Persistence
id: phase9-det-002-registry-run-key-persistence
status: experimental
description: Detects registry value modifications beneath Windows Run or RunOnce startup locations.
author: Mohammed H. Majeed
date: 2026-05-30
logsource:
  product: windows
  category: registry_set
detection:
  selection:
    TargetObject|contains:
      - '\Software\Microsoft\Windows\CurrentVersion\Run\'
      - '\Software\Microsoft\Windows\CurrentVersion\RunOnce\'
  condition: selection
falsepositives:
  - Approved startup applications
  - Software installers or update agents
  - Enterprise logon utilities
level: medium
tags:
  - attack.persistence
  - attack.t1547.001
```

## Rule 3 — Scheduled Task Creation via Schtasks

```yaml
title: Scheduled Task Creation via Schtasks
id: phase9-det-003-scheduled-task-creation
status: experimental
description: Detects scheduled task creation using schtasks.exe with the /Create argument.
author: Mohammed H. Majeed
date: 2026-05-30
logsource:
  product: windows
  category: process_creation
detection:
  selection_image:
    Image|endswith:
      - '\schtasks.exe'
  selection_commandline:
    CommandLine|contains:
      - '/Create'
  condition: selection_image and selection_commandline
falsepositives:
  - Approved IT automation
  - Software installation and update tasks
  - Backup or maintenance tooling
level: medium
tags:
  - attack.persistence
  - attack.execution
  - attack.t1053.005
```

---

# 13. Detection and Tuning Summary

| Alert | Data Source | Severity | Validation Evidence | Status |
|---|---|---:|---|---|
| DET-001 Encoded PowerShell Execution | Sysmon 1 | Medium | Harmless encoded PowerShell matched | Enabled |
| DET-002 Registry Run Key Persistence | Sysmon 13 | Medium | Temporary Notepad Run-key value matched and removed | Enabled |
| DET-003 Scheduled Task Creation via Schtasks | Sysmon 1 | Medium | Future Notepad task matched and deleted | Enabled |
| DET-004 New Local Administrator Activity | Security 4732 | High | Temporary admin membership matched and cleaned up | Enabled |
| DET-005 Unusual Outbound Network Connection | Sysmon 3 | Medium | Benign OneDrive tuning: 38 → 18 → 0 | Enabled |
| DET-006 Command-Line DNS Query Activity | Sysmon 22 | Low/Medium | Controlled `curl.exe` → `github.com` DNS evidence matched | Enabled |

---

# 14. SOC Skills Practiced

- Detection engineering in Splunk
- SPL alert creation and scheduled rule configuration
- Sysmon process-creation detection
- Sysmon registry persistence detection
- Sysmon network-connection analysis
- Sysmon DNS query analysis
- Windows Security local administrator group monitoring
- Safe controlled behavior generation and cleanup
- MITRE ATT&CK mapping
- Sigma-style portable rule documentation
- False-positive reduction and allowlist tradeoff analysis
- Telemetry validation before building detections
- Honest deferral of unsupported detection paths
- Scope-controlled troubleshooting under active lab constraints

---

# 15. Interview Translation

## Portfolio / Resume Summary

Built six enabled Splunk detection alerts using Sysmon and Windows Security
telemetry, including encoded PowerShell, registry Run-key persistence,
scheduled-task creation, local administrator membership changes, unusual
outbound connections, and command-line DNS queries. Generated and cleaned up
safe controlled test behaviors, documented three Sigma-style rules with MITRE
ATT&CK mappings, and completed false-positive tuning that reduced validated
benign OneDrive network matches from 38 to zero in the tested window.

## Strong Interview Talking Points

### How was Phase 9 different from Phase 8?
Phase 8 focused on finding and investigating endpoint events with SPL and
building dashboards. Phase 9 converted those investigation ideas into scheduled
alert rules, validated them with safe test behavior, and tuned known benign
noise so the detections became more useful.

### What was the strongest tuning example?
A broad public outbound-connection search returned 38 expected OneDrive and
OneDrive updater connections. I validated the benign source processes, excluded
their full known paths in two passes, and reduced the known-benign results to
zero in the test window.

### Did every originally planned detection work?
No. LSASS process access could not be validated because Sysmon Event ID 10 was
not present in the ingested telemetry. A harmless Certutil remote-download
attempt was blocked and generated no searchable detection evidence. I documented
both honestly and completed the deliverable using supported, validated data
rather than forcing configuration changes.

### Why is local administrator membership high severity?
Adding an account to the Administrators group grants privileged access directly.
Even though my temporary test account was removed and deleted, the equivalent
behavior on a production endpoint requires prompt validation.

---

# 16. Completion Checklist

| Deliverable | Status |
|---|---|
| Six enabled Splunk alerts visible in final Alerts page | Complete |
| DET-001 Encoded PowerShell | Complete |
| DET-002 Registry Run Key Persistence | Complete |
| DET-003 Scheduled Task Creation via Schtasks | Complete |
| DET-004 New Local Administrator Activity | Complete |
| DET-005 Tuned Unusual Outbound Connection | Complete |
| DET-006 Command-Line DNS Query Activity | Complete |
| Sigma-style Rule 1 — Encoded PowerShell | Complete |
| Sigma-style Rule 2 — Registry Run Key Persistence | Complete |
| Sigma-style Rule 3 — Scheduled Task Creation | Complete |
| Before/after false-positive tuning write-up | Complete |
| LSASS validation limitation documented | Complete |
| Certutil validation limitation documented | Complete |
| Test cleanup and troubleshooting documented | Complete |

---

# 17. Final Phase 9 Outcome

Phase 9 transformed the lab from a search-and-dashboard environment into a
detection-engineering environment. I created six enabled Splunk alerts from
working endpoint evidence, wrote portable Sigma-style logic, performed safe test
activity with cleanup, handled ingestion delay without unnecessary changes, and
tuned a noisy network rule based on verified benign behavior.

The most important professional takeaway was:

```text
Validate the data source → Generate controlled behavior → Detect evidence
→ Save repeatable alert → Identify noise → Tune responsibly → Document honestly
```

Phase 9 is complete. The next roadmap milestone is Phase 10: building an Active
Directory environment for identity-focused monitoring and investigation.
