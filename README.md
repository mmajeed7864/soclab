# SOC Analyst Lab & Roadmap

A public, end-to-end roadmap for becoming a SOC Analyst L1/L2 and Cybersecurity Analyst, built from a real home lab and documented as the work progresses.

Live site: [mmajeed7864.github.io/soclab](https://mmajeed7864.github.io/soclab)

## About

I'm Mohammed H. Majeed, a Cloud & Network Operations Technician based in Charlotte, NC, transitioning into SOC and cybersecurity analyst roles.

This repository documents my hands-on path from IT operations into security operations: SIEM deployment, endpoint telemetry, IDS alerts, Wazuh XDR, log analysis, high-volume triage practice, Active Directory attacks, incident notes, cloud security monitoring, and portfolio-ready reporting.

The goal is simple: show real job-function proof, not just screenshots or theory.

## Current Focus

The roadmap now includes a production-style SOC practice layer:

- Scripted lab attack loop using a Kali attacker VM
- 30-60 minute alert generation sessions
- 50+ alerts/tickets per triage session
- Elastic and Wazuh queue review
- 30-second first-look triage decisions
- False-positive tuning decisions
- SOC ticket writing, severity reasoning, and shift handoff notes
- L2-ready escalation notes
- Case notes and lessons learned added to build logs

This is designed to practice the real SOC motion: sort signal from noise, decide what deserves more time, close benign activity, tune recurring noise, write clean tickets, and escalate cleanly.

## Credentials

Certifications that matter most for the target roles:

- CompTIA Network+ - earned
- CompTIA Security+ - earned
- CompTIA CySA+ - earned
- CompTIA PenTest+ - WGU track
- CCSP - (ISC)2 cloud security track
- AWS Certified Security - Specialty - in progress

WGU cybersecurity degree certification map:

- CompTIA A+, Network+, Security+, Project+, CySA+, PenTest+, and Data+
- CompTIA IT Operations Specialist, Secure Infrastructure Specialist, Network Vulnerability Assessment Professional, Network Security Professional, and Security Analytics Professional
- ITIL 4 Foundation
- LPI Linux Essentials
- SSCP - (ISC)2
- CCSP - (ISC)2
- Current site carryovers: CompTIA Linux+ study and AWS Security Specialty external cloud target

Education:

- B.S. Cybersecurity & Information Assurance, Western Governors University, expected January 2027

Current role:

- Cloud & Network Operations Technician, Bay Alarm, April 2024 to present

Previous role:

- Network & Infrastructure Support Analyst, Apple Inc., February 2023 to January 2024

## Roadmap

The full interactive roadmap is on the live site. The order is built around SOC Analyst L1/L2 and Cybersecurity Analyst work: logs, triage, ticketing, phishing, Splunk, endpoint, identity, cloud, incident response, automation, and portfolio proof.

### Track 1 - Build Your SOC Lab

1. Set Up Your Virtual Lab Environment
2. Deploy Elastic SIEM + Suricata IDS
3. Wazuh XDR - Second Detection Platform

### Track 2 - SOC L1 Core Workflow

4. Windows, Linux, and Network Log Fundamentals
5. Scripted Attack Loop + High-Volume Triage
6. SOC Ticket Writing, Escalation, and Shift Handoff
7. Phishing and Email Security Investigation
8. Splunk Fundamentals for SOC
9. Detection Engineering and Rule Tuning

### Track 3 - Endpoint and Identity Investigation

10. Build an Active Directory Environment
11. AD Attack Simulation + Alert Triage
12. Endpoint / EDR Investigation with Wazuh, Sysmon, and Defender-Style Telemetry
13. IAM, Hardening, and Least Privilege

### Track 4 - Cybersecurity Analyst Operations

14. Vulnerability Management + Patch Prioritization
15. Threat Intelligence + IOC Enrichment
16. Incident Response Playbooks + Containment Decisions
17. GRC, Compliance, Audit Evidence, and Risk Reporting

### Track 5 - Deep Investigation and External Practice

18. Network Traffic Analysis
19. Malware Triage + Sandbox Analysis
20. Digital Forensics + Memory Basics
21. Threat Hunting
22. External SOC Platforms: LetsDefend, CyberDefenders, and BTLO

### Track 6 - Cloud SOC and Microsoft Security Stack

23. AWS CloudTrail + GuardDuty + Security Hub
24. Microsoft Sentinel + Defender + Entra ID / M365 Investigations

### Track 7 - L2 SOC Automation

25. Python Automation + SOAR Case Automation

### Track 8 - Land the Job

26. Portfolio, Resume, LinkedIn, and Interview Prep

## Technical Skills

SIEM and detection:

- Elastic SIEM, Kibana, Wazuh XDR, Suricata IDS, Sysmon, Fleet, Elastic Agent, Splunk, Microsoft Sentinel
- KQL, SPL, detection rules, alert tuning, MITRE ATT&CK mapping

SOC operations:

- High-volume alert triage
- Alert queue management
- 30-second first-look triage
- False-positive tuning
- Ticket writing
- Escalation summaries
- Shift handoff notes
- Incident reports and playbooks

Endpoint and identity:

- Windows Event Logs, Sysmon, Microsoft Defender-style telemetry
- Active Directory, Group Policy, Windows Server, BloodHound
- Password spraying and Kerberoasting detection

Network and infrastructure:

- Wireshark, tcpdump, Nmap, Suricata, pfSense
- VMware Workstation, VirtualBox, UTM, Ubuntu Server, Kali Linux, Windows 10/11
- Host-only networking, NAT, VM snapshots, isolated lab design

Adversary simulation and investigation:

- Kali Linux, scripted attack loops, Metasploit, Impacket, CrackMapExec, Mimikatz
- Volatility, Autopsy, FTK Imager, PEStudio, YARA

Cloud and Microsoft security:

- AWS CloudTrail, GuardDuty, Security Hub, IAM, CloudWatch
- Azure, Microsoft 365, Entra ID, Defender, Sentinel

Frameworks and reporting:

- MITRE ATT&CK, NIST CSF, CIS Controls v8, PICERL
- Case notes, dashboards, GitHub documentation, analyst-style writeups

## Daily Build Logs

I post lab work as I complete it:

- [DAY-01-LAB-LOG.md](./DAY-01-LAB-LOG.md) - Virtual lab foundation, NAT/host-only networking, VM roles, SSH, snapshots, and troubleshooting
- [DAY-02-LAB-LOG.md](./DAY-02-LAB-LOG.md) - Windows telemetry, Fleet, Sysmon, Suricata, dashboards, and troubleshooting
- [DAY-03-LAB-LOG.md](./DAY-03-LAB-LOG.md) - Wazuh XDR deployment, Windows agent, FIM, SCA, vulnerability detection, and Elastic vs Wazuh comparison
- [DAY-04-LAB-LOG.md](./DAY-04-LAB-LOG.md) - Windows, Linux, and network log fundamentals
- [DAY-05-LAB-LOG.md](./DAY-05-LAB-LOG.md) - Scripted alert loop, high-volume SOC triage, false-positive decisions, escalation notes, and shift handoff
- [DAY-06-LAB-LOG.md](./DAY-06-LAB-LOG.md) - SOC ticket writing, severity reasoning, escalation summaries, and shift handoff notes

More logs are added with each lab session.

## For Recruiters

If you're hiring for SOC Analyst L1/L2, Cybersecurity Analyst, Security Operations Analyst, or Junior Cybersecurity Analyst roles in Charlotte, NC or remote, this portfolio demonstrates:

- Hands-on SIEM, IDS, XDR, endpoint, and Active Directory lab experience
- High-volume alert triage practice with scripted lab-generated alert volume
- Realistic false-positive tuning and escalation workflows
- SOC-style documentation, ticket notes, incident summaries, and build logs
- Written SOC tickets, severity justification, escalation summaries, and shift handoff examples
- Self-directed learning and consistent execution over time

LinkedIn: [linkedin.com/in/mohammed-majeed-40a661271](https://www.linkedin.com/in/mohammed-majeed-40a661271)  
Email: hmajeed04@gmail.com  
Location: Charlotte, NC, open to remote

## For Other Learners

Everything in the roadmap is free or has a free tier. The live site includes step-by-step setup guidance, tool lists, phase goals, and daily build logs.

If you're starting from zero:

1. Start with Phase 1 on the live site
2. Build the lab foundation
3. Document every phase in GitHub
4. Use Phase 5 to practice realistic SOC triage repetition, not just tool setup
5. Use Phase 6 to turn triage evidence into tickets, escalation summaries, and shift handoff notes
6. Keep adding detections, reports, and lessons learned

Feel free to fork the repo, copy the structure, and build your own version.

## License

The content on this site is shared for educational use. You can reference or adapt it for your own learning journey.

Built and maintained by Mohammed H. Majeed - Charlotte, NC
