# Day 02 Lab Log - Phase 2 Complete: Elastic SIEM, Fleet, Sysmon, Windows Telemetry, and Suricata IDS

**Date:** May 19, 2026  
**Phase Completed:** Phase 2 - Elastic SIEM + Windows Endpoint Telemetry + Suricata IDS  
**Focus:** Elasticsearch, Kibana, Fleet Server, Elastic Agent, Sysmon, Windows Event Logs, Suricata IDS, Kibana Discover, dashboards, and troubleshooting  
**Primary SIEM Host IP:** `192.168.56.101`  
**Windows Victim Host:** `DESKTOP-3JKM5O9`

---

## Phase 2 Title

**Phase 2 - Elastic SIEM + Windows Endpoint Telemetry + Suricata IDS**

## Phase 2 Status

**Completed**

Phase 2 was the first major security tooling phase of the home SOC lab. Phase 1 created the virtual lab foundation. Phase 2 turned that foundation into a working SIEM pipeline by installing and configuring Elastic/Kibana, Fleet, Elastic Agent, Sysmon, Windows log collection, and Suricata IDS.

This phase is one of the most important parts of the roadmap because it creates the core SOC workflow:

```text
Endpoint activity -> agent collection -> SIEM storage -> analyst search -> dashboard/investigation
```

This writeup is intentionally detailed so other aspiring SOC analysts can understand not just what was installed, but why each component matters and how the pieces fit together.

---

# 1. Purpose of Phase 2

The goal of Phase 2 was to build a working SIEM pipeline using Elastic.

A SIEM, or Security Information and Event Management platform, is where security teams collect, search, correlate, and investigate logs. In a real SOC, analysts review activity from endpoints, servers, firewalls, cloud systems, identity providers, EDR tools, IDS sensors, and business applications.

In this lab, Elastic/Kibana acted as the SIEM.

Phase 2 focused on collecting telemetry from a Windows 10 endpoint and preparing the environment for later alert triage, ticket writing, rule tuning, and incident response practice.

Main goals:

- Install Elasticsearch.
- Install Kibana.
- Configure Kibana access from the host machine.
- Configure Fleet Server.
- Enroll the Windows 10 victim with Elastic Agent.
- Install Sysmon on the Windows victim.
- Collect Windows Security/System/Application logs.
- Collect Sysmon endpoint telemetry.
- Validate logs in Kibana Discover.
- Install and validate Suricata IDS.
- Create the first SOC-focused Kibana dashboard.
- Troubleshoot missing logs, enrollment issues, and integration problems.
- Understand how endpoint and network logs flow into a SIEM.

---

# 2. High-Level Architecture

The Phase 2 endpoint telemetry pipeline looked like this:

```text
Windows 10 Victim VM
    |
    |-- Windows Event Logs
    |-- Sysmon Operational Logs
    |
Elastic Agent
    |
Fleet Policy
    |
Elasticsearch
    |
Kibana Discover / Dashboards
```

Suricata added the network IDS layer:

```text
Network traffic
    |
Suricata IDS
    |
eve.json / fast.log
    |
Elastic/Kibana review
```

Endpoint telemetry helps answer:

```text
What happened on the machine?
What process ran?
Who ran it?
What command line was used?
What parent process launched it?
What DNS query happened?
What network connection was made?
```

Network telemetry helps answer:

```text
What traffic occurred?
Which IPs communicated?
Which ports and protocols were used?
Did an IDS alert trigger?
Was there suspicious DNS, HTTP, TLS, or scanning behavior?
```

---

# 3. Lab Environment Used

## Ubuntu SIEM VM

Role:

- Elasticsearch server
- Kibana web interface
- Fleet Server
- Elastic management node
- Suricata IDS host

Known host-only IP:

```text
192.168.56.101
```

Purpose:

```text
Collect, store, manage, and display security logs.
```

## Windows 10 Victim VM

Role:

- Endpoint being monitored
- Windows log source
- Sysmon telemetry source
- Elastic Agent endpoint

Known host-only IP used later:

```text
192.168.56.104
```

