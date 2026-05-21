# Day 04 Lab Log - Windows, Linux, and Network Log Fundamentals

**Date:** May 21, 2026  
**Phase Completed:** Phase 4 - Windows, Linux, and Network Log Fundamentals  
**Focus:** SOC Analyst L1 log interpretation, KQL searching, raw field review, and investigation fundamentals

---

## Phase Overview

Phase 4 focused on learning how to read and understand the core logs that SOC analysts use during real investigations. After building the lab infrastructure in earlier phases, this phase shifted from tool setup to log interpretation.

The goal was to understand what common Windows, Sysmon, Linux, and network logs mean, which fields matter, why SOC analysts care about them, and how those logs help answer basic investigation questions.

Main workflow:

```text
Windows 10 Victim VM -> Sysmon / Windows Event Logs -> Elastic Agent -> Elasticsearch -> Kibana Discover
Ubuntu SIEM VM -> Linux auth/service logs -> terminal review with grep, tail, and journalctl
Windows 10 Victim VM -> network activity -> Sysmon Event ID 3 and 22 -> Kibana Discover
```

By the end of this phase, I reviewed and documented 20+ important log and event scenarios that map directly to SOC Analyst L1 investigation work.

---

## Environment Used

- Oracle VirtualBox
- Ubuntu SIEM VM
  - Elasticsearch
  - Kibana
  - Fleet Server
  - Suricata from the earlier phase
  - Host-only IP: `192.168.56.101`
- Windows 10 Victim VM
  - Sysmon installed with the SwiftOnSecurity config
  - Elastic Agent enrolled through Fleet
  - Host-only IP: `192.168.56.104`
- Wazuh Server VM
  - Powered off during most of Phase 4 to save resources
- Main analysis interface
  - Kibana Discover
  - Data view: `logs-*`
  - Time ranges used: Last 24 hours / Last 15 minutes

---

## Phase Goal

Create a personal log reference sheet for SOC Analyst work. For each important log or event, understand:

- What the event means
- Where the log comes from
- Which fields matter
- What activity generated the log
- Why a SOC analyst would care
- How to search for it in Kibana
- How it connects to suspicious activity or alert triage

---

## Log And Event Scenarios Reviewed

### 1. Sysmon Event ID 1 - Process Creation

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "1"
```

Sysmon Event ID 1 shows when a process starts on the Windows endpoint. Important fields included `winlog.event_data.Image`, `winlog.event_data.CommandLine`, `winlog.event_data.ParentImage`, `winlog.event_data.ParentCommandLine`, `winlog.event_data.User`, and `host.name`.

**SOC value:** This is one of the most important endpoint logs for investigating PowerShell, `cmd.exe`, suspicious scripts, malware execution, persistence, and parent-child process chains.

### 2. net.exe / net1.exe User And Group Commands

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "1" and winlog.event_data.CommandLine : *net*
```

Observed activity included `net user`, `net localgroup administrators`, `net.exe`, and `net1.exe`.

**SOC value:** Attackers often abuse built-in Windows tools to enumerate accounts, create users, add users to groups, and check administrator membership. Analysts should review command line, parent process, user context, and timing.

### 3. Sysmon Event ID 11 - File Created

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "11"
```

This event shows when a file is created on disk. Important fields included `winlog.event_data.TargetFilename`, `winlog.event_data.Image`, `winlog.event_data.User`, and `host.name`.

**SOC value:** Useful for investigating malware drops, suspicious scripts, persistence files, files written to temp/public folders, and attacker staging activity.

### 4. Sysmon Event ID 3 - Network Connection

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "3"
```

Generated activity included `ping 8.8.8.8`, `nslookup google.com`, `nslookup github.com`, `curl.exe https://example.com`, and `curl.exe https://github.com`.

**SOC value:** Helps identify which process connected to which IP, port, and protocol. Useful for suspicious outbound connections, malware C2, unusual ports, and process-to-network correlation.

