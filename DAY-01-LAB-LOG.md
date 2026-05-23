# Day 01 Lab Log - Phase 1 Complete: Virtual Lab Foundation for SOC Analyst Home Lab

**Date:** May 18, 2026  
**Phase Completed:** Phase 1 - Virtual Lab Environment Setup  
**Focus:** VirtualBox lab architecture, NAT/host-only networking, Windows/Ubuntu/Kali VM roles, SSH workflow, snapshots, and troubleshooting

---

## Phase 1 Title

**Phase 1 - Virtual Lab Environment Setup**

## Phase 1 Status

**Completed**

This phase built the foundation for the entire SOC Analyst home lab. The goal was not just to install VirtualBox and create virtual machines. The goal was to build a safe, repeatable, isolated lab environment that could support future phases involving SIEM deployment, endpoint monitoring, IDS traffic analysis, Wazuh XDR, phishing investigation, SOC ticket writing, and attack simulation.

This writeup is intentionally detailed so other aspiring SOC analysts can follow the same process and understand why each step matters.

---

# 1. Purpose of Phase 1

Phase 1 created the base virtual environment needed for the rest of the roadmap.

A SOC analyst home lab needs multiple systems that can communicate with each other in a controlled environment. In a real company, a SOC analyst reviews logs from endpoints, servers, firewalls, EDR tools, SIEM platforms, identity systems, and network security tools.

In this lab, those pieces are simulated with virtual machines.

The purpose of this phase was to create:

- A safe virtual network
- A Windows victim endpoint
- An Ubuntu SIEM server
- A Kali Linux attacker/testing machine
- A repeatable IP addressing plan
- A baseline snapshot strategy
- A clean foundation before installing security tools

The main idea:

```text
Build the lab first.
Confirm networking.
Take snapshots.
Then install security tools in later phases.
```

---

# 2. High-Level Lab Design

The initial Phase 1 lab design included three main virtual machines:

```text
Ubuntu SIEM VM       = Security monitoring / SIEM server
Windows 10 Victim VM = Endpoint being monitored and tested
Kali Linux VM        = Attacker/testing machine for later phases
```

Later phases added Wazuh as a separate VM, but Phase 1 focused on the original core environment.

## Phase 1 VM Roles

| VM | Role | Why It Exists |
|---|---|---|
| Ubuntu SIEM VM | Monitoring server | Hosts Elastic/Kibana/Fleet/Suricata in later phases |
| Windows 10 Victim VM | Endpoint | Generates logs, Sysmon telemetry, Windows events, user activity |
| Kali Linux VM | Testing/attacker box | Used later for scanning, traffic generation, and safe attack simulation |

---

# 3. Why VirtualBox Was Used

Oracle VirtualBox was used as the hypervisor because it is free, beginner-friendly, and works well for home labs.

A hypervisor lets multiple virtual machines run on one physical laptop/desktop. Each VM behaves like a separate computer with its own operating system, disk, CPU, RAM, and network adapter.

For a SOC lab, virtualization is useful because:

- It allows multiple machines to run on one computer.
- It lets the analyst safely break and fix systems.
- It allows snapshots before risky changes.
- It creates an isolated network separate from the real home network.
- It makes it possible to simulate endpoints, servers, and attacker machines.

---

# 4. Important Safety Principle

The lab must be isolated enough that experiments do not affect the real home network or work devices.

This is why the lab used VirtualBox networking instead of exposing everything directly to the home LAN.

The key idea:

```text
NAT = internet access
Host-only = private lab communication
```

This allows the VMs to download packages while still keeping lab communication mostly contained inside a private virtual network.

---

# 5. VirtualBox Network Design

Phase 1 used two main network adapter types:

## NAT Adapter

Purpose:

```text
Allows the VM to access the internet through the host machine.
```

Used for:

- Downloading Ubuntu packages
- Installing Elastic/Wazuh packages later
- Updating tools
- Accessing public repositories

