# Day 03 Lab Log - Phase 3 Complete: Wazuh XDR Deployment

## Phase Goal

Phase 3 added **Wazuh XDR** as a second detection platform in the SOC home lab. Phase 2 focused on Elastic, Kibana, Fleet, Sysmon, and Suricata. This phase added Wazuh so the lab could compare a flexible SIEM/log analytics workflow against an XDR-style endpoint security workflow.

Final Phase 3 pipeline:

```text
Windows 10 Victim VM -> Wazuh Agent -> Wazuh Server / Indexer / Dashboard
```

The larger SOC lab now includes:

```text
Ubuntu SIEM VM        = Elastic, Kibana, Fleet, Suricata
Windows 10 Victim VM = endpoint being monitored
Wazuh Server VM      = Wazuh manager, indexer, and dashboard
Kali VM              = attacker/testing VM for later phases
```

For Phase 3, only the **Wazuh server VM** and **Windows 10 victim VM** needed to run. Elastic and Kali were kept powered off to save resources.

---

## What I Built

### Dedicated Wazuh Server VM

I created a new Ubuntu Server VM specifically for Wazuh instead of installing Wazuh on the existing Elastic SIEM VM. This kept the Elastic stack from Phase 2 stable and made the lab architecture cleaner.

VM settings:

```text
VM name: wazuh-server
RAM: 8 GB
CPU: 4 cores
Disk: ~80 GB dynamic VDI
Adapter 1: Host-only Adapter
Adapter 2: NAT
```

Network design:

```text
Host-only adapter = private lab communication between VMs
NAT adapter       = internet access for downloads and updates
```

Assigned IPs:

```text
Host-only IP: 192.168.56.105
NAT IP:       10.0.3.15
```

### Ubuntu Server and SSH

Ubuntu Server was installed with OpenSSH enabled so the server could be managed from Windows PowerShell instead of relying on the VirtualBox console.

```powershell
ssh mmajeed@192.168.56.105
```

Internet connectivity was verified from the Wazuh server, and host-only communication was validated between the Wazuh server and Windows victim VM.

### Wazuh All-in-One Install

I installed the Wazuh manager, indexer, dashboard, and Filebeat using the Wazuh all-in-one installer.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget apt-transport-https unzip gnupg
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

Installed components:

```text
Wazuh Manager
Wazuh Indexer
Wazuh Dashboard
Filebeat
```

The Wazuh dashboard became available at:

```text
https://192.168.56.105
```

The browser displayed a certificate warning because Wazuh uses a self-signed certificate by default. That is expected in a private lab environment.

Security note: the generated Wazuh admin password appeared in terminal output, so any screenshots containing that password should stay private and should not be uploaded to GitHub or LinkedIn.

### Windows Wazuh Agent Deployment

From the Wazuh dashboard, I deployed a Windows agent to the Windows 10 victim VM.

```text
Deploy new agent -> Windows
Server address: 192.168.56.105
Agent name: wazuhsoclab
Group: default
```

The generated PowerShell command installed the Wazuh agent on Windows, and the service was started with:

```powershell
NET START Wazuh
```

Wazuh confirmed the endpoint was active:

```text
Agent: wazuhsoclab
IP address: 192.168.56.104
Operating system: Microsoft Windows 10 Pro
Status: Active
Group: default
Version: 4.14.5
```

This confirmed the working pipeline:

```text
Windows 10 Victim -> Wazuh Agent -> Wazuh Server -> Wazuh Dashboard
```

---

## Tests Performed

### Basic Endpoint Activity

On the Windows victim VM, I generated basic endpoint activity:

```powershell
whoami
ipconfig
net user
net localgroup administrators
```

This confirmed Wazuh was receiving Windows endpoint data.

### Account Creation and Administrator Group Change

I performed a safe local account test:

```powershell
net user testwazuh P@ssw0rd123 /add
net localgroup administrators testwazuh /add
net localgroup administrators testwazuh /delete
net user testwazuh /delete
```

Wazuh detected:

```text
User account enabled or created
User account changed
Administrators Group Changed
Users Group Changed
Domain Users Group Changed
User account disabled or deleted
```

Key alert:

```text
Rule description: Administrators Group Changed
Rule level: 12
Rule ID: 60154
```

MITRE ATT&CK mappings:

```text
T1136 - Create Account
T1098 - Account Manipulation
T1078 - Valid Accounts
```

This was one of the most important detections in the phase because a new user being added to the Administrators group can indicate privilege escalation or persistence.

### File Integrity Monitoring

I tested File Integrity Monitoring by creating, modifying, and deleting a test file in `C:\Users\Public`.

```powershell
echo "first test" > C:\Users\Public\wazuh-fim-test.txt
Start-Sleep -Seconds 10
echo "second test" >> C:\Users\Public\wazuh-fim-test.txt
Start-Sleep -Seconds 10
del C:\Users\Public\wazuh-fim-test.txt
```

FIM did not show results at first because `C:\Users\Public` was not properly configured as a monitored directory. The configuration also had invalid XML with double angle brackets.

Corrected FIM configuration:

```xml
<syscheck>
  <disabled>no</disabled>
  <frequency>60</frequency>
  <scan_on_start>yes</scan_on_start>

  <directories check_all="yes" realtime="yes" report_changes="yes">C:\Users\Public</directories>
</syscheck>
```

After restarting the Wazuh agent, FIM successfully detected the file deletion:

```text
Path: c:\users\public\wazuh-fim-test.txt
Event: deleted
Rule description: File deleted
Rule level: 7
```

### Security Configuration Assessment

Wazuh Security Configuration Assessment ran against the Windows endpoint.

```text
Policy: CIS Microsoft Windows 10 Enterprise Benchmark v4.0.0
Passed: 117
Failed: 300
Not applicable: 7
Score: 28%
Checks: 422
```

This confirmed Wazuh was not only collecting alerts, but also assessing endpoint hardening and compliance posture.

### Vulnerability Detection

Vulnerability Detection initially showed no results, so I verified the server-side configuration:

```bash
sudo grep -nA8 -B2 "vulnerability-detection" /var/ossec/etc/ossec.conf
```

The Wazuh server had Vulnerability Detection enabled:

```xml
<vulnerability-detection>
  <enabled>yes</enabled>
  <index-status>yes</index-status>
  <feed-update-interval>60m</feed-update-interval>
</vulnerability-detection>
```

I updated the Windows agent syscollector settings so inventory, packages, ports, processes, users, groups, services, browser extensions, and hotfixes would sync faster for lab testing.

```xml
<wodle name="syscollector">
  <disabled>no</disabled>
  <interval>5m</interval>
  <scan_on_start>yes</scan_on_start>
  <hardware>yes</hardware>
  <os>yes</os>
  <network>yes</network>
  <packages>yes</packages>
  <ports all="yes">yes</ports>
  <processes>yes</processes>
  <users>yes</users>
  <groups>yes</groups>
  <services>yes</services>
  <browser_extensions>yes</browser_extensions>
  <hotfixes>yes</hotfixes>

  <synchronization>
    <max_eps>10</max_eps>
  </synchronization>
</wodle>
```

After restarting the Wazuh agent and Wazuh manager, Vulnerability Detection populated:

```text
Total findings: 301
Critical: 2
High: 229
Medium: 69
Low: 1
Pending evaluation: 0
```

This confirmed the vulnerability detection workflow was working.

---

## Troubleshooting Notes

### VM Resource Usage

Running Elastic, Wazuh, Windows, and Kali at the same time was heavier than needed for this phase.

Fix:

```text
Run only Wazuh server + Windows victim during Phase 3.
Keep Elastic and Kali powered off unless needed.
```

### Lab Connectivity

Wazuh initially could not ping some lab IPs because not every VM was powered on or needed.

Fix:

```text
Run only the VMs required for the phase.
Confirm Windows <-> Wazuh connectivity over the host-only network.
```

### VirtualBox Clipboard

Ubuntu Server did not provide a smooth copy/paste workflow through the VirtualBox console.

Fix:

```text
Install OpenSSH Server and manage the Wazuh server through SSH from Windows PowerShell.
```

### FIM Configuration

FIM did not detect the first test because the monitored directory was missing or misconfigured.

Fix:

```text
Correct the <syscheck> section.
Add C:\Users\Public as a realtime monitored directory.
Restart the Wazuh agent.
```

### Vulnerability Detection Delay

Vulnerability findings did not populate immediately.

Fix:

```text
Confirm the server vulnerability module is enabled.
Reduce the syscollector interval for lab testing.
Enable hotfix collection.
Restart the Windows Wazuh agent and Wazuh manager.
```

---

## Elastic vs Wazuh Comparison

### Elastic/Kibana Strengths

Elastic was stronger for flexible SIEM-style log analytics and custom dashboards.

Elastic was useful for:

```text
Raw log searching
Kibana Discover investigations
Sysmon event analysis
Windows event log ingestion
Process execution analysis
Parent-child process analysis
Network connection visibility
Destination IP analysis
Custom dashboards
Event volume charts
Event code breakdowns
Suricata log ingestion
Fleet and Elastic Agent management
```

Elastic felt like a flexible log analytics and SIEM platform where the analyst controls how data is searched, filtered, visualized, and investigated.

### Wazuh Strengths

Wazuh was stronger for built-in endpoint security monitoring and XDR-style visibility.

Wazuh was useful for:

```text
Endpoint agent status
Built-in alert rules
Threat Hunting dashboards
MITRE ATT&CK mapping
File Integrity Monitoring
Security Configuration Assessment
CIS benchmark checks
Vulnerability Detection
System inventory
Compliance-style visibility
Account and group change detection
Windows event alerting
```

Wazuh felt more like an endpoint security/XDR platform because it immediately provided security-focused detections, compliance checks, endpoint posture information, vulnerability visibility, and MITRE mapping.

### Main Difference

```text
Elastic = flexible SIEM/log analytics platform
Wazuh   = XDR-style endpoint security and compliance platform
```

Elastic is stronger when the goal is to search raw telemetry, build custom dashboards, and investigate process, network, and event activity deeply.

Wazuh is stronger when the goal is to monitor endpoint security posture, use built-in detections, track file integrity, review compliance checks, identify vulnerabilities, and map alerts to MITRE ATT&CK.

---

## Skills Practiced

SOC Analyst skills:

```text
Agent deployment
Endpoint monitoring
Alert triage
Threat Hunting dashboard usage
Windows event analysis
User and group change detection
Privilege escalation detection
File Integrity Monitoring
MITRE ATT&CK mapping
High severity alert review
```

Security Analyst skills:

```text
Endpoint posture review
CIS benchmark assessment
Vulnerability detection
System inventory review
Compliance-style reporting
Security configuration assessment
Risk visibility
Hardening gap identification
```

---

## Final Phase 3 Status

Phase 3 is complete.

Completed:

```text
Dedicated Wazuh VM created
Ubuntu Server installed
SSH enabled
Wazuh all-in-one installed
Wazuh dashboard accessible
Windows Wazuh agent enrolled
Windows agent active
Threat Hunting alerts confirmed
Account creation and admin group change alerts confirmed
File Integrity Monitoring confirmed
Security Configuration Assessment confirmed
Vulnerability Detection confirmed
Elastic vs Wazuh comparison completed
```

Recommended snapshots:

```text
wazuh-server snapshot:
Phase3-Wazuh-Server-Installed-Agent-Connected

windows10victim snapshot:
Phase3-Wazuh-Agent-FIM-SCA-VulnDetection-Working
```

---

## Website Summary

In Phase 3, I deployed Wazuh XDR as a second detection platform alongside my existing Elastic SIEM lab. I created a dedicated Ubuntu Server VM for Wazuh, configured host-only and NAT networking, installed the Wazuh manager, indexer, and dashboard, then enrolled my Windows 10 victim VM using the Wazuh agent. I validated endpoint monitoring by generating account creation, administrator group modification, and file integrity monitoring events. Wazuh successfully detected Windows account changes, high-severity administrator group changes, file deletion activity, CIS benchmark failures, and vulnerability findings. I also compared Wazuh against Elastic: Elastic was stronger for flexible SIEM log searching and custom dashboards, while Wazuh was stronger for endpoint posture, built-in alerts, FIM, SCA, MITRE mapping, and vulnerability detection.