Known host name:

```text
DESKTOP-3JKM5O9
```

Purpose:

```text
Generate endpoint activity and send logs into Elastic.
```

## Kali VM

Role:

- Testing/attacker machine for later phases
- Not heavily used in early Phase 2

Purpose later:

```text
Generate scans, safe attack traffic, and controlled IDS/SIEM events.
```

---

# 4. Tools Installed or Configured

## Elasticsearch

Elasticsearch is the backend data store. It stores indexed log data so Kibana can search it.

SOC analogy:

```text
Elasticsearch is the database where security events live.
```

## Kibana

Kibana is the web interface used to search, filter, visualize, and investigate logs.

SOC analogy:

```text
Kibana is the analyst console.
```

## Fleet

Fleet manages Elastic Agents and policies.

SOC analogy:

```text
Fleet is the management console for endpoint log collectors.
```

## Elastic Agent

Elastic Agent runs on endpoints and ships logs back to Elastic.

SOC analogy:

```text
Elastic Agent is the sensor/collector installed on the endpoint.
```

## Sysmon

Sysmon is a Microsoft Sysinternals tool that creates detailed Windows endpoint telemetry.

SOC analogy:

```text
Sysmon gives analysts deeper endpoint visibility than default Windows logs.
```

## SwiftOnSecurity Sysmon Config

The SwiftOnSecurity Sysmon configuration was used to improve event collection quality without writing a full Sysmon configuration from scratch.

Purpose:

```text
Collect useful endpoint events while reducing some unnecessary noise.
```

## Suricata

Suricata is an IDS/IPS engine used for network traffic inspection.

SOC analogy:

```text
Suricata is the network sensor that can alert on suspicious traffic patterns.
```

---

# 5. Why Elastic Was Used First

Elastic was used before Wazuh because it teaches classic SIEM concepts directly:

- Data ingestion
- Data streams
- Data views
- Log searching
- KQL queries
- Dashboards
- Agent policies
- Integrations
- Endpoint telemetry analysis
- Troubleshooting missing data

Elastic is flexible and forces the analyst to understand raw fields.

This matters because later phases depend on being comfortable with fields like:

```text
@timestamp
event.code
host.name
agent.name
winlog.channel
winlog.event_data.CommandLine
winlog.event_data.ParentImage
winlog.event_data.User
```

---

# 6. Elastic Installation on Ubuntu SIEM VM

## Purpose

Install Elasticsearch and Kibana on the Ubuntu SIEM VM so logs could be collected, indexed, searched, and visualized.

## General Steps Completed

The high-level process was:

```text
1. Update Ubuntu packages.
2. Install required packages.
3. Add the Elastic repository/signing key.
4. Install Elasticsearch.
5. Install Kibana.
6. Configure Elasticsearch.
7. Configure Kibana.
8. Start and enable services.
9. Access Kibana from the host browser.
```

Common Linux service commands used:

```bash
sudo systemctl status elasticsearch
sudo systemctl status kibana
sudo systemctl restart elasticsearch
sudo systemctl restart kibana
sudo systemctl enable elasticsearch
sudo systemctl enable kibana
```

## Why Service Status Mattered

When a SOC tool is not working, checking service status is usually the first step.

Example:

```bash
sudo systemctl status kibana
```

This answers:

```text
Is the service running?
Did it fail?
Is there an error?
Does it need a restart?
```

---

# 7. Kibana Access

Kibana was accessed from the host machine browser using the Ubuntu SIEM VM host-only IP:

```text
http://192.168.56.101:5601
```

Important notes:

- Kibana runs on port `5601`.
- The Ubuntu VM must be powered on.
- The Kibana service must be running.
- The host-only IP must be correct.
- If Kibana only listens on localhost, the host browser cannot reach it.

Useful check:

```bash
sudo ss -tulpn | grep 5601
```

Expected result:

```text
Kibana listening on 0.0.0.0:5601 or 192.168.56.101:5601
```

If it only listens on:

```text
127.0.0.1:5601
```

then it may not be reachable from the host browser.

---

# 8. Fleet Server and Elastic Agent

