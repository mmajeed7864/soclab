[README.md](https://github.com/user-attachments/files/28046762/README.md)
# SOC Analyst Lab & Roadmap

A public, end-to-end roadmap for becoming a SOC Analyst (L1 → L2) and Cybersecurity Analyst — built from scratch in a working home lab, documented in real time.

🌐 **Live site:** [mmajeed7864.github.io/soclab](https://mmajeed7864.github.io/soclab)

---

## About

I'm **Mohammed H. Majeed** — a Cloud & Network Operations Technician based in Charlotte, NC, transitioning into SOC and cybersecurity analyst roles.

This repository documents my complete journey from IT support → SOC Analyst → Cybersecurity Analyst → Cloud Security Engineer. Every phase is real lab work, real investigations, real documentation. No theory dumps. No copied content. Built by doing.

It's also a free, public resource for anyone trying to break into security without paying for a bootcamp.

---

## Credentials

**Certifications**
- ✅ CompTIA Security+
- ✅ CompTIA Network+
- ✅ CompTIA CySA+
- 🔄 AWS Certified Security – Specialty (In Progress)
- 🔄 CompTIA A+, Linux+, Data+, PenTest+ (In Progress — via WGU)

**Education**
- 🎓 B.S. Cybersecurity & Information Assurance — Western Governors University (Expected Jan 2027)

**Current Role**
- Cloud & Network Operations Technician — Bay Alarm (Apr 2024 – Present)

**Previous**
- Network & Infrastructure Support Analyst — Apple Inc. (Feb 2023 – Jan 2024)

---

## The Roadmap

**8 tracks. 26 phases.** Visit the [live site](https://mmajeed7864.github.io/soclab) for the full interactive version with detailed step-by-step instructions, tool guides, and goals for each phase. The order is built around real SOC Analyst L1/L2 work: logs, triage, ticketing, phishing, Splunk, endpoint, identity, cloud, and automation.

### Track 1 — Build Your SOC Lab
1. Set Up Your Virtual Lab Environment
2. Deploy Elastic SIEM + Suricata IDS
3. Wazuh XDR — Second Detection Platform

### Track 2 — SOC L1 Core Workflow
4. Windows, Linux, and Network Log Fundamentals
5. SOC L1 Alert Queue, Triage, and Case Notes
6. SOC Ticket Writing, Escalation, and Shift Handoff
7. Phishing and Email Security Investigation
8. Splunk Fundamentals for SOC
9. Detection Engineering and Rule Tuning

### Track 3 — Endpoint and Identity Investigation
10. Build an Active Directory Environment
11. AD Attack Simulation + Alert Triage
12. Endpoint / EDR Investigation with Wazuh, Sysmon, and Defender-Style Telemetry
13. IAM, Hardening, and Least Privilege

### Track 4 — Cybersecurity Analyst Operations
14. Vulnerability Management + Patch Prioritization
15. Threat Intelligence + IOC Enrichment
16. Incident Response Playbooks + Containment Decisions
17. GRC, Compliance, Audit Evidence, and Risk Reporting

### Track 5 — Deep Investigation and External Practice
18. Network Traffic Analysis
19. Malware Triage + Sandbox Analysis
20. Digital Forensics + Memory Basics
21. Threat Hunting
22. External SOC Platforms: LetsDefend, CyberDefenders, and BTLO

### Track 6 — Cloud SOC and Microsoft Security Stack
23. AWS CloudTrail + GuardDuty + Security Hub
24. Microsoft Sentinel + Defender + Entra ID / M365 Investigations

### Track 7 — L2 SOC Automation
25. Python Automation + SOAR Case Automation

### Track 8 — Land the Job
26. Portfolio, Resume, LinkedIn, and Interview Prep

---

## Tech Stack

**SIEM & Detection:** Elastic SIEM · Kibana · Wazuh XDR · Suricata IDS · Sysmon · Fleet · Elastic Agent · Splunk · Microsoft Sentinel
**Cloud Security:** AWS CloudTrail · GuardDuty · Security Hub · IAM · CloudWatch · S3 · VPC · Microsoft Defender for Endpoint
**Endpoint & Identity:** Active Directory · Group Policy · Windows Event Logs · Sysmon · JAMF MDM
**Network & Forensics:** Wireshark · tcpdump · NMAP · pfSense · Volatility · Autopsy · FTK Imager · YARA
**OS & Virtualization:** Windows 10/11 · Ubuntu · Kali Linux · VirtualBox
**Frameworks:** MITRE ATT&CK · NIST CSF · CIS Controls v8 · PICERL

---

## Daily Build Logs

I post lab work as I complete it. See:
- [DAY-01-LAB-LOG.md](./DAY-01-LAB-LOG.md) — Lab foundation + early Elastic SIEM deployment
- [DAY-02-LAB-LOG.md](./DAY-02-LAB-LOG.md) — Phase 2 completion · Windows telemetry · Fleet · Sysmon · Suricata · 5 Kibana dashboards · troubleshooting
- [DAY-03-LAB-LOG.md](./DAY-03-LAB-LOG.md) — Phase 3 completion · Wazuh XDR deployment · Windows agent · FIM · SCA · vulnerability detection · Elastic vs Wazuh comparison

More logs are added with each lab session.

---

## For Recruiters

If you're hiring for **SOC Analyst (L1/L2), Cybersecurity Analyst, Security Operations Analyst, or Junior Cybersecurity Analyst** roles in the Charlotte, NC area or remote — I'd love to connect.

- **LinkedIn:** [linkedin.com/in/mohammed-majeed-40a661271](https://www.linkedin.com/in/mohammed-majeed-40a661271)
- **Email:** hmajeed04@gmail.com
- **Location:** Charlotte, NC (open to remote)

What this portfolio demonstrates:
- Hands-on SIEM, IDS, EDR, and Active Directory experience in a production-style environment
- Real SOC L1 investigations mapped to MITRE ATT&CK
- Documented incident reports, escalation summaries, and IR playbooks
- Self-directed learning and consistent execution over time

---

## For Other Learners

Everything in the roadmap is **free or has a free tier**. No paid bootcamps, no $500 courses. The site includes step-by-step setup instructions and tool guides for every phase.

If you're starting from zero:
1. Begin with **Phase 1** on the live site
2. Read the **Before You Start** section
3. Spin up the lab
4. Work the phases in order

Feel free to fork this repo, copy the structure, and build your own version. The whole point is to lower the barrier for the next person trying to get into security.

---

## License

The content on this site (roadmap structure, instructions, lab guides) is shared for educational use. Feel free to reference or adapt it for your own learning journey.

---

*Built and maintained by Mohammed H. Majeed · Charlotte, NC*