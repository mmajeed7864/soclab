# Phase 8 Detailed Lab Log — Splunk Fundamentals for SOC, SPL Investigations, and SOC Dashboards

## Phase 8 Title
**Phase 8 — Splunk Fundamentals for SOC**

## Phase 8 Status
**Completed**

Phase 8 focused on building practical Splunk familiarity after completing earlier investigations in Elastic SIEM and Wazuh. The goal was not simply to install another tool. The goal was to prove that Windows endpoint telemetry could be collected in Splunk, investigated with SPL, converted into analyst decisions, and displayed through dashboards that a SOC analyst could use during triage.

By the end of this phase, I had deployed a dedicated Splunk Enterprise server, connected a Windows endpoint through the Splunk Universal Forwarder, ingested Windows Security and Sysmon telemetry, completed 10 SPL investigation searches, built three SOC dashboards, and compared the Splunk workflow against my previous Elastic/KQL experience.

---

# 1. Purpose of Phase 8

Splunk and SPL appear frequently in SOC Analyst job descriptions. Earlier phases established a working security lab using Elastic SIEM, Suricata, Sysmon, and Wazuh, followed by investigation, triage, ticket writing, escalation, and phishing analysis.

Phase 8 added a second SIEM workflow so I could practice:

- Installing and maintaining a Splunk Enterprise server
- Receiving endpoint telemetry through a Universal Forwarder
- Searching Windows Security and Sysmon events with SPL
- Analyzing authentication, process execution, PowerShell, and DNS activity
- Turning searches into SOC dashboard panels
- Comparing Splunk/SPL against Elastic/KQL from an analyst perspective
- Recognizing when troubleshooting is necessary versus when optional tuning should be deferred

---

# 2. Lab Architecture Used

## Splunk Enterprise Server

| Item | Configuration |
|---|---|
| VM Name | `splunk-soc` |
| Role | Dedicated Splunk Enterprise server |
| Operating System | Ubuntu Server 24.04 LTS AMD64 |
| Hostname | `splunk-soc` |
| Host-only IP | `192.168.56.102` |
| NAT IP | `10.0.2.15` during setup |
| RAM | 8 GB |
| CPU | 4 cores |
| Storage | 60 GB dynamically allocated, approximately 49 GB free before install |
| Splunk Web | `http://192.168.56.102:8000` |
| Receiving Port | TCP `9997` |

## Windows Victim Endpoint

| Item | Configuration |
|---|---|
| Role | Monitored endpoint / log source |
| Hostname | `DESKTOP-3JKM5O9` |
| Host-only IP | `192.168.56.104` |
| Telemetry Components | Sysmon64, Elastic Agent, Wazuh Agent, Splunk Universal Forwarder |
| Forwarding Destination | `192.168.56.102:9997` |

## Existing Elastic SIEM Server

The existing Elastic SIEM VM remained protected and separate from Splunk. It already hosted Elasticsearch, Kibana, Fleet, Elastic Agent management, and Suricata visibility.

### Architecture decision

I deliberately created a dedicated Splunk VM instead of installing Splunk on the existing Elastic SIEM server.

The Elastic VM already showed memory pressure and was part of a working telemetry pipeline. Installing another SIEM on the same host would increase resource pressure and create unnecessary risk to completed work.

Final Phase 8 workflow:

```text
Windows Victim Endpoint
        |
        | Splunk Universal Forwarder
        v
Splunk Enterprise Server — SPL Searches and Dashboards

Existing Elastic SIEM Server — preserved for prior work and platform comparison
```

### SOC relevance

This decision mirrors a real analyst/engineering mindset: preserve working telemetry platforms, avoid unnecessary changes to established systems, and isolate new platform testing where possible.

---

# 3. Pre-Installation Validation and Snapshot Protection

Before installing Splunk Enterprise, I confirmed that the new `splunk-soc` server had the required resources and networking:

```bash
hostname
ip -4 addr show enp0s8
df -h /
free -h
```

Validated results:

| Check | Result |
|---|---|
| Hostname | `splunk-soc` |
| Host-only IP | `192.168.56.102/24` |
| Root disk | Approximately 58 GB total / 49 GB available |
| RAM | Approximately 7.8 GiB total / 7.3 GiB available |
| Swap used before install | 0 B |

Before changing the server, I created a clean baseline snapshot:

```text
Phase8-Clean-Splunk-Ubuntu-Before-Install
```