## Why Fleet Was Needed

Fleet provides centralized management for Elastic Agents.

Instead of manually configuring each endpoint collector, Fleet lets the analyst create an agent policy and enroll endpoints into that policy.

SOC analogy:

```text
Fleet policy = instructions for what logs the endpoint should collect.
Elastic Agent = endpoint collector that follows the policy.
```

## Basic Fleet Workflow

```text
1. Open Kibana.
2. Go to Fleet.
3. Create or use an agent policy.
4. Add integrations.
5. Generate enrollment command/token.
6. Run the command on the endpoint.
7. Confirm the agent appears as healthy.
```

## Windows Elastic Agent Enrollment

The Windows victim was enrolled into Fleet using an Elastic Agent enrollment command from Kibana.

Expected result:

```text
Agent appears in Fleet.
Agent status becomes Healthy.
Windows logs begin flowing into Elastic.
```

This established:

```text
Windows endpoint -> Elastic Agent -> Fleet policy -> Elasticsearch -> Kibana
```

---

# 9. Sysmon Installation on Windows Victim

## Why Sysmon Was Installed

Default Windows logs are useful, but Sysmon provides deeper endpoint visibility.

Sysmon records security-relevant events such as:

- Process creation
- Network connections
- File creation
- DNS queries
- Registry changes
- Process access
- Image/DLL loads

This is extremely useful for SOC work because many investigations start with process, command-line, parent process, user, and network context.

## Important Sysmon Event IDs

| Sysmon Event ID | Meaning |
|---:|---|
| 1 | Process creation |
| 3 | Network connection |
| 7 | Image loaded |
| 10 | Process access |
| 11 | File created |
| 13 | Registry value set |
| 22 | DNS query |

## Sysmon Install Pattern

Sysmon was installed on the Windows victim with a configuration file.

General install pattern:

```powershell
Sysmon64.exe -accepteula -i sysmonconfig.xml
```

The exact file names may vary depending on the Sysmon binary/config downloaded.

## Validation

After installation, Sysmon logs should appear in Windows Event Viewer under:

```text
Applications and Services Logs
  -> Microsoft
    -> Windows
      -> Sysmon
        -> Operational
```

In Kibana, Sysmon logs appear under:

```text
winlog.channel : "Microsoft-Windows-Sysmon/Operational"
```

---

# 10. Windows Integration and Winlog Collection

In Elastic, `winlog` refers to Windows Event Log data collected by Elastic Agent / Windows integration.

Fields often look like:

```text
winlog.channel
winlog.event_data.*
winlog.computer_name
```

Examples:

```text
winlog.channel : "Security"
winlog.channel : "System"
winlog.channel : "Microsoft-Windows-Sysmon/Operational"
```

The Windows integration collects logs such as:

- Security
- System
- Application
- Sysmon/Operational if configured
- PowerShell logs if enabled/configured

## Why Winlog Collection Mattered

Without Windows log collection, Kibana would not show endpoint activity.

This integration allowed later searches like:

```kql
winlog.channel : "Security" and event.code : "4624"
```

and:

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "1"
```

---

# 11. Kibana Discover Validation

Kibana Discover was used to confirm that logs were arriving.

## Important Data View

The main data view used was:

```text
logs-*
```

This allowed the lab to search across log data streams.

## Important Searches

Broad search for Windows agent:

```kql
agent.name : "DESKTOP-3JKM5O9"
```

Sysmon logs:

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational"
```

