# Capstone 01 - SOC L1 Core Workflow Readiness Assessment

## Assessment Status

```text
Status: PASSED
SOC Readiness Rating: 90%
SOC Level: 90%
Sections Completed: All
Mandatory Remediation Required: None before Phase 10
Next Roadmap Step: Phase 10 - Build an Active Directory Environment
```

## Assessment Scope

Capstone 01 assessed the combined skills from the first two tracks of the SOC & Cybersecurity Analyst Roadmap:

```text
Track 1 - Build Your SOC Lab
Track 2 - SOC L1 Core Workflow
Coverage: Phases 1-9
```

This assessment was designed to test whether the lab work could be applied in a realistic SOC and cybersecurity analyst workflow. It was not a memorization quiz. The focus was practical analyst reasoning, evidence-based decision-making, documentation, investigation workflow, detection tuning, and interview-ready explanation of technical choices.

The final result validates that the Track 1 lab foundation and Track 2 SOC L1 workflow are ready to support the next roadmap stage: endpoint and identity investigation.

## Capstone Sections Completed

### Part 1 - SOC Knowledge Review

Completed a SOC knowledge review covering:

- Virtual lab architecture
- Telemetry flow from endpoint to SIEM
- Windows Security and Sysmon evidence
- Alert triage and severity logic
- Splunk alerting
- Detection tuning
- SOC documentation
- Analyst communication

Result: completed successfully as part of the passed capstone.

### Part 2 - Timed Alert Queue Simulation

Completed a timed alert queue simulation that required fast first-look decisions across mixed alert types, including:

- Authentication events
- Phishing indicators
- Encoded PowerShell
- Suspicious download activity
- Discovery commands
- Persistence activity
- DNS and network evidence
- Approved software activity
- Vulnerability findings
- Domain-controller service installation context

Result: completed successfully as part of the passed capstone.

### Part 3 - Hands-On Splunk Investigation

Completed hands-on Splunk investigation work using real lab telemetry and screenshots. The investigation work focused on evidence correlation, timeline building, severity adjustment, and deciding whether activity should be closed, tuned, investigated further, or escalated.

Result: strong hands-on performance.

### Part 4 - SOC Documentation Package

Completed SOC documentation work, including:

- SOC ticket writing
- Evidence summary
- Shift handoff note
- Closure reasoning
- Interview-ready explanation of the final analyst decision

Result: strong documentation performance.

### Part 5 - Detection Engineering and Tuning

Completed a detection engineering challenge focused on privileged account activity. The detection logic retained high-value security visibility while classifying known lab validation activity instead of hiding it.

Result: strong detection and tuning performance.

### Part 6 - Interview Defense

Completed an interview-style defense explaining investigation decisions, detection logic, why evidence supported closure, and why unsupported telemetry paths should be documented rather than overclaimed.

Result: strong analyst communication performance.

## Hands-On Case 1 - Suspicious Execution and Persistence Review

### Summary

Investigated suspicious PowerShell process-creation activity on the Windows Victim endpoint in Splunk using Sysmon telemetry. The investigation identified PowerShell execution with the `-EncodedCommand` indicator and then reviewed whether persistence was established through a Windows Registry Run key.

### Evidence Reviewed

| Evidence Area | Finding |
|---|---|
| Host | `DESKTOP-3JKM5O9` |
| User context | `DESKTOP-3JKM5O9\mmajeed` |
| Telemetry source | Sysmon Operational logs collected in Splunk |
| Process evidence | Sysmon Event ID 1 showed PowerShell process creation using `-NoProfile -EncodedCommand` |
| Registry evidence | Sysmon Event ID 13 showed a Registry Run-key value named `Phase9Test` |
| Payload path | `C:\Windows\System32\notepad.exe` |
| Cleanup evidence | Sysmon Event ID 12 confirmed the `Phase9Test` Run-key value was deleted shortly after creation |

### Analyst Verdict

A persistence-capable Registry Run-key mechanism was created, but the `Phase9Test` naming, harmless Notepad payload, known lab user context, and cleanup evidence confirmed controlled lab validation activity rather than active malicious persistence.

The persistence artifact was closed as low-severity benign lab activity after evidence confirmed it did not remain active.

### Skills Demonstrated

- SPL investigation of Sysmon process telemetry
- Encoded PowerShell identification
- Registry Run-key persistence review
- Creation-versus-cleanup timeline validation
- Evidence-based severity adjustment
- Avoidance of unsupported escalation

## Hands-On Case 2 - Privileged Account Change Investigation