Before installing the Splunk Universal Forwarder on the Windows endpoint, I also created a rollback snapshot:

```text
Phase8-Windows-Victim-Before-Splunk-Forwarder
```

### SOC relevance

Rollback planning is not just lab convenience. Security platforms and collection agents affect visibility. Validating resources and taking snapshots before deployment reflects controlled change management.

---

# 4. Splunk Enterprise Installation

## Installer download and validation

I downloaded the official Splunk Enterprise Linux `.deb` installer directly from Splunk on the Ubuntu server:

```bash
cd /tmp && wget -O splunk-10.4.0-f798d4d49089-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.4.0/linux/splunk-10.4.0-f798d4d49089-linux-amd64.deb"
ls -lh /tmp/splunk-10.4.0-f798d4d49089-linux-amd64.deb
```

The download successfully completed from the official `download.splunk.com` source and produced the expected installer file.

## Install command

```bash
sudo dpkg -i /tmp/splunk-10.4.0-f798d4d49089-linux-amd64.deb
```

The installation completed successfully and placed Splunk Enterprise under:

```text
/opt/splunk
```

## Non-root service execution

When initially attempting to start Splunk through `sudo`, Splunk warned that running Splunk Enterprise as root is deprecated. Rather than bypassing the warning, I configured Splunk to run under its own dedicated service account:

```bash
sudo groupadd splunk
sudo useradd --system --gid splunk --home-dir /opt/splunk --shell /bin/bash splunk
sudo chown -R splunk:splunk /opt/splunk
sudo -H -u splunk /opt/splunk/bin/splunk start --accept-license
```

Successful startup validated:

```text
All preliminary checks passed.
Starting splunk server daemon (splunkd)...
Waiting for web server at http://127.0.0.1:8000 ... Done
The Splunk web interface is at http://splunk-soc:8000
```

### SOC relevance

Running a SIEM service under a dedicated non-root account follows the principle of least privilege and reduces unnecessary risk compared with running the service as root.

---

# 5. Splunk Boot Persistence and Web Access

I accessed Splunk Web from the Windows host at:

```text
http://192.168.56.102:8000
```

After confirming that Splunk Web was available, I configured Splunk to start automatically using systemd.

Because Splunk must be stopped before configuring boot-start, I performed the sequence:

```bash
sudo -H -u splunk /opt/splunk/bin/splunk stop
sudo /opt/splunk/bin/splunk enable boot-start -systemd-managed 1 -user splunk -group splunk
sudo systemctl start Splunkd
sudo systemctl status Splunkd --no-pager
```

I then rebooted the Splunk VM and validated that Splunk automatically returned without being manually started:

```bash
sudo reboot
ssh mmajeed@192.168.56.102
sudo systemctl status Splunkd --no-pager
```

Validation evidence included:

```text
splunkd started (build f798d4d49089)
```

### SOC relevance

A SIEM server that requires manual startup after every reboot is not operationally reliable. This validation proved that the platform survives routine server maintenance or an unplanned reboot.

---

# 6. Preparing Splunk to Receive Endpoint Data

From Splunk Web, I enabled the conventional receiving port for forwarder data:

```text
Settings → Forwarding and receiving → Configure receiving → New Receiving Port → 9997
```

I then confirmed on the Splunk server that Splunk was listening on TCP port `9997`:

```bash
sudo ss -ltnp | grep 9997
```

Validation result:

```text
LISTEN ... 0.0.0.0:9997 ... users:(("splunkd",...))
```

Before installing the forwarder, I tested connectivity from the Windows Victim endpoint to the Splunk receiver:

```powershell
$tcp = New-Object System.Net.Sockets.TcpClient
$task = $tcp.ConnectAsync("192.168.56.102",9997)
if ($task.Wait(3000) -and $tcp.Connected) {
    "PASS: Windows Victim can reach Splunk receiver on 192.168.56.102:9997"
} else {
    "FAIL: Port 9997 is not reachable within 3 seconds"
}
$tcp.Close()
```

Validation result:

```text
PASS: Windows Victim can reach Splunk receiver on 192.168.56.102:9997
```

### SOC relevance

This separated transport validation from agent troubleshooting. Before blaming an endpoint agent, an analyst or engineer should confirm that the receiving service is actually listening and reachable.

---

# 7. Splunk Universal Forwarder Installation on Windows

## Endpoint health validation

Before adding another telemetry agent to the Windows endpoint, I validated that the existing tools and storage were healthy.