Sysmon process creation:

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "1"
```

Windows Security logs:

```kql
winlog.channel : "Security"
```

Windows successful logons:

```kql
winlog.channel : "Security" and event.code : "4624"
```

Windows failed logons:

```kql
winlog.channel : "Security" and event.code : "4625"
```

---

# 12. Generating Test Activity on Windows

To confirm logs were being collected, safe commands were executed on the Windows victim.

Examples:

```powershell
whoami
hostname
ipconfig
net user
net localgroup administrators
Start-Process notepad.exe
Start-Process calc.exe
nslookup google.com
curl.exe https://example.com
```

Expected log types:

```text
whoami / hostname / ipconfig  -> Sysmon Event ID 1 process creation
net user / localgroup         -> Sysmon Event ID 1 process creation
notepad / calc                -> Sysmon Event ID 1 process creation
nslookup                      -> Sysmon Event ID 1 and possibly DNS query
curl                          -> Sysmon Event ID 1 and network connection
```

This confirmed the endpoint was generating telemetry and the SIEM could search it.

---

# 13. Useful Kibana Columns

In Discover, adding columns made logs easier to interpret.

Useful columns:

```text
@timestamp
event.code
event.action
host.name
agent.name
winlog.channel
winlog.event_data.User
winlog.event_data.Image
winlog.event_data.CommandLine
winlog.event_data.ParentImage
winlog.event_data.ParentCommandLine
winlog.event_data.TargetFilename
winlog.event_data.QueryName
winlog.event_data.DestinationIp
winlog.event_data.DestinationPort
```

Why columns matter:

```text
Without the right columns, every log looks the same.
With the right columns, the analyst can quickly see user, command, parent process, destination, and event type.
```

This became very important in later triage phases.

---

# 14. Suricata IDS Installation

## Why Suricata Was Installed

Suricata was added to introduce network-based detection.

Elastic/Sysmon focuses heavily on endpoint telemetry. Suricata focuses on network traffic.

A SOC analyst should understand both.

## Suricata Role

Suricata can:

- Inspect network packets
- Trigger IDS alerts
- Log DNS/HTTP/TLS metadata
- Create `eve.json`
- Create `fast.log`
- Detect suspicious traffic patterns

## Important Suricata Log Files

```text
/var/log/suricata/eve.json
/var/log/suricata/fast.log
```

`eve.json` is structured JSON output.

`fast.log` is a quick alert log.

## Useful Commands

Check Suricata status:

```bash
sudo systemctl status suricata
```

Check Suricata logs:

```bash
sudo ls -lah /var/log/suricata/
```

View recent alerts:

```bash
sudo tail -n 20 /var/log/suricata/fast.log
```

View JSON events:

```bash
sudo tail -n 20 /var/log/suricata/eve.json
```

Restart Suricata:

```bash
sudo systemctl restart suricata
```

---

# 15. What Suricata Added to the Lab

Suricata gave the lab a network detection component.

This means the lab could now support:

- IDS alert review
- Network traffic investigation
- Source/destination IP analysis
- Port/protocol review
- DNS/HTTP/TLS metadata review
- Future Kali-generated traffic detection

This helped prepare for future phases involving:

- Network log fundamentals
- Threat hunting
- PCAP/network analysis
- Attack simulation
- Suricata alert triage

---

# 16. First Kibana Dashboard

Phase 2 also included early Kibana visualization/dashboard work.

The goal was to move beyond raw logs and begin seeing patterns visually.

The first SOC-focused dashboard included:

1. Sysmon event volume over time
2. Top Sysmon event codes
3. Top Sysmon processes
4. Top parent processes
5. Destination IP / network connection activity

## Why Dashboards Matter

In a SOC, dashboards help analysts quickly answer:

```text
Are logs flowing?
Which hosts are active?
Are there spikes in activity?
Which event types are most common?
Are IDS alerts appearing?
```

Dashboards are not a replacement for investigation, but they help with visibility.

---

# 17. Troubleshooting Notes

Phase 2 had multiple real troubleshooting moments. These are important because SOC work is not always clean. Tools break, logs do not show up, and dashboards are confusing until the analyst learns where to look.

## Quick Troubleshooting Reference

| Problem | What I Checked | Root Cause / Likely Cause | How I Fixed It |
|---|---|---|---|
| Elastic Agent would not enroll | Fleet command, Fleet URL, token, admin PowerShell | Token/URL/copy-paste issue | Regenerated the enrollment command from Fleet, confirmed the reachable SIEM IP, ran PowerShell as Administrator, and re-enrolled the agent cleanly |
| Windows logs were missing in Kibana | Time range, `logs-*`, Fleet health, `agent.name`, `winlog.channel` | Query too narrow or logs not reaching the right data stream | Cleared filters, widened the time picker, searched broadly by `agent.name`, then narrowed to `winlog.channel` |
| Sysmon logs were not obvious | Event Viewer path, Windows integration, raw fields | Sysmon channel needed explicit collection/field review | Verified `Microsoft-Windows-Sysmon/Operational` in Event Viewer and searched `winlog.channel : "Microsoft-Windows-Sysmon/Operational"` |
| Kibana showed no records | Data view, time range, filters, field existence | Wrong time range/data view or over-specific query | Used `logs-*`, Last 24 hours, no filters, and broad queries before adding field filters |
| ECS fields were blank | `process.name`, `process.command_line`, raw event fields | Normalized ECS fields did not always populate | Used raw Sysmon fields like `winlog.event_data.Image`, `CommandLine`, `ParentImage`, and `ParentCommandLine` |
| Fleet output used wrong address | Fleet output settings, VM adapter IPs | Output pointed at NAT-side IP instead of host-only SIEM IP | Changed Fleet Elasticsearch output to `https://192.168.56.101:9200`, then restarted Kibana and the Windows agent |
| Kibana/Fleet encryption errors appeared | Kibana logs and Fleet UI warnings | Missing Kibana encryption keys | Generated Kibana encryption keys, added them to `kibana.yml`, and restarted Kibana |
| Suricata needed validation | Service status, `eve.json`, `fast.log`, rule test | Needed proof that IDS service and logs were working | Checked `systemctl status suricata`, tailed Suricata logs, tested rules, and restarted the service after config changes |
| VM performance lagged | Running VM count, RAM usage, services | Too many heavy security VMs running at once | Ran only the VMs needed for the active phase: Ubuntu SIEM + Windows victim for Phase 2 |