### 5. Sysmon Event ID 22 - DNS Query

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "22"
```

Generated activity included `nslookup google.com`, `nslookup github.com`, and `nslookup microsoft.com`.

**SOC value:** DNS logs help identify what domains an endpoint attempted to resolve. Useful for phishing domains, malware C2, suspicious downloads, and domain reputation review.

### 6. Windows Security Event ID 4624 - Successful Logon

```kql
winlog.channel : "Security" and event.code : "4624"
```

Important fields included `TargetUserName`, `LogonType`, `IpAddress`, `WorkstationName`, and `TargetDomainName`.

Useful logon types:

- `2` = Interactive logon
- `3` = Network logon
- `10` = RemoteInteractive / RDP-style logon
- `11` = CachedInteractive

**SOC value:** Helps determine who accessed a system, how they logged in, and whether the login method matches expected behavior.

### 7. Windows Security Event ID 4625 - Failed Logon

```kql
winlog.channel : "Security" and event.code : "4625"
```

**SOC value:** Failed logons are used to detect brute force attempts, password spraying, mistyped credentials, invalid accounts, and unauthorized access attempts.

### 8. Windows Security Event ID 4672 - Special Privileges Assigned

```kql
winlog.channel : "Security" and event.code : "4672"
```

**SOC value:** Shows when an account logs in with administrative or special privileges. This matters when correlated with suspicious logins, new accounts, group changes, or unusual process execution.

### 9. Windows Security Event ID 4720 - User Account Created

```kql
winlog.channel : "Security" and event.code : "4720"
```

Generated activity example:

```powershell
net user phase4user P@ssw0rd123 /add
```

**SOC value:** Attackers may create local accounts for persistence. Analysts should check who created the account, when it was created, and what happened before and after.

### 10. Windows Security Event ID 4732 - User Added To Local Group

```kql
winlog.channel : "Security" and event.code : "4732"
```

Generated activity example:

```powershell
net localgroup administrators phase4user /add
```

**SOC value:** Important for privilege escalation and persistence investigations because attackers often add accounts to local administrator groups.

### 11. Windows Security Event ID 4726 - User Account Deleted

```kql
winlog.channel : "Security" and event.code : "4726"
```

**SOC value:** Can represent normal cleanup, but it can also show attacker cleanup after creating a temporary persistence account.

### 12. Windows Security Event ID 4740 - Account Locked Out

```kql
winlog.channel : "Security" and event.code : "4740"
```

**SOC value:** Account lockouts may indicate brute force attempts, password spraying, repeated failed logins, or user error.

### 13. Windows System Event ID 7045 - New Service Installed

```kql
winlog.channel : "System" and event.code : "7045"
```

**SOC value:** Attackers commonly create services for persistence, execution, or malware installation. Analysts should review the service name, executable path, user context, and whether the service is expected.

### 14. Sysmon Event ID 7 - Image Loaded

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "7"
```

**SOC value:** Useful for spotting suspicious DLL loading, unsigned modules, DLL hijacking, or unusual libraries loaded by trusted processes.

### 15. Sysmon Event ID 10 - Process Access

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "10"
```

High-value LSASS search:

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "10" and winlog.event_data.TargetImage : "*lsass.exe"
```

**SOC value:** Important for detecting credential dumping, LSASS access, process injection, and suspicious process tampering.

