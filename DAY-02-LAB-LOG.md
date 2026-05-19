# Day 2 — Lab Build Log

**Date:** May 19, 2026
**Phase Completed:** Phase 2 — Elastic SIEM + Suricata IDS (Windows telemetry, Fleet, and dashboards)
**SIEM Host IP:** `192.168.56.101`

---

## What I Built

In Day 2 I completed the detection and log-ingestion layer of the SOC lab. Built and configured a Windows 10 victim VM, deployed Sysmon with the SwiftOnSecurity config, stood up Fleet Server, enrolled the Windows agent into Elastic, and built the first SOC-focused Kibana dashboard with five panels.

---

## Windows 10 Victim VM + Sysmon

- Built and configured a Windows 10 victim VM in VirtualBox
- Fixed networking by attaching both **Host-Only** and **NAT** adapters
- Confirmed Windows-to-Ubuntu communication via ICMP
- Installed **Sysmon** with the SwiftOnSecurity configuration
- Verified Sysmon creating Windows Event Logs under `Microsoft-Windows-Sysmon/Operational`

---

## Elastic Fleet + Agent Enrollment

- Installed **Fleet Server** on the Ubuntu SIEM VM
- Enrolled the Windows 10 victim with **Elastic Agent**
- Added the **System integration** for baseline Windows telemetry
- Added a **Custom Windows Event Logs integration** targeting `Microsoft-Windows-Sysmon/Operational`
- Confirmed Windows/Sysmon logs ingesting into Elastic under the `winlog.winlog` dataset

---

## First SOC-Focused Kibana Dashboard

Built a 5-panel dashboard:

1. Sysmon event volume over time
2. Top Sysmon event codes
3. Top Sysmon processes
4. Top parent processes
5. Destination IP / network connection activity

---

## ⚠ Issues Hit + How I Fixed Them

This is the real value of building in public — documenting the troubleshooting, not just the wins.

| Issue | Root Cause | Fix |
|---|---|---|
| Kibana enrollment + auth failures | Stale enrollment tokens | Regenerated tokens via Elasticsearch API, re-ran setup carefully |
| VirtualBox clipboard broken | Guest Additions missing | Installed Guest Additions on Windows, switched to SSH into Ubuntu |
| Windows VM had no internet | Only had Host-Only adapter | Added NAT adapter, kept Host-Only for lab traffic |
| Windows and Ubuntu could not ping | Network profile set to Public, ICMP blocked | Changed Host-Only interface to Private, added inbound ICMP firewall rule |
| Fleet Server install failing | Service token auth not working | Generated fresh Fleet Server token via Elasticsearch API, reinstalled cleanly |
| Kibana alerting/Fleet errors | Missing encryption keys | Generated Kibana encryption keys, added to `kibana.yml`, restarted |
| Windows logs not in Discover | Fleet output pointing to NAT IP `10.0.3.15` | Updated managed Fleet Elasticsearch output to `https://192.168.56.101:9200`, restarted Kibana + Windows agent, `winlog.winlog` data streams started flowing |

---

## Skills Sharpened

- SIEM deployment + Fleet/Agent enrollment end-to-end
- Sysmon configuration and Windows telemetry collection
- Kibana dashboard authoring with SOC-relevant panels
- VirtualBox dual-adapter networking
- Windows firewall and network profile troubleshooting
- Token/auth troubleshooting, service management, log pipeline validation

---

## Current Lab State

- ✅ Elasticsearch + Kibana running
- ✅ Suricata IDS validated
- ✅ Windows 10 victim VM built with Sysmon
- ✅ Fleet Server running
- ✅ Elastic Agent enrolled and shipping Sysmon logs
- ✅ First Kibana SOC dashboard live

---

## Next Up — Phase 3

- Build a full Active Directory domain on Windows Server
- Simulate AD attacks from Kali — password spray, Kerberoasting, AS-REP Roasting, BloodHound recon, Pass-the-Hash, DCSync
- Detect each attack in Kibana and map findings to MITRE ATT&CK

---

> Documenting every step publicly. Follow along: [SOC Analyst Roadmap](https://mmajeed7864.github.io/soclab)