## Issue 1 - Token / Enrollment Problems

Problem:

```text
The enrollment token/command must be copied exactly.
If the token is wrong, expired, missing, or pasted incorrectly, the agent will not enroll properly.
```

Troubleshooting approach:

```text
1. Regenerate or copy the enrollment command from Fleet.
2. Confirm the Fleet Server URL/IP is correct.
3. Run PowerShell as Administrator on Windows.
4. Paste the full enrollment command carefully.
5. Confirm the agent appears in Fleet.
```

Lesson:

```text
Agent enrollment problems often come from token, URL, certificate, or copy/paste issues.
```

## Issue 2 - Windows Logs Not Showing in Kibana

At one point, logs did not appear after running commands on the Windows victim.

Possible causes considered:

- Wrong time range
- Wrong data view
- Wrong field/query
- Agent not healthy
- Integration missing
- Logs delayed
- Windows event channel not being collected

Troubleshooting steps:

```text
1. Set time range to Last 24 hours or Last 30 minutes.
2. Use a broad query first.
3. Confirm the data view is logs-*.
4. Check Fleet agent health.
5. Search by agent.name.
6. Search by winlog.channel.
7. Remove overly specific filters.
```

Useful broad query:

```kql
agent.name : "DESKTOP-3JKM5O9"
```

Lesson:

```text
When logs do not show, start broad and narrow down.
```

## Issue 3 - Kibana UI Navigation Confusion

Kibana's interface can be confusing because different versions move options around.

What mattered:

```text
Find Fleet, Integrations, Data Views, and Discover.
```

Lesson:

```text
Do not memorize only button locations. Understand the function:
Fleet manages agents.
Integrations define what data is collected.
Data Views define what Kibana searches.
Discover is where analysts inspect logs.
```

## Issue 4 - Winlog Integration Confusion

At one point it seemed like `winlog` did not exist or was hard to find.

Clarification:

```text
winlog refers to Windows Event Log data in Elastic.
Windows logs appear through the Windows integration / Elastic Agent.
```

Important fields:

```text
winlog.channel
winlog.event_data.*
```

Lesson:

```text
The integration may not be called exactly what the field name suggests. The field name winlog appears after Windows Event Logs are ingested.
```

## Issue 5 - No Records Showing

