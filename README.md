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
- Windows Server 2022 Active Directory domain controller and private lab DNS
- Organized domain users, security groups, and OUs for identity-focused investigations
- Windows 10 domain join and domain authentication validation
- DC01 Security, System, Application, Directory Service, and DNS Server logs forwarded into Splunk
- Kerberos pre-authentication and privileged group-membership investigations using Events 4771, 4776, 4728, and 4729
- Five controlled AD security scenarios covering password spray behavior, AS-REP roastable configuration, Kerberoast-relevant telemetry, privileged group changes, and AD enumeration
- Three endpoint/EDR-style investigations covering a suspicious PowerShell process chain, scheduled-task persistence-style behavior, and Linux cron persistence
- Sysmon Event ID 1 process analysis in Splunk with raw XML field extraction when normalized fields were incomplete
- Windows and Linux endpoint timelines with evidence limits, containment decisions, cleanup, and visibility-gap documentation
- Ten-finding IAM and Active Directory hardening review covering privileged accounts, service accounts, Kerberos settings, password policy, account lockout, test accounts, and endpoint local administrators
- Safe IAM remediation across service-account settings, domain password and lockout policy, an unused test account, and unnecessary local administrator access
- Change-control decisions for access-sensitive and service-dependent findings that should not be changed without ownership review, alternate access, testing, and rollback
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

The full interactive roadmap is on the live site. The order is built around SOC Analyst L1/L2 and Cybersecurity Analyst work: logs, triage, ticketing, phishing, Splunk, endpoint, identity, cloud, incident response, automation, and portfolio proof. It keeps the existing 26 phases and adds lightweight practice checkpoints instead of stretching the roadmap with extra filler phases.

The core SOC analyst tracks also end with a realistic, graded, retakeable capstone assessment before moving to the next track. The capstones simulate real SOC and cybersecurity analyst conditions: timed alert triage, hands-on investigation, documentation, detection logic, and interview-style defense of decisions. They are designed to prove the skills can be applied under pressure, not just that a phase was completed.

### Practice Checkpoints

Starting with the end of Track 3, each core track checkpoint includes:

- One external rep: 2 LetsDefend cases or 1 CyberDefenders lab
- One mixed queue day using alert types from earlier phases
- At least 2 cases where incomplete telemetry requires a defensible non-conclusion
- Severity reasoning that references asset role and business criticality

These checkpoints are reinforcement exercises, not additional numbered phases.

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

Status: complete and capstone passed. Capstone 01 validated the Track 1 lab foundation and Track 2 SOC L1 workflow.

### Track 3 - Endpoint and Identity Investigation

10. Build an Active Directory Environment
11. AD Attack Simulation + Alert Triage
12. Endpoint / EDR Investigation with Wazuh, Sysmon, Defender-Style Telemetry, and one Linux victim investigation
13. IAM, Hardening, and Least Privilege

Status: Phases 10-13 complete. Track 3 now includes the AD foundation, five controlled identity-security investigations, three endpoint/EDR-style investigations, and a 10-finding IAM and hardening review with safe remediation and documented change-control boundaries. Practice Checkpoint 01 and Capstone 02 are next.

### Track 4 - Cybersecurity Analyst Operations

14. Vulnerability Management + Patch Prioritization
15. Threat Intelligence + IOC Enrichment
16. Incident Response Playbooks + Containment Decisions, including execution of one published third-party runbook
17. GRC, Compliance, Audit Evidence, and Risk Reporting

### Track 5 - Deep Investigation and External Practice

18. Network Traffic Analysis
19. Malware Triage + Sandbox Analysis
20. Digital Forensics + Memory Basics
21. Threat Hunting
22. Scale External SOC Practice + Unfamiliar Case Investigations

### Track 6 - Cloud SOC and Microsoft Security Stack

23. AWS CloudTrail + GuardDuty + Security Hub
24. Microsoft Sentinel + Defender + Entra ID / M365 Investigations

### Track 7 - L2 SOC Automation

25. Python Automation + SOAR Case Automation

### Track 8 - Land the Job

26. Portfolio, Resume, LinkedIn, and Interview Prep

### Track-End Capstone Assessments

The core SOC analyst tracks end with a capstone checkpoint (outside the numbered 26 phases) that must be completed before moving on:

- Capstone 01 - SOC L1 Core Workflow Readiness (after Phase 9, covers Tracks 1-2) - passed
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
- External unfamiliar-case investigations and mixed queue days
- Defensible non-conclusions when telemetry is incomplete
- Asset criticality in severity decisions
- False-positive tuning
- Ticket writing
- ServiceNow and Jira case tracking, with optional hands-on workflow practice in a ServiceNow developer instance or TheHive
- Escalation summaries
- Shift handoff notes
- Phishing triage
- IOC extraction
- IOC enrichment with VirusTotal, AbuseIPDB, and Shodan
- Email security incident reports
- Incident reports and playbooks
- Third-party runbook execution and exception documentation

Endpoint and identity:

- Windows Event Logs, Sysmon, Microsoft Defender for Endpoint telemetry
- Linux endpoint investigation using authentication, cron, process, and file-timestamp evidence
- Active Directory Domain Services, Windows Server, domain controller administration, DNS, OUs, users, security groups, domain join, and Kerberos authentication
- Splunk investigation of Windows identity events including 4771, 4776, 4769, 4728, and 4729
- Password-spray investigation, AS-REP roastable configuration review, Kerberoast-relevant telemetry, privileged group monitoring, and AD enumeration analysis
- PowerShell process-chain analysis and scheduled-task persistence investigation using Sysmon Event ID 1 in Splunk
- Linux cron persistence review using crontab, script, permission, timestamp, execution, and cleanup evidence
- Raw Sysmon XML field extraction, endpoint timeline building, containment decisions, and visibility-gap documentation
- Evidence-safe identity triage that separates suspicious behavior from confirmed compromise
- IAM and least-privilege review of domain users, privileged groups, service accounts, SPNs, Kerberos pre-authentication, enabled test identities, and endpoint local administrators
- Active Directory password and account lockout policy review and remediation
- Service-account ownership, managed credential, gMSA, and Windows LAPS recommendations
- Risk-rated hardening findings with validation, residual-risk notes, and change-control decisions

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
- [DAY-10-LAB-LOG.md](./DAY-10-LAB-LOG.md) - Windows Server Active Directory domain build, DNS, users/groups/OUs, Windows domain join, DC01 Splunk forwarding, Kerberos failures, privileged group changes, troubleshooting, and analyst mini-tickets
- [DAY-11-LAB-LOG.md](./DAY-11-LAB-LOG.md) - Five controlled AD security scenarios, Splunk identity triage, evidence-safe verdicts, and troubleshooting
- [DAY-12-LAB-LOG.md](./DAY-12-LAB-LOG.md) - Windows and Linux endpoint investigations covering PowerShell process chains, scheduled-task and cron persistence, raw Sysmon XML analysis, containment decisions, cleanup, and evidence limits
- [DAY-13-LAB-LOG.md](./DAY-13-LAB-LOG.md) - Ten-finding IAM and hardening review covering service accounts, Kerberos settings, password and lockout policy, stale accounts, local administrators, safe remediation, and change-control decisions
- [capstone-01-soc-l1-core-workflow-readiness-assessment.md](./capstone-01-soc-l1-core-workflow-readiness-assessment.md) - Capstone 01 results log validating the Track 1 foundation and Track 2 SOC L1 workflow

More logs are added with each lab session.

## For Recruiters

If you're hiring for SOC Analyst L1/L2, Cybersecurity Analyst, Security Operations Analyst, or Junior Cybersecurity Analyst roles in Charlotte, NC or remote, this portfolio demonstrates:

- Hands-on SIEM, IDS, XDR, endpoint, and Active Directory lab experience
- Built a Windows Server domain controller, joined a Windows endpoint to the domain, and centralized DC01 identity logs in Splunk
- Investigated Kerberos pre-authentication failures and privileged group membership changes with evidence-safe SOC conclusions
- Simulated and triaged five identity-security scenarios covering password spray behavior, risky service-account configurations, Kerberos service-ticket activity, Domain Admins changes, and AD enumeration
- Investigated suspicious PowerShell execution, scheduled-task persistence-style behavior, and Linux cron persistence using Sysmon/Splunk telemetry and Linux host artifacts
- Built endpoint timelines, extracted fields from raw Sysmon XML, documented visibility gaps, and made evidence-based containment and cleanup decisions
- Completed a 10-finding IAM and Active Directory hardening review with risk ratings, safe remediation, and documented production change-control recommendations
- Strengthened domain password and lockout policy, corrected risky service-account settings, disabled an unused test account, and removed unnecessary endpoint administrator access
- Current SOC analyst responsibilities across Splunk, Elastic, Wazuh, Microsoft Defender for Endpoint, phishing investigation, IOC enrichment, ServiceNow/Jira ticketing, and escalation handoff
- Assessment proof: Passed the cumulative SOC L1 Core Workflow Readiness Assessment covering Phases 1-9
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

No certification is required to start, but Network+ and Security+ level concepts are helpful. At a steady weekly pace, the core lab can be completed in about three months, with the broader L2, cloud, automation, and portfolio path designed for roughly five months of focused work.

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
10. Use Phase 10 to build AD, join a Windows endpoint, centralize DC logs, and learn where identity evidence lives
11. Use Phase 11 for controlled identity attack simulation and evidence-safe investigation
12. Use Phase 12 to investigate endpoint process trees, persistence evidence, device timelines, and containment decisions
13. Use Phase 13 to review IAM exposure, harden safe settings, and document changes that require controlled implementation
14. Complete Practice Checkpoint 01 after Track 3: unfamiliar external cases, a mixed queue day, incomplete-evidence decisions, and asset-aware severity
15. Keep adding detections, reports, capstone results, and lessons learned

Feel free to fork the repo, copy the structure, and build your own version.

## License

The content on this site is shared for educational use. You can reference or adapt it for your own learning journey.

Built and maintained by Mohammed H. Majeed - Charlotte, NC