### Summary

Investigated local administrator membership activity on the Windows Victim endpoint using Windows Security telemetry in Splunk. The objective was to determine whether temporary privileged accounts remained active or represented controlled lab validation.

### Evidence Reviewed

| Evidence Area | Finding |
|---|---|
| Host | `DESKTOP-3JKM5O9` |
| Actor | `DESKTOP-3JKM5O9\mmajeed` |
| Account investigated | `Phase9TempAdmin` |
| Group affected | `Builtin\Administrators` |
| Account creation | Windows Security Event ID 4720 |
| Administrator addition | Windows Security Event ID 4732 |
| Administrator removal | Windows Security Event ID 4733 |
| Account deletion | Windows Security Event ID 4726 |

### Timeline Findings

- A `Phase9TempAdmin` account was created, added to Administrators, removed from Administrators, and deleted.
- A second account with the same `Phase9TempAdmin` name but a distinct SID was also created, added to Administrators, removed, and deleted.
- No `Phase9TempAdmin` administrator access remained active.

### Analyst Verdict

The full lifecycle confirmed benign controlled lab validation activity. Adding a local administrator is security-significant and required investigation, but Splunk evidence proved the temporary privileged accounts were removed and deleted, so escalation was not required.

### Skills Demonstrated

- Windows Security Event investigation in Splunk
- Local administrator privilege-change analysis
- SID-based correlation of separate account instances
- Lifecycle reconstruction across account creation, privilege grant, privilege removal, and deletion
- Professional case closure and no-escalation justification

## SOC Documentation Package

The capstone included a completed SOC documentation package for privileged-account activity.

Documentation included:

- Host and actor identification
- Affected administrator group
- Event timeline
- Evidence summary
- Cleanup confirmation
- Low-severity closure decision
- Shift handoff guidance
- Interview-ready explanation of why escalation was not required

This documentation demonstrates the ability to turn technical evidence into a clear analyst record another SOC team member could understand.

## Detection Engineering and Tuning Challenge

### Detection Focus

Built a context-aware Splunk detection using Windows Security Event ID `4732` to identify accounts added to the local `Administrators` group.

### Detection Reasoning

The detection logic was designed to:

- Identify local Administrators membership additions
- Extract event time, host, actor account, member added, unique SID, and affected group
- Classify known `Phase9TempAdmin` test activity as known lab validation
- Label other additions for review as unexpected administrator additions
- Preserve all privileged-account events for analyst visibility

### Tuning Decision

Known lab activity was classified rather than suppressed. This was the right tuning decision because local administrator membership changes are high-value security events. Hiding them with a broad exclusion could create a blind spot if similar behavior occurred later outside the lab context.

## Interview Defense

The final interview-style section required explaining:

- How suspicious Registry Run-key activity was investigated and closed only after cleanup was verified
- Why local administrator additions should stay visible even when known lab test activity is classified
- Why LSASS validation was deferred when Sysmon Event ID 10 telemetry was not available
- Why unsupported telemetry paths should be documented honestly instead of overclaimed
- How the work maps to SOC L1 analyst responsibilities

## Strengths Demonstrated

Major strengths shown in Capstone 01:

- Hands-on Splunk investigation
- Sysmon and Windows Security evidence correlation
- Privileged-account lifecycle analysis
- Registry Run-key persistence verification
- SOC ticket and shift-handoff writing
- Detection classification and tuning reasoning
- Interview defense of analyst decisions
- Multi-VM SOC lab management

## Continued Practice Areas

No mandatory remediation or retake is required before Phase 10. Continued practice areas for sharpening speed and precision include:

- Faster recall of event IDs and common SOC concepts
- More precise wording when an event is suspicious but not confirmed malicious
- Continued close-versus-tune decision practice for benign alert noise
- More Elastic/KQL versus Splunk/SPL workflow comparison

These are normal growth areas and do not block progression into the next roadmap phase.

## Portfolio Translation

```text
Completed and passed Capstone 01, a cumulative SOC L1 workflow assessment covering the first two roadmap tracks and Phases 1-9. The assessment validated SIEM workflows, Windows/Sysmon evidence, alert triage, hands-on Splunk investigations, SOC documentation, detection tuning, and interview-style defense of analyst decisions.
```

## Final Verdict

```text
Capstone 01 Passed
Track 1 Foundation + Track 2 SOC L1 Core Workflow validated
SOC Readiness: 90%
SOC Level: 90%
Next Progression: Phase 10 - Build an Active Directory Environment
```