### 16. Sysmon Event ID 13 - Registry Value Set

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "13"
```

Common suspicious registry areas include `Run`, `RunOnce`, `Winlogon`, `Services`, and `Policies`.

**SOC value:** Attackers often modify registry keys for persistence, disabling security tools, startup execution, or changing system behavior.

### 17. Linux SSH Authentication Logs

```bash
sudo grep sshd /var/log/auth.log | tail -n 30
```

**SOC value:** Helps identify successful and failed remote login attempts, source IPs, target users, and authentication patterns.

### 18. Linux sudo Usage Logs

```bash
sudo grep sudo /var/log/auth.log | tail -n 30
```

**SOC value:** Useful for auditing privileged activity, reviewing administrative commands, and identifying possible privilege escalation behavior.

### 19. Linux Service Logs With journalctl

```bash
sudo journalctl -u ssh --no-pager | tail -n 30
sudo journalctl -u elasticsearch --no-pager | tail -n 30
sudo journalctl -u kibana --no-pager | tail -n 30
```

**SOC value:** Helps troubleshoot whether important services are running, crashing, restarting, or throwing errors.

### 20. Network Log Fundamentals

Reviewed fields:

- Source IP
- Destination IP
- Source port
- Destination port
- Protocol
- DNS query
- User agent
- IDS signature

**SOC value:** Network logs help analysts determine who connected to what, when, over which protocol, and from which process.

### 21. Suricata / IDS Alert Fields

Log locations:

```bash
/var/log/suricata/fast.log
/var/log/suricata/eve.json
```

Important fields included alert signature, source IP, destination IP, source port, destination port, protocol, rule ID / SID, and severity.

**SOC value:** IDS alerts provide evidence for triage and help analysts correlate network activity with endpoint logs.

---

## Issues Encountered And Troubleshooting

### ECS Fields Were Blank

Some Elastic ECS fields like `process.name` and `process.command_line` showed `-` in Kibana.

**Fix:** I used raw Sysmon fields instead:

- `winlog.event_data.Image`
- `winlog.event_data.CommandLine`
- `winlog.event_data.ParentImage`
- `winlog.event_data.ParentCommandLine`
- `winlog.event_data.User`

**Lesson learned:** Not all logs are perfectly normalized into ECS fields. SOC analysts need to know how to inspect raw event fields when normalized fields are missing.

### Specific Test File Did Not Appear In Sysmon Event ID 11

A test file was created at:

```text
C:\Users\Public\phase4-file-test.txt
```

The exact file did not clearly show under Sysmon Event ID 11.

**Fix / troubleshooting:**

- Confirmed Sysmon Event ID 11 logs existed.
- Added `winlog.event_data.TargetFilename` as a column.
- Tried searches using `phase4`, `Public`, `Users`, and `Temp`.
- Confirmed other Event ID 11 file creation events were present.
- Moved on after validating the event type with real file-created events.

**Lesson learned:** Not every test event appears exactly as expected depending on Sysmon config filtering. The correct SOC approach is to validate the log type, understand the fields, and avoid getting stuck on one exact example.

### KQL Wildcard Searching Was Picky

Some wildcard searches for file paths did not return expected results.

**Fix:**

- Broadened searches.
- Removed strict path filters.
- Searched by `event.code` first.
- Added fields as columns to manually inspect values.
- Used raw fields instead of assumed field names.

**Lesson learned:** When a query does not work, broaden the search, inspect raw fields, and then narrow down.

### Too Many Events / Noisy Results

Sysmon Event ID 11 produced many noisy Microsoft .NET file creation events.

**Fix:** I used columns to identify what the events represented and focused on understanding field meaning instead of treating every event as suspicious.

**Lesson learned:** SOC analysts must separate normal system noise from suspicious activity. High event volume is normal.

### Needed To Move Faster Through Event IDs

Once the pattern was understood, event IDs were reviewed in groups of three instead of one-by-one.

**Lesson learned:** Once the investigation pattern is learned, analysts can move faster by applying the same logic to new event types.

---

## Key Takeaways

- Sysmon Event ID 1 is one of the most valuable logs for process execution investigations.
- `net.exe` and `net1.exe` are important to monitor because they can be used for user/group enumeration and account manipulation.
- Sysmon Event ID 3 helps connect processes to network destinations.
- Sysmon Event ID 22 helps identify domain lookups made by processes.
- Windows Security logs are essential for authentication and account-change investigations.
- Linux `/var/log/auth.log` is important for SSH, sudo, and authentication monitoring.
- `journalctl` is useful for checking service health and troubleshooting security infrastructure.
- Network fields such as source IP, destination IP, destination port, protocol, and DNS query are core SOC investigation fields.
- Not every field will be normalized perfectly, so raw fields matter.
- Troubleshooting logs requires broad searches, field inspection, and narrowing down from there.

---

## Interview Translation

In Phase 4, I practiced reading and interpreting common Windows, Sysmon, Linux, and network logs used in SOC investigations. I reviewed Sysmon process creation, network connection, file creation, DNS query, image load, process access, and registry modification events.

I also reviewed Windows Security events for successful and failed logons, privileged logons, account creation, account deletion, local group modification, account lockouts, and service installation. On the Linux side, I reviewed SSH authentication logs, sudo usage logs, and service logs using `auth.log`, `grep`, `tail`, and `journalctl`.

This phase helped me understand how SOC analysts use logs to answer questions like what happened, when it happened, what host was involved, what user was involved, what process ran, what network connection occurred, and whether the activity looks suspicious.

---

## Phase 4 Completion Status

Completed:

- Windows/Sysmon Event ID review
- Windows Security Event ID review
- Linux authentication log review
- Linux sudo log review
- Linux service log review
- Network/DNS log review
- IDS field review
- 20+ log/event examples documented
- Troubleshooting notes documented
- Interview explanation prepared

Phase 4 is complete.