NAT is useful because the VM can reach the internet, but the internet cannot easily reach back into the VM.

## Host-Only Adapter

Purpose:

```text
Allows VMs to communicate with each other on a private VirtualBox network.
```

Used for:

- Windows victim talking to Ubuntu SIEM
- Ubuntu SIEM receiving logs
- Kali testing against the lab machines
- Wazuh communicating with agents in later phases

Typical host-only network range:

```text
192.168.56.0/24
```

Example lab IPs:

```text
Ubuntu SIEM VM:       192.168.56.101
Windows 10 Victim VM: 192.168.56.104
Wazuh Server later:   192.168.56.105
```

---

# 6. Why Two Adapters Were Used

Each major VM needed internet access and lab communication.

The design:

```text
Adapter 1: NAT
Adapter 2: Host-only Adapter
```

or depending on the VM order:

```text
Adapter 1: Host-only Adapter
Adapter 2: NAT
```

The important part is not the adapter number. The important part is that each VM has:

```text
One adapter for internet
One adapter for private lab traffic
```

This mirrors real-world networking concepts where machines may have different routes/interfaces for different purposes.

---

# 7. Phase 1 Network Diagram

Basic Phase 1 layout:

```text
                    Internet
                       |
                 NAT Adapter
                       |
          -------------------------
          |                       |
     Ubuntu SIEM VM          Windows Victim VM
     192.168.56.101         192.168.56.104
          |                       |
          -------- Host-only -------
                    Network
                       |
                  Kali Linux VM
              192.168.56.x later
```

Later phases added:

```text
Wazuh Server VM
192.168.56.105
```

But Phase 1 created the foundation.

---

# 8. VM 1 - Ubuntu SIEM VM

## Purpose

The Ubuntu SIEM VM was created to become the central security monitoring server.

Later phases used this VM to install:

- Elasticsearch
- Kibana
- Elastic Agent
- Fleet Server
- Suricata IDS
- Dashboards
- Log ingestion pipelines

## Recommended Ubuntu VM Settings

Suggested settings:

```text
Name: Ubuntu-SIEM or SIEM-UbuntuVM
Type: Linux
Version: Ubuntu 64-bit
RAM: 8 GB preferred if available
CPU: 2-4 cores
Disk: 80 GB dynamically allocated
Network Adapter 1: NAT
Network Adapter 2: Host-only Adapter
```

## Why These Specs Matter

Elastic and Kibana are resource-heavy. The SIEM VM needs more memory and disk than a basic Linux VM.

If resources are limited, the SIEM VM should be prioritized because it runs the main logging stack.

---

# 9. VM 2 - Windows 10 Victim VM

## Purpose

The Windows 10 victim VM acts as the monitored endpoint.

This VM is where user and endpoint activity happens. Later phases installed:

- Sysmon
- Elastic Agent
- Wazuh Agent
- Windows logging integrations

This VM generated logs such as:

- Process creation
- DNS queries
- Network connections
- Login events
- Failed logons
- Account changes
- File activity
- PowerShell activity

## Recommended Windows VM Settings

```text
Name: windows10victim
Type: Microsoft Windows
Version: Windows 10 64-bit
RAM: 4 GB minimum
CPU: 2 cores minimum
Disk: 60-80 GB dynamically allocated
Network Adapter 1: NAT
Network Adapter 2: Host-only Adapter
```

## Why Windows Was Needed

Most entry-level SOC jobs involve Windows logs. Windows endpoints generate many of the events analysts review daily:

- 4624 successful logon
- 4625 failed logon
- 4672 special privileges
- 4720 user created
- 4732 user added to local group
- Sysmon Event ID 1 process creation
- Sysmon Event ID 3 network connection
- Sysmon Event ID 22 DNS query

Phase 1 created the endpoint that would later generate all of that telemetry.

---

# 10. VM 3 - Kali Linux VM

## Purpose