Kibana sometimes showed no records.

Common causes:

```text
Wrong time range
Wrong query
Wrong data view
No logs matching filter
Agent not sending logs yet
Specific field does not exist in current dataset
```

Fix:

```text
1. Clear filters.
2. Set time range to Last 24 hours.
3. Use logs-*.
4. Search broad.
5. Add columns only after confirming documents exist.
```

Lesson:

```text
No results does not always mean the tool is broken. It often means the query is too narrow.
```

## Issue 6 - Discover vs Dashboard Confusion

There was confusion around where to create visualizations and how to view records.

Clarification:

```text
Discover is for raw records/logs.
Dashboard/Visualize/Lens is for charts.
```

SOC workflow:

```text
Use Discover first to prove the data exists.
Use visualizations later to summarize the data.
```

Lesson:

```text
Do not build dashboards before confirming raw events are searchable.
```

## Issue 7 - ECS Fields Were Sometimes Blank

Some normalized Elastic fields did not always populate as expected.

Examples:

```text
process.name
process.command_line
```

Instead, raw Windows/Sysmon fields were more reliable:

```text
winlog.event_data.Image
winlog.event_data.CommandLine
winlog.event_data.ParentImage
winlog.event_data.ParentCommandLine
winlog.event_data.User
```

Lesson:

```text
If normalized ECS fields are blank, inspect raw event fields.
```

This became extremely useful in Phases 4-6.

## Issue 8 - Fleet Output Pointing to the Wrong IP

Windows logs were not flowing correctly when Fleet output pointed to a NAT-side IP instead of the host-only IP.

Problem:

```text
Fleet output pointed to 10.0.3.15 instead of the host-only SIEM IP.
```

Fix:

```text
Updated the managed Fleet Elasticsearch output to:
https://192.168.56.101:9200
```

Then services/agents were restarted and `winlog.winlog` data streams started flowing.

Lesson:

```text
In a dual-adapter lab, always confirm which IP the endpoint can actually reach.
```

## Issue 9 - Kibana Alerting / Fleet Encryption Key Errors

Kibana/Fleet showed errors related to missing encryption keys.

Fix:

```text
Generated Kibana encryption keys.
Added them to kibana.yml.
Restarted Kibana.
```

Lesson:

```text
Some Elastic features require encryption keys before Fleet and alerting behave properly.
```

## Issue 10 - Resource Limits

Elastic, Kibana, Windows, and additional VMs can be heavy.

Resolution:

```text
Run only the VMs needed for the current phase.
```

Examples:

```text
Phase 2 Elastic work:
Run Ubuntu SIEM + Windows victim.

Wazuh work later:
Run Wazuh server + Windows victim.

Kali testing:
Run Kali only when needed.
```

Lesson:

```text
Resource planning is part of lab design.
```

---

# 18. What Phase 2 Proved

By the end of Phase 2, the lab proved:

- Elastic/Kibana could run on Ubuntu.
- Kibana was reachable from the host machine.
- Fleet could manage agents.
- Windows Elastic Agent could enroll.
- Windows endpoint logs could be collected.
- Sysmon telemetry could be collected.
- Logs could be searched in Discover.
- Useful fields could be added as columns.
- Suricata IDS could run on the Ubuntu SIEM VM.
- Basic dashboards/visualizations could be started.
- Troubleshooting could restore missing visibility.

Most importantly:

```text
The lab now had a working SIEM pipeline.
```

---

# 19. SOC Analyst Skills Practiced

## SIEM Setup

Understanding how a SIEM collects and stores logs.

## Agent Management

Understanding endpoint agent enrollment, health, and policy.

## Log Ingestion

Understanding how endpoint activity becomes searchable SIEM data.

## Windows Telemetry

Understanding the difference between Windows Security logs, System logs, Application logs, and Sysmon logs.

## Sysmon Visibility

Understanding process creation, DNS, network connections, command-line activity, and parent process relationships.

## Network Detection

Understanding Suricata as IDS telemetry.

## Troubleshooting

