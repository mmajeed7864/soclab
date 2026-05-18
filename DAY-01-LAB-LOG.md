# Day 1 — Lab Build Log

**Date:** May 18, 2026
**Phases Completed:** Phase 1 (Lab Foundation) + Phase 2 (Elastic SIEM + Suricata IDS)
**SIEM Host IP:** `192.168.56.101`

---

## Phase 1 — Lab Foundation ✅

Built the core SOC lab infrastructure from scratch in Oracle VirtualBox.

- Created Ubuntu Server SIEM VM with dual networking
- Configured **NAT adapter** for internet access (package downloads)
- Configured **Host-Only adapter** for isolated lab communication
- Confirmed Ubuntu VM can communicate with Windows host
- Identified SIEM VM host-only IP: `192.168.56.101`
- Troubleshot DNS resolution, package repositories, service restarts, and network adapter behavior

---

## Phase 2 — Elastic SIEM + Suricata IDS ✅

### Elasticsearch + Kibana

- Installed **Elasticsearch** and **Kibana** on the Ubuntu SIEM VM
- Verified Elasticsearch running with `systemctl` and `curl`
- Configured Kibana to bind to `0.0.0.0` so it's accessible from Windows host browser
- Completed Kibana enrollment + setup at `http://192.168.56.101:5601`
- Successfully logged into the Elastic web interface

### Suricata IDS

- Installed **Suricata IDS** and `jq` for JSON parsing
- Updated rule sets with `suricata-update`
- Identified key interfaces:
  - `enp0s8` → NAT / internet traffic
  - `enp0s3` → Host-Only / lab traffic
- Created custom ICMP test rule in `/tmp/local.rules`
- Fixed rule syntax issues
- Validated rule with `suricata -T`
- Used `tcpdump` to confirm SIEM VM could see ICMP traffic from Windows host
- Ran Suricata in `pcap` mode on `enp0s3`
- Generated traffic by pinging the SIEM VM from Windows
- **Confirmed alerts in:**
  - `/var/log/suricata/fast.log`
  - `/var/log/suricata/eve.json`
- Custom alert signature: `SOC LAB ICMP TEST ALERT`

---

## Skills Practiced

| Category | What I Did |
|---|---|
| Linux Admin | Service management, package install, troubleshooting |
| SIEM Setup | Elastic + Kibana install, enrollment, authentication reset |
| IDS | Suricata install, rule writing, rule validation |
| Networking | VirtualBox NAT vs Host-Only, DNS troubleshooting, multi-adapter config |
| Detection Validation | tcpdump packet capture, IDS alert verification, log inspection |

---

## Current Lab State

- ✅ Elasticsearch running
- ✅ Kibana running and accessible
- ✅ Suricata installed and rules updated
- ✅ Custom IDS alerting validated end-to-end
- ✅ Windows-to-Ubuntu host-only traffic confirmed

---

## Next Up — Phase 3

- Install **Sysmon** on Windows 10 VM with the SwiftOnSecurity configuration
- Connect Windows telemetry to Elastic via **Fleet / Elastic Agent**
- Build first **Kibana detection dashboards**

---

> Documenting every step of this build publicly. Follow along: [SOC Analyst Roadmap](https://mmajeed7864.github.io/soclab)