Kali Linux was included as the attacker/testing machine for later phases.

Important note:

```text
Kali is not used to attack real systems.
Kali is only used inside the lab against lab-owned virtual machines.
```

Later roadmap phases can use Kali for:

- Nmap scans
- Safe traffic generation
- Testing detection rules
- Simulated reconnaissance
- IDS alert generation
- Controlled attack exercises

## Recommended Kali VM Settings

```text
Name: kali-lab
Type: Linux
Version: Debian 64-bit
RAM: 2-4 GB
CPU: 2 cores
Disk: 40-60 GB dynamically allocated
Network Adapter 1: NAT
Network Adapter 2: Host-only Adapter
```

## Why Kali Was Included

SOC analysts do not need to become penetration testers immediately, but understanding attacker behavior helps them recognize suspicious activity.

Kali gives a safe way to generate controlled scanning and attack-like activity later in the roadmap.

---

# 11. Installation Process Overview

The general VM installation process for each machine followed this pattern:

```text
1. Download ISO
2. Create VM in VirtualBox
3. Assign RAM/CPU/disk
4. Attach ISO
5. Configure NAT + Host-only adapters
6. Install OS
7. Log in
8. Confirm IP addresses
9. Test internet access
10. Test VM-to-VM connectivity
11. Take snapshot
```

This process matters because every future phase depends on clean networking and repeatable VM states.

---

# 12. Ubuntu Installation Steps

## Step 1 - Create VM

In VirtualBox:

```text
New -> Name VM -> Select Linux / Ubuntu 64-bit
```

## Step 2 - Assign Resources

Recommended:

```text
RAM: 8192 MB if available
CPU: 2-4 cores
Disk: 80 GB dynamically allocated
```

## Step 3 - Attach Ubuntu ISO

Attach the Ubuntu ISO as the optical disk.

## Step 4 - Configure Network

Use:

```text
Adapter 1: NAT
Adapter 2: Host-only Adapter
```

## Step 5 - Install Ubuntu

During install:

```text
Hostname: ubuntu-siem or siem-ubuntu
Username: lab user
Password: saved locally
```

Do not store passwords in GitHub or screenshots.

## Step 6 - Install OpenSSH Server

If prompted, install OpenSSH server or install it later:

```bash
sudo apt update
sudo apt install -y openssh-server
```

Why:

```text
SSH allows easier copy/paste and administration from the host laptop.
```

## Step 7 - Confirm IP Addresses

Run:

```bash
ip addr
```

Look for:

```text
NAT IP: usually 10.x.x.x
Host-only IP: usually 192.168.56.x
```

The SIEM VM was later referenced as:

```text
192.168.56.101
```

---

# 13. Windows 10 Installation Steps

## Step 1 - Create VM

In VirtualBox:

```text
New -> Microsoft Windows -> Windows 10 64-bit
```

## Step 2 - Assign Resources

Recommended:

```text
RAM: 4096 MB or more
CPU: 2 cores
Disk: 60-80 GB dynamically allocated
```

## Step 3 - Attach Windows ISO

Attach the Windows 10 ISO.

## Step 4 - Configure Network

Use:

```text
Adapter 1: NAT
Adapter 2: Host-only Adapter
```

## Step 5 - Install Windows

Install Windows normally.

## Step 6 - Confirm IP

Open PowerShell:

```powershell
ipconfig
```

Look for:

```text
Host-only IPv4 Address: 192.168.56.x
```

The Windows victim later used:

```text
192.168.56.104
```

---

# 14. Kali Installation Steps

## Step 1 - Create VM

In VirtualBox:

```text
New -> Linux -> Debian 64-bit
```

## Step 2 - Assign Resources

Recommended:

```text
RAM: 2048-4096 MB
CPU: 2 cores
Disk: 40-60 GB dynamically allocated
```

## Step 3 - Attach Kali ISO

Attach the Kali ISO.