Diagnosing missing logs, wrong queries, token issues, integration confusion, Fleet output issues, and UI navigation problems.

---

# 20. Important Searches to Remember

## Broad Endpoint Search

```kql
agent.name : "DESKTOP-3JKM5O9"
```

## Sysmon All Logs

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational"
```

## Sysmon Process Creation

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "1"
```

## Sysmon Network Connections

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "3"
```

## Sysmon DNS Queries

```kql
winlog.channel : "Microsoft-Windows-Sysmon/Operational" and event.code : "22"
```

## Windows Security Logs

```kql
winlog.channel : "Security"
```

## Successful Logons

```kql
winlog.channel : "Security" and event.code : "4624"
```

## Failed Logons

```kql
winlog.channel : "Security" and event.code : "4625"
```

---

# 21. Community Explanation for Aspiring SOC Analysts

Phase 2 is where the lab becomes a real SOC lab.

Phase 1 gives you virtual machines.

Phase 2 gives you visibility.

Without visibility, you cannot investigate anything.

Beginner-friendly explanation:

```text
The Windows VM creates activity.
Sysmon records detailed endpoint events.
Elastic Agent collects those events.
Fleet manages the agent.
Elasticsearch stores the data.
Kibana lets the analyst search and visualize it.
Suricata adds network alerting.
```

This is the same basic idea behind many real SOC environments:

```text
Collect telemetry -> centralize logs -> search events -> investigate activity -> document findings
```

---

# 22. Interview Translation

A strong way to explain Phase 2 in an interview:

```text
In Phase 2 of my SOC lab, I built an Elastic-based SIEM pipeline. I installed Elasticsearch and Kibana on an Ubuntu SIEM VM, configured Fleet, enrolled a Windows 10 endpoint with Elastic Agent, and installed Sysmon to collect detailed endpoint telemetry. I validated log ingestion in Kibana Discover using Windows Security logs and Sysmon logs, including process creation, DNS, and network event data. I also installed Suricata IDS to begin collecting network-based telemetry. During the setup I troubleshot agent enrollment, missing logs, time range issues, Fleet output configuration, data views, and field mapping differences between ECS fields and raw winlog event data.
```

Shorter version:

```text
I built a working Elastic SIEM lab with Windows endpoint telemetry, Sysmon, Elastic Agent/Fleet, Kibana dashboards, and Suricata IDS. I used it to practice log ingestion, query troubleshooting, endpoint visibility, and SIEM investigation fundamentals.
```

---

# 23. Phase 2 Checklist

Completed:

- Ubuntu SIEM VM used as Elastic server.
- Elasticsearch installed/configured.
- Kibana installed/configured.
- Kibana reachable from browser.
- Fleet configured.
- Fleet Server installed.
- Elastic Agent enrolled on Windows victim.
- Windows logs ingested.
- Sysmon installed on Windows.
- Sysmon logs visible in Kibana.
- Important fields added as columns.
- Test activity generated on Windows.
- Kibana Discover used for validation.
- Suricata installed/configured.
- Suricata logs checked.
- First SOC dashboard created.
- Multiple ingestion/query issues troubleshot.
- Working SIEM telemetry pipeline established.

---

# 24. Final Phase 2 Summary

Phase 2 transformed the lab from a group of virtual machines into a functioning SOC monitoring environment. Elastic/Kibana became the SIEM, Fleet managed the endpoint agent, Elastic Agent shipped Windows logs, Sysmon provided deep endpoint telemetry, and Suricata introduced network IDS visibility.

This phase was important because it created the core visibility needed for every later investigation phase. Later work involving event IDs, high-volume triage, SOC tickets, escalation notes, phishing investigations, and threat hunting all depends on the telemetry pipeline built here.

Phase 2 was not just tool installation. It taught how logs move, how agents work, how to validate data, how to troubleshoot missing events, and how to think like an analyst when the SIEM does not immediately show what is expected.

**Phase 2 is complete.**

---

## Roadmap Link

Follow the full roadmap here:

[SOC and Cybersecurity Analyst Roadmap](https://mmajeed7864.github.io/soclab)