Confirmed endpoint details:

| Item | Result |
|---|---|
| Host-only IP | `192.168.56.104` |
| NAT IP during check | `10.0.3.15` |
| C: drive total size | Approximately 49.44 GB |
| C: drive free space | Approximately 18.34 GB |
| `Elastic Agent` service | Running / Automatic |
| `Sysmon64` service | Running / Automatic |
| `WazuhSvc` service | Running / Automatic |

## Installer validation

The first Universal Forwarder download was only approximately 21 MB, which was inconsistent with the expected Windows MSI size. Instead of executing an unverified installer, I deleted it and redownloaded the official package using `curl.exe`.

```powershell
cd $env:USERPROFILE\Downloads
Remove-Item ".\splunkforwarder-10.4.0-f798d4d49089-windows-x64.msi" -Force

curl.exe -L --fail --retry 3 -o "splunkforwarder-10.4.0-f798d4d49089-windows-x64.msi" "https://download.splunk.com/products/universalforwarder/releases/10.4.0/windows/splunkforwarder-10.4.0-f798d4d49089-windows-x64.msi"

Get-Item ".\splunkforwarder-10.4.0-f798d4d49089-windows-x64.msi" | Select-Object Name, Length
Get-AuthenticodeSignature ".\splunkforwarder-10.4.0-f798d4d49089-windows-x64.msi" | Select-Object Status, StatusMessage, @{Name="Signer";Expression={$_.SignerCertificate.Subject}}
```

Validated evidence:

```text
Installer size: 154,816,512 bytes
Signature status: Valid
Signer: Splunk Inc.
```

## Installer configuration

During the Windows Universal Forwarder installation, I configured:

| Setting | Selection |
|---|---|
| Environment | On-premises Splunk Enterprise instance |
| Service account | Local System |
| Deployment Server | Left blank; not needed for one endpoint |
| Receiving Indexer | `192.168.56.102` |
| Receiving Port | `9997` |

Local System was selected because this endpoint agent needed access to local Windows event logs, including Security and Sysmon telemetry.

## Forwarder validation

After installation, I confirmed the service was active:

```powershell
Get-Service SplunkForwarder | Select-Object Name, Status, StartType
```

Result:

```text
SplunkForwarder   Running   Automatic
```

I then confirmed its active forwarding connection:

```powershell
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" list forward-server
```

Result:

```text
Active forwards:
    192.168.56.102:9997
Configured but inactive forwards:
    None
```

### SOC relevance

This workflow demonstrated installer validation, service validation, transport validation, and proof that the endpoint was actively connected to the SIEM server.

---

# 8. Troubleshooting: Active Forwarder but No Searchable Windows Events

## Problem observed

After connecting the Universal Forwarder, Splunk showed `_internal` forwarder data from:

```text
DESKTOP-3JKM5O9
```

This proved the endpoint could communicate with Splunk. However, searches for Windows Event Logs initially returned no results.

## Investigation

I used Splunk internal data and the Universal Forwarder configuration to isolate the issue:

- `_internal` events from the Windows host were visible in Splunk.
- The forwarder connection to `192.168.56.102:9997` was active.
- Effective `btool` input output did not initially show enabled Windows event log inputs.

## Root cause

The endpoint agent was connected, but the required Windows Event Log/Sysmon inputs were not effectively enabled for searchable ingestion.

## Fix

I configured the forwarder to collect the required endpoint channels through `inputs.conf`:

```powershell
$inputsPath = "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf"

@'
[WinEventLog://Application]
disabled = 0
index = main
start_from = newest
current_only = 0

[WinEventLog://Security]
disabled = 0
index = main
start_from = newest
current_only = 0

[WinEventLog://System]
disabled = 0
index = main
start_from = newest
current_only = 0

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = main
start_from = newest
current_only = 0
renderXml = true
'@ | Set-Content -Path $inputsPath -Encoding ASCII

Restart-Service SplunkForwarder
```

I then validated effective configuration using:

```powershell
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" btool inputs list --debug |
Select-String -Pattern "WinEventLog://|disabled =|index =|renderXml ="
```

Validation confirmed active Security and Sysmon input stanzas with:

```text
disabled = 0
index = main
renderXml = true
```

## Successful ingestion result

After the fix, Splunk successfully showed searchable Windows endpoint telemetry, including:

```text
WinEventLog:Security
XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

An initial validated result set showed:

```text
Security events: 4,933
Sysmon events:   2,563
Total visible:   7,111
```

Counts continued increasing as historical endpoint data was ingested.

### SOC relevance

This was a realistic troubleshooting lesson: an active agent connection does not automatically mean useful data is being collected. Analysts and SIEM engineers must distinguish:

```text
Agent connected  ≠  Required telemetry ingesting
```

---

# 9. SPL Investigation Searches Completed

Phase 8 required ten SPL searches. These searches focused on SOC-relevant endpoint evidence already available in the Splunk dataset.

## Search 1 — Sysmon DNS Query Investigation

### Purpose

Identify DNS queries to `github.com` and determine which process/user was associated with the activity.

### SPL pattern used

```spl
index=main host=DESKTOP-3JKM5O9 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| search SysmonEventID=22
| rex field=_raw "<Data Name=[\"']QueryName[\"']>(?<QueryName>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']Image[\"']>(?<Image>[^<]+)</Data>"
| rex field=_raw "<Data Name=[\"']User[\"']>(?<User>[^<]+)</Data>"
| search QueryName="*github.com*"
| table _time, host, User, Image, QueryName
| sort - _time
```

### Finding

`github.com` DNS queries were visible and attributable to the known lab user, with clean attributable rows showing `C:\Windows\System32\curl.exe`.

### Verdict

```text
Benign by context — controlled lab domain lookup.
```

---

## Search 2 — Sysmon Process Creation Investigation

### Purpose

Investigate notable process executions from Sysmon process creation telemetry.

### Evidence reviewed

- PowerShell execution
- Command-line visibility
- Parent process visibility
- User attribution

### Finding

Process creation events successfully exposed PowerShell activity, users, command lines, and parent processes for analyst review.

### Verdict

```text
Telemetry validated; process execution review operational.
```

---

## Search 3 — PowerShell Command-Line Triage

### Purpose

Separate ordinary PowerShell execution from higher-interest command patterns.

### SPL logic categories

- High-interest PowerShell pattern:
  - `-enc`
  - `-EncodedCommand`
  - `FromBase64String`
  - `Invoke-Expression`
  - `IEX`
  - `DownloadString`
- Security policy inspection:
  - `secedit /export`
- General PowerShell execution

### Finding

The repeated security-policy inspection activity was parented by the Wazuh/OSSEC agent.

### Verdict

```text
Benign by context — security agent performing expected policy assessment.
```

### SOC lesson

PowerShell plus `SYSTEM` is worth reviewing, but parent-process context prevented an incorrect escalation.

---

## Search 4 — Failed Authentication Investigation

### Purpose

Identify Windows Security Event ID `4625` failed logons.

### Finding

Failed local logon events were present for the known lab user with:

```text
EventCode:              4625
Account:                mmajeed
Logon Type:             2
Source Network Address: 127.0.0.1
Failure Reason:         Unknown user name or bad password
```

### Verdict

```text
Needs context validation; localhost failures do not prove a remote authentication attack.
```

---

## Search 5 — Failed Authentication Pattern Analysis

### Purpose

Aggregate failed authentication events by account and source to move from raw log review to triage decision-making.

### Finding

The authentication dataset later showed high-volume local failed logons tied to:

```text
Account: mmajeed
Source:  127.0.0.1
```

### Dashboard-safe triage logic

Instead of labeling localhost failures as confirmed brute force, the final dashboard wording was corrected to:

```text
High-volume local failures — validate lab context
```

### Verdict

```text
Review-worthy volume, but no remote source or confirmed brute-force evidence.
```

---

## Search 6 — Successful Logon Analysis

### Purpose

Analyze Windows Security Event ID `4624` successful logons by account and logon type.

### Finding

Successful logons were dominated by expected Windows identities:

- `SYSTEM`
- `DWM-1`
- `LOCAL SERVICE`
- `NETWORK SERVICE`
- Known lab user: `mmajeed`

### Verdict

```text
Benign by context — expected operating-system, service, session, and known-user activity.
```

---

## Search 7 — Privileged Logon Investigation

### Purpose

Analyze Windows Security Event ID `4672`, which records special privileges assigned to new logons.

### Finding

Privileged events were dominated by expected identities:

- `SYSTEM`
- `LOCAL SERVICE`
- `NETWORK SERVICE`
- `DWM-*`
- `defaultuser0`
- Known lab user `mmajeed`

### Verdict

```text
Benign by context — no unexpected privileged account identified.
```

### SOC lesson

A high number of `4672` events is not automatically malicious. The account receiving privileges and surrounding context determine whether investigation should escalate.

---

## Searches 8–10 — Additional Completed SPL Practice

The final set of completed SPL practice expanded investigation and dashboard-building around the validated endpoint telemetry sources already available in Splunk:

- Windows authentication and privilege event aggregation
- Endpoint process execution frequency analysis
- PowerShell execution review/category breakdown
- DNS domain frequency analysis
- DNS process attribution and investigation-detail tables

### Scope-control decision

A proposed outbound network connection search using Sysmon Event ID `3` was investigated briefly, then deliberately deferred after local validation showed that Event ID `3` was not being collected by the current Sysmon policy.

This was the correct Phase 8 decision because:

- Security and Sysmon telemetry were already searchable.
- Required SPL searches and dashboards could be completed with available data.
- Enabling additional Sysmon telemetry was optional tuning, not a deliverable blocker.
- Avoiding unnecessary configuration changes protected a working pipeline.

Deferred item:

```text
Optional future tuning: enable/test Sysmon Event ID 3 network connection collection.
```

---

# 10. Dashboards Built

## Dashboard 1 — Authentication & Privileged Access

### Purpose

Provide a SOC authentication triage view for successful logons, failed authentication attempts, and privileged account activity.

### Panels

1. **Successful Logons by Account**
   - Visualizes Event ID `4624` successful logons.
   - SYSTEM and expected service/session accounts dominate the activity.

2. **Failed Logons by Account and Source**
   - Aggregates Event ID `4625` failures.
   - Displays account, source address, count, first/last seen, and triage decision.
   - Corrected analyst wording:
     ```text
     High-volume local failures — validate lab context
     ```

3. **Privileged Logons by Account**
   - Aggregates Event ID `4672`.
   - Separates expected Windows/service/session identities from accounts requiring contextual review.

### Analyst takeaway

The dashboard demonstrates that authentication events require contextual interpretation rather than automatic escalation based solely on count.

---

## Dashboard 2 — Endpoint Process & PowerShell Triage

### Purpose

Support endpoint process and command-line investigation using Sysmon process telemetry.

### Panels

1. **Top Executed Processes**
   - Shows process execution frequency.
   - Visible high-volume processes included normal Windows and Splunk Universal Forwarder components.

2. **PowerShell Review Categories**
   - Categorizes PowerShell executions into:
     - General PowerShell execution
     - Security policy inspection
     - High-interest PowerShell pattern, if present

3. **PowerShell Execution Details**
   - Displays time, user, parent image, command line, and analyst verdict.
   - Demonstrated that process ancestry can explain activity that otherwise appears suspicious.

### Analyst takeaway

PowerShell must be analyzed with parent-process and command-line context. Monitoring-agent activity can resemble suspicious behavior unless validated correctly.

---

## Dashboard 3 — DNS Activity Investigation

### Purpose

Monitor Sysmon DNS query telemetry and provide process attribution for investigated domains.

### Panels

1. **Top Queried Domains**
   - Shows the most frequent DNS queries on the Windows Victim.
   - Includes expected Windows/application traffic and `github.com` lab activity.

2. **DNS Queries by Process**
   - Shows which endpoint processes generated DNS activity.
   - Supports quick identification of processes worth reviewing.

3. **GitHub DNS Investigation Details**
   - Displays event time, user, process image, queried domain, and analyst verdict.
   - Correlates `curl.exe` with `github.com`.

### Analyst takeaway

DNS activity alone is not malicious. Analysts must connect queried domains to process/user context and determine whether activity is expected.

---

# 11. Splunk vs. Elastic SOC Workflow Comparison

## Data Ingestion and Setup

### Elastic

Elastic required a broader platform setup involving Elasticsearch, Kibana, Fleet Server, Elastic Agent enrollment, and integrations for Windows and Sysmon telemetry. This felt like building a full security monitoring stack and gave strong exposure to the underlying collection architecture.

### Splunk

Splunk was deployed as a dedicated server, then connected to a Windows endpoint through the Universal Forwarder and receiving port `9997`. Once inputs were configured correctly, Windows Security and Sysmon logs were searchable through SPL.

### Comparison

Elastic involved more full-stack platform integration. Splunk felt faster for turning connected endpoint telemetry into analyst searches and dashboards, while still requiring careful input configuration.

---

## Search Language and Investigation Workflow

### Elastic / KQL

KQL felt straightforward for filtering known fields and reviewing raw events, such as:

```text
event.code
host.name
winlog.channel
winlog.event_data.Image
winlog.event_data.CommandLine
```

### Splunk / SPL

SPL felt stronger for transforming raw evidence into analyst summaries and dashboard panels. In this phase, SPL supported:

- Grouping failed authentication attempts by user/source
- Building triage decision wording
- Categorizing PowerShell activity
- Correlating activity with parent processes
- Extracting Sysmon XML fields using `rex`
- Turning results directly into dashboard panels

### Comparison

Elastic/KQL was easier for fast raw-event filtering. Splunk/SPL offered stronger aggregation, categorization, and reporting workflows for analyst-facing outputs.

---

## Dashboard Creation

In Splunk, investigation searches were converted into three dashboard views:

| Dashboard | Analyst Use |
|---|---|
| Authentication & Privileged Access | Review logons, failures, and privileged account activity |
| Endpoint Process & PowerShell Triage | Analyze process execution and PowerShell context |
| DNS Activity Investigation | Review queried domains and process attribution |

Splunk made it natural to build panels directly from completed searches and turn investigation logic into repeatable dashboards.

---

## Overall Analyst Experience

Elastic helped establish the foundation: log collection, endpoint visibility, KQL filtering, and high-volume triage practice.

Splunk reinforced the same investigation mindset while making it easier to express analyst conclusions in searches and dashboards.

The most important lesson across both platforms was:

```text
Suspicious technique does not equal confirmed malicious activity.
Context determines the verdict.
```

Examples from Phase 8:

| Activity | Initial Concern | Context | Verdict |
|---|---|---|---|
| PowerShell as SYSTEM | Could indicate attacker execution | Parent process tied to security/monitoring agent | Benign by context |
| Failed logons | Could indicate brute force | Localhost source on controlled lab endpoint | Validate context; not confirmed brute force |
| Privileged logons | Could indicate admin misuse | Dominated by expected Windows identities | Benign by context |
| `curl.exe` DNS activity to GitHub | Could indicate download/staging behavior | Controlled lab lookup by known user | Benign by context |

---

# 12. Issues Encountered and Troubleshooting Summary

| Issue | Diagnosis | Resolution | Lesson |
|---|---|---|---|
| Existing Elastic VM had limited available memory | Installing Splunk there could risk working SIEM services | Created dedicated `splunk-soc` VM | Protect stable systems and isolate new platform deployments |
| Splunk warned against running as root | Root execution is poor service practice | Created dedicated `splunk` service user | Apply least privilege to security infrastructure |
| Boot-start configuration failed while Splunk was running | Service must be stopped before enabling systemd management | Stopped service, enabled boot-start, reboot-tested | Validate service persistence operationally |
| First Universal Forwarder download was too small | Installer likely incomplete/incorrect | Redownloaded and validated digital signature | Verify security-tool installers before execution |
| Forwarder connected but Windows logs were initially missing | Connection worked but required log inputs were not effectively enabled | Added Windows Security/Sysmon stanzas to `inputs.conf` and restarted service | Connectivity does not equal useful telemetry |
| Sysmon Event ID 3 unavailable | Current Sysmon policy did not collect network connection telemetry | Deferred optional tuning | Do not waste time on non-blocking telemetry expansions |
| Failed-logon dashboard initially labeled localhost volume as brute force | Triage logic lacked source-aware context | Corrected panel wording | Public SOC evidence must avoid overclaiming |

---

# 13. SOC Skills Practiced

Phase 8 demonstrated hands-on experience with:

- Splunk Enterprise installation and initial platform setup
- Dedicated SIEM VM architecture decisions
- Linux service account security practices
- systemd boot persistence validation
- Splunk receiving-port configuration
- Splunk Universal Forwarder installation and validation
- Verifying signed endpoint-agent installers
- Windows Security and Sysmon ingestion troubleshooting
- SPL searches and raw XML field extraction with `rex`
- Windows authentication event analysis:
  - `4624` successful logons
  - `4625` failed logons
  - `4672` privileged logons
- Sysmon DNS and process analysis:
  - Event ID `1` process creation
  - Event ID `22` DNS query
- PowerShell triage using command-line and parent-process context
- Dashboard creation for authentication, endpoint process, and DNS workflows
- Platform comparison: Splunk/SPL versus Elastic/KQL
- Scope control: deferring optional tuning that did not block required deliverables

---

# 14. Interview Translation

## Resume / Interview Summary

In Phase 8 of my SOC home lab, I deployed Splunk Enterprise on a dedicated Ubuntu Server VM, configured non-root service execution and reboot persistence, and enabled a receiving pipeline for a Windows endpoint through the Splunk Universal Forwarder. I troubleshot an active forwarder connection that initially lacked searchable Windows inputs, configured Windows Security and Sysmon ingestion, wrote SPL investigations for authentication, process execution, PowerShell, and DNS activity, and built three SOC dashboards. I also compared Splunk/SPL workflows with my prior Elastic/KQL investigations and documented why context matters before escalating suspicious-looking activity.

## Interview Talking Points

### Why did you deploy Splunk on a separate VM instead of your existing Elastic SIEM VM?

The Elastic server was already working and showed resource pressure. Installing Splunk on that same VM would risk disrupting an established telemetry pipeline. I created a dedicated Splunk VM to preserve the working system and keep the platform comparison clean.

### What did you troubleshoot?

The Universal Forwarder connected successfully and generated internal logs, but Windows Security and Sysmon data were initially missing from searches. I used the active forwarder connection and effective `btool` configuration to identify that the required inputs were not enabled, then configured `inputs.conf`, restarted the forwarder, and validated searchable endpoint telemetry.

### What was your most important analyst finding?

PowerShell activity running under `SYSTEM` initially looked review-worthy, but the parent process showed it was tied to a known monitoring/security agent performing security policy inspection. That reinforced that analysts should use context before escalating.

### Did you tune every missing telemetry source?

No. I confirmed Sysmon Event ID 3 network connection telemetry was not currently collected, but it did not block the required SPL investigations or dashboards. I documented it as optional future tuning and continued with the Phase 8 deliverables. This avoided wasting time or risking a working telemetry pipeline.

---

# 15. What Was Deferred

The following item was intentionally deferred:

## Sysmon Event ID 3 — Network Connection Telemetry

I tested for network connection events after generating controlled `curl.exe` traffic. Local validation showed no new Sysmon Event ID `3` events were being recorded under the current Sysmon policy.

Decision:

```text
Deferred — optional future telemetry tuning, not a blocker for Phase 8.
```

Why this was the correct choice:

- Security and Sysmon process/DNS telemetry were already successfully ingested.
- The ten SPL searches and three dashboards could be completed with existing evidence.
- Editing the active Sysmon policy was outside the required deliverables.
- Preserving momentum and protecting the working lab was higher ROI.

---

# 16. Phase 8 Completion Checklist

| Requirement | Status |
|---|---|
| Dedicated Splunk Enterprise VM created | Complete |
| Pre-installation snapshot taken | Complete |
| Splunk Enterprise installed | Complete |
| Splunk configured to run as non-root account | Complete |
| Splunk Web accessible | Complete |
| Boot-start configured and reboot validated | Complete |
| TCP 9997 receiver configured | Complete |
| Windows Victim snapshot taken before forwarder install | Complete |
| Universal Forwarder installer verified and signed | Complete |
| Universal Forwarder installed | Complete |
| Active forwarding connection verified | Complete |
| Windows Security telemetry searchable | Complete |
| Sysmon telemetry searchable | Complete |
| Initial missing-input issue troubleshot and fixed | Complete |
| 10 SPL searches completed | Complete |
| 3 Splunk dashboards built | Complete |
| Splunk vs. Elastic SOC workflow comparison documented | Complete |
| Optional Event ID 3 tuning documented as deferred | Complete |

---

# 17. Final Phase 8 Outcome

Phase 8 added Splunk to the lab as a second working SIEM investigation environment. I did more than install the product: I built the endpoint-to-SIEM telemetry pipeline, verified its resilience after reboot, troubleshot missing Windows inputs, extracted and investigated real endpoint events with SPL, created three analyst-facing dashboards, and compared the workflow to Elastic/KQL.

The most important professional lesson was not a command or dashboard. It was the analyst decision process:

```text
Collect telemetry → Confirm visibility → Search evidence → Add context → Assign verdict → Document accurately
```

Phase 8 strengthened practical readiness for SOC Analyst work by demonstrating the ability to work across multiple SIEM platforms and turn raw security logs into defensible analyst conclusions.