## Step 4 - Configure Network

Use:

```text
Adapter 1: NAT
Adapter 2: Host-only Adapter
```

## Step 5 - Install Kali

Install normally.

## Step 6 - Confirm IP

Run:

```bash
ip addr
```

Confirm the host-only address.

---

# 15. Basic Connectivity Testing

Once the VMs were installed, networking needed to be verified.

## Test Internet from Ubuntu

```bash
ping -c 4 google.com
```

Expected result:

```text
Replies received
```

If this works, NAT internet access is working.

## Test Internet from Windows

```powershell
ping google.com
```

or:

```powershell
nslookup google.com
```

Expected result:

```text
DNS resolves and network responds
```

## Test Lab Connectivity

From Ubuntu SIEM to Windows:

```bash
ping -c 4 192.168.56.104
```

From Windows to Ubuntu SIEM:

```powershell
ping 192.168.56.101
```

Expected result:

```text
VMs can reach each other over host-only network
```

---

# 16. Windows Firewall Note

Sometimes Windows does not respond to ping because Windows Defender Firewall blocks ICMP echo requests.

Important lesson:

```text
A failed ping does not always mean the host is unreachable.
It may mean the firewall is blocking ICMP.
```

If needed, Windows firewall ICMP rules can be enabled later. However, for many later phases, agent communication can still work even if ping is blocked.

SOC lesson:

```text
Network troubleshooting requires understanding both connectivity and firewall behavior.
```

---

# 17. SSH Access to Ubuntu SIEM

Once Ubuntu had OpenSSH installed, the host laptop could connect by SSH.

From Windows PowerShell:

```powershell
ssh <username>@192.168.56.101
```

Example:

```powershell
ssh mmajeed@192.168.56.101
```

Why SSH matters:

- Easier copy/paste
- Easier command execution
- More realistic Linux administration
- Better than typing long commands inside VirtualBox console

---

# 18. Why Snapshots Were Important

Snapshots were taken after clean setup milestones.

A snapshot preserves the VM state so the lab can be restored if something breaks.

Recommended Phase 1 snapshots:

```text
Ubuntu SIEM:
Phase1-Clean-Ubuntu-Networking-Working

Windows 10 Victim:
Phase1-Clean-Windows-Networking-Working

Kali:
Phase1-Clean-Kali-Networking-Working
```

Why snapshots matter:

- Security tools can break configs.
- Package installs can fail.
- Network settings can get misconfigured.
- It is faster to revert than rebuild from scratch.
- Professional lab work requires rollback points.

---

# 19. Phase 1 Issues and Troubleshooting

## Issue 1 - Understanding NAT vs Host-Only

Early lab setup required understanding why a VM may have more than one network adapter.

Resolution:

```text
NAT was used for internet.
Host-only was used for private lab communication.
```

Lesson:

```text
Do not rely on one adapter for everything.
Separate internet access from internal lab traffic.
```

---

## Issue 2 - IP Address Confusion

Different VMs had different interfaces and IP ranges.

Example:

```text
10.x.x.x = NAT
192.168.56.x = Host-only lab network
```

Resolution:

Use:

```bash
ip addr
```

on Linux and:

```powershell
ipconfig
```

on Windows.

Lesson:

```text
Use the 192.168.56.x address for lab VM-to-VM communication.
Use NAT for internet access.
```

---

## Issue 3 - Running Too Many VMs at Once

The laptop has limited resources, so running every VM at the same time can cause lag or service problems.

Resolution:

Only run the VMs needed for the current phase.

Examples:

```text
Elastic work:
Run Ubuntu SIEM + Windows victim

Wazuh work:
Run Wazuh server + Windows victim

Kali testing:
Run Kali + target VM + monitoring VM if needed
```

Lesson:

```text
Resource planning is part of lab design.
```

---

## Issue 4 - Need for SSH / Copy-Paste

Typing long commands directly into the VirtualBox console is inefficient and error-prone.

