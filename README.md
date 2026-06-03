[README (3).md](https://github.com/user-attachments/files/28418704/README.3.md)
# SOC Analyst Lab & Roadmap

A public, end-to-end roadmap for becoming a SOC Analyst L1/L2 and Cybersecurity Analyst, built from a real home lab and documented as the work progresses.

Live site: [mmajeed7864.github.io/soclab](https://mmajeed7864.github.io/soclab)

## About

I'm Mohammed H. Majeed, a SOC Analyst based in Charlotte, NC, building toward SOC L1/L2 and cybersecurity analyst roles.

This repository documents my hands-on path through security operations: SIEM deployment, endpoint telemetry, IDS alerts, Wazuh XDR, Microsoft Defender for Endpoint, phishing investigation, IOC enrichment, log analysis, high-volume triage practice, incident notes, detection engineering, cloud security monitoring, and portfolio-ready reporting.

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
- Phishing investigation, IOC extraction, and email security ticket writing
- Final phishing incident report with verdicts and recommended response actions
- Dedicated Splunk Enterprise server deployment
- Splunk Universal Forwarder endpoint telemetry pipeline
- Windows Security and Sysmon telemetry searchable in Splunk
- 10 SPL investigations across authentication, DNS, PowerShell, and process activity
- Three Splunk SOC dashboards for authentication, endpoint, and DNS review
- Splunk/SPL vs Elastic/KQL workflow comparison
- Six enabled Splunk detection alerts across process execution, persistence, privileged account changes, network connections, and DNS telemetry
- Three documented Sigma-style rules with MITRE ATT&CK mapping and false-positive notes
- Before/after false-positive tuning on outbound network activity (38 benign OneDrive matches reduced to 0)
- Honest deferral of unsupported LSASS and Certutil validation paths
- Case notes and lessons learned added to build logs

This is designed to practice the real SOC motion: sort signal from noise, decide what deserves more time, close benign activity, tune recurring noise, write clean tickets, investigate suspicious emails, extract IOCs, search telemetry in multiple SIEMs, build dashboards, and escalate cleanly.

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

- SOC Analyst, Bay Alarm, April 2024 to present

Previous role:

- Network & Infrastructure Support Analyst, Apple Inc., February 2023 to January 2024

## Roadmap

The full interactive roadmap is on the live site. The order is built around SOC Analyst L1/L2 and Cybersecurity Analyst work: logs, triage, ticketing, phishing, Splunk, endpoint, identity, cloud, incident response, automation, and portfolio proof.

The core SOC analyst tracks also end with a realistic, graded, retakeable capstone assessment before moving to the next track. The capstones simulate real SOC and cybersecurity analyst conditions: timed alert triage, hands-on investigation, documentation, detection logic, and interview-style defense of decisions. They are designed to prove the skills can be applied under pressure, not just that a phase was completed.

### Track 1 - Build Your SOC Lab

1. Set Up Your Virtual Lab Environment
2. Deploy Elastic SIEM + Suricata IDS
3. Wazuh XDR - Second Detection Platform

Status: complete and validated through Capstone 01.

### Track 2 - SOC L1 Core Workflow

4. Windows, Linux, and Network Log Fundamentals
5. Scripted Attack Loop + High-Volume Triage
6. SOC Ticket Writing, Escalation, and Shift Handoff
7. Phishing and Email Security Investigation
8. Splunk Fundamentals for SOC
9. Detection Engineering and Rule Tuning

Status: complete and capstone passed. Capstone 01 was passed with a 90% SOC readiness rating and 90% SOC level.

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

### Track-End Capstone Assessments

The core SOC analyst tracks end with a capstone checkpoint (outside the numbered 26 phases) that must be completed before moving on:

- Capstone 01 - SOC L1 Core Workflow Readiness (after Phase 9, covers Tracks 1-2) - passed, 90% SOC readiness, 90% SOC level
- Capstone 02 - Endpoint and Identity Investigation (after Phase 13)
- Capstone 03 - Cybersecurity Analyst Operations (after Phase 17)
- Capstone 04 - Deep Investigation Capstone (after Phase 22, before Track 6)

## Technical Skills

SIEM and detection:

- Splunk, Elastic SIEM, Kibana, Wazuh XDR, Suricata IDS, Sysmon, Fleet, Elastic Agent, Splunk Universal Forwarder, Microsoft Defender for Endpoint, Microsoft Sentinel
- KQL, SPL, detection rules, alert tuning, MITRE ATT&CK mapping

SOC operations:

- High-volume alert triage
- Alert queue management
- 30-second first-look triage
- False-positive tuning
- Ticket writing
- ServiceNow and Jira case tracking
- Escalation summaries
- Shift handoff notes
- Phishing triage
- IOC extraction
- IOC enrichment with VirusTotal, AbuseIPDB, and Shodan
- Email security incident reports
- Incident reports and playbooks

Endpoint and identity:

- Windows Event Logs, Sysmon, Microsoft Defender for Endpoint telemetry
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

Email security and phishing:

- Email header review, SPF, DKIM, DMARC, reply-to, return-path, and sender domain analysis
- URL analysis, attachment review, credential harvesting detection, BEC recognition, IOC enrichment, and phishing verdict writing

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
- [DAY-07-LAB-LOG.md](./DAY-07-LAB-LOG.md) - Phishing investigation, IOC extraction, email security triage, phishing tickets, and final incident report
- [DAY-08-LAB-LOG.md](./DAY-08-LAB-LOG.md) - Splunk Enterprise deployment, Universal Forwarder pipeline, Windows/Sysmon telemetry, SPL investigations, dashboards, and Splunk vs Elastic workflow comparison
- [DAY-09-LAB-LOG.md](./DAY-09-LAB-LOG.md) - Detection engineering in Splunk: six enabled alerts, three Sigma-style rules, before/after false-positive tuning, and honest deferral of unsupported LSASS and Certutil validation paths
- [capstone-01-soc-l1-core-workflow-readiness-assessment.md](./capstone-01-soc-l1-core-workflow-readiness-assessment.md) - Capstone 01 results log: passed with a 90% SOC readiness rating and 90% SOC level, validating Track 1 foundation and Track 2 SOC L1 workflow

More logs are added with each lab session.

## For Recruiters

If you're hiring for SOC Analyst L1/L2, Cybersecurity Analyst, Security Operations Analyst, or Junior Cybersecurity Analyst roles in Charlotte, NC or remote, this portfolio demonstrates:

- Hands-on SIEM, IDS, XDR, endpoint, and Active Directory lab experience
- Current SOC analyst responsibilities across Splunk, Elastic, Wazuh, Microsoft Defender for Endpoint, phishing investigation, IOC enrichment, ServiceNow/Jira ticketing, and escalation handoff
- Assessment proof: Passed the cumulative SOC L1 Core Workflow Readiness Assessment with a 90% SOC readiness rating and 90% SOC level
- High-volume alert triage practice with scripted lab-generated alert volume
- Realistic false-positive tuning and escalation workflows
- SOC-style documentation, ticket notes, incident summaries, and build logs
- Written SOC tickets, severity justification, escalation summaries, and shift handoff examples
- Phishing investigation workflow, extracted IOCs, user-impact questions, and response recommendations
- Splunk Enterprise, Universal Forwarder, SPL investigations, and SOC dashboard evidence
- Detection engineering in Splunk: enabled alert rules, Sigma-style logic, MITRE ATT&CK mapping, and before/after false-positive tuning
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
6. Use Phase 7 to practice phishing investigation, IOC extraction, and email security reports
7. Use Phase 8 to practice Splunk/SPL investigations and dashboards without disrupting the Elastic lab
8. Use Phase 9 to turn investigations into enabled detections, write Sigma-style rules, and tune false positives
9. Complete and document the Track 1-2 capstone assessment before moving into Active Directory work
10. Keep adding detections, reports, capstone results, and lessons learned

Feel free to fork the repo, copy the structure, and build your own version.

## License

The content on this site is shared for educational use. You can reference or adapt it for your own learning journey.

Built and maintained by Mohammed H. Majeed - Charlotte, NC