Resolution:

Install and use SSH for Linux VMs.

```bash
sudo apt install -y openssh-server
```

Then connect from host:

```powershell
ssh <username>@<host-only-ip>
```

Lesson:

```text
SSH improves workflow and mirrors real Linux administration.
```

---

## Issue 5 - Windows Firewall / Ping Behavior

Windows may not always respond to ping even when it is online.

Resolution:

Understand that ICMP may be blocked by firewall policy.

Lesson:

```text
If ping fails, check firewall and service connectivity before assuming the host is down.
```

---

# 20. What Phase 1 Proved

By the end of Phase 1, the lab had a working virtual foundation.

Confirmed:

- VirtualBox installed and usable
- Ubuntu SIEM VM created
- Windows 10 victim VM created
- Kali Linux VM created or planned as attacker/testing machine
- NAT internet access configured
- Host-only lab network configured
- VMs could be assigned private lab IPs
- SSH access could be used for Linux administration
- Snapshots could protect progress
- The lab was ready for Elastic, Sysmon, Suricata, and Wazuh in future phases

---

# 21. Why Phase 1 Matters for SOC Analyst Skills

Phase 1 may look like basic setup, but it maps directly to real IT/security work.

A SOC analyst does not only look at alerts. They also need to understand:

- Hosts
- IP addressing
- Network paths
- Internal vs external traffic
- Firewalls
- Services
- Logs
- Troubleshooting
- System roles
- Endpoint vs server responsibilities

Phase 1 introduced those concepts through hands-on setup.

---

# 22. Interview Translation

A strong way to explain Phase 1 in an interview:

```text
I built a virtual SOC lab using VirtualBox with separate Ubuntu, Windows, and Kali virtual machines. I configured NAT networking for internet access and host-only networking for private lab communication. The Windows VM acts as the monitored endpoint, the Ubuntu VM acts as the SIEM server, and Kali is reserved for controlled testing and traffic generation. I validated connectivity using ipconfig, ip addr, ping, and SSH, then took clean snapshots before installing security tools. This gave me a safe environment to build Elastic, Wazuh, Sysmon, Suricata, and future SOC investigations without affecting my real network.
```

---

# 23. Community Explanation for Aspiring SOC Analysts

If someone new to SOC labs asks why Phase 1 matters, explain it like this:

```text
Before you can investigate alerts, you need machines that create alerts and a place to collect them. Phase 1 builds that environment. The Windows VM becomes the endpoint. The Ubuntu VM becomes the SIEM. Kali becomes the testing machine. NAT gives the VMs internet access, and host-only networking lets the lab machines talk privately. Once that works, you can safely install logging agents, generate events, and practice real SOC workflows.
```

---

# 24. Phase 1 Checklist

Completed or established:

- VirtualBox used as the hypervisor
- Ubuntu SIEM VM created
- Windows 10 victim VM created
- Kali Linux testing VM created/planned
- NAT networking configured
- Host-only networking configured
- VM IP addresses identified
- Internet connectivity tested
- VM-to-VM connectivity tested
- SSH access established for Ubuntu
- Snapshot strategy established
- Lab roles defined
- Lab ready for Phase 2 Elastic/Sysmon/Suricata work

---

# 25. Final Phase 1 Summary

Phase 1 created the foundation for the entire SOC Analyst home lab. The lab was designed around a realistic security operations structure: a monitored Windows endpoint, an Ubuntu-based SIEM server, and a Kali testing machine. VirtualBox networking was configured with NAT for internet access and host-only networking for private lab communication.

The phase also introduced important troubleshooting concepts such as IP identification, adapter roles, ping behavior, firewall considerations, SSH access, and snapshot management. This foundation made the later phases possible, including Elastic SIEM deployment, Sysmon telemetry, Suricata IDS, Wazuh XDR, ticket writing, and SOC investigation practice.

Phase 1 is complete.
