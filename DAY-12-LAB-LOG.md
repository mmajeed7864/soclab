# Phase 12 Detailed Lab Log - Endpoint and EDR Investigation

## Status

**Completed - June 14, 2026**

**Track 3 - Endpoint and Identity Investigation**

## Completion Summary

Phase 12 focused on endpoint and EDR-style investigation using Windows Sysmon
telemetry in Splunk, Wazuh agent verification, and Linux host artifacts. I
completed three controlled investigations:

1. A suspicious PowerShell process chain.
2. Scheduled-task persistence-style behavior.
3. Linux cron persistence.

For each case, I established the host and user context, reviewed process or
artifact evidence, built a timeline, documented what the evidence did and did
not prove, selected a containment action, and cleaned up the simulation.

The activity was generated intentionally inside the home lab. The observed
techniques were security-relevant, but the evidence did not prove malware,
credential theft, command-and-control traffic, or endpoint compromise.

### Final Deliverables

| Requirement | Result |
|---|---|
| Windows process-chain investigation | PowerShell and command-shell activity reviewed with Sysmon Event ID 1 in Splunk |
| Windows persistence investigation | Scheduled task creation and query activity investigated through `schtasks.exe` telemetry |
| Linux endpoint investigation | User cron entry, script, repeated execution, and cleanup documented |
| Endpoint visibility | Splunk/Sysmon evidence reviewed and Wazuh agent availability verified |
| Timeline building | Actor, host, process or artifact, command, time, and follow-on activity documented |
| Containment decisions | Close, remove, scope, or escalate decisions tied to evidence and lab context |
| Evidence boundaries | Each case states what was observed and what could not be concluded |

---

# 1. Objective

The goal was to practice the endpoint-investigation workflow that follows an
EDR or SIEM alert:

```text
Receive suspicious endpoint activity
  -> Confirm endpoint visibility
  -> Identify the user, host, process, and artifact
  -> Reconstruct the timeline
  -> Review parent and child process relationships
  -> Check persistence and follow-on behavior
  -> State what the evidence proves
  -> Select containment or closure
  -> Document cleanup and remaining visibility gaps
```

Phase 12 was not designed to prove that a host had been compromised. It was
designed to show that suspicious endpoint behavior can be investigated without
overstating the conclusion.

---

# 2. Lab Environment and Evidence Sources

| System | Role | Evidence used |
|---|---|---|
| Windows 10 victim | Controlled endpoint activity | Sysmon Event ID 1, process image, command line, parent process, user, host |
| Splunk Enterprise | Central investigation platform | Event search, raw XML review, field extraction, timeline construction |
| Wazuh | Endpoint/XDR visibility | Agent status and endpoint visibility verified separately |
| Ubuntu Linux VM | Linux endpoint simulation | User crontab, script metadata, permissions, timestamped marker output |

## Primary Windows Evidence

Sysmon Event ID 1 records process creation. The most useful fields for these
investigations were:

- `_time`
- `host`
- `User`
- `Image`
- `ParentImage`
- `CommandLine`
- `ParentCommandLine`
- `ProcessId`
- `ParentProcessId`
- `OriginalFileName`
- `Hashes`

When normalized fields were incomplete, I reviewed the raw XML and extracted
the values required for the investigation rather than assuming the telemetry
was absent.

## Primary Linux Evidence

The Linux persistence case used:

- The user crontab entry.
- The referenced script path.
- File ownership and permissions.
- Timestamped script output.
- Repeated execution across multiple minutes.
- Artifact-removal and cleanup verification.

---

# 3. Investigation 1 - Suspicious PowerShell Process Chain

## Scenario

I generated a harmless Windows process chain that used PowerShell with
security-relevant command-line options:

```text
-NoProfile
-ExecutionPolicy Bypass
```

The activity created a marker file named:

```text
phase12_inv1.txt
```

A follow-on `notepad.exe` process was also executed as part of the controlled
scenario.

## Why the Behavior Matters

`-ExecutionPolicy Bypass` and `-NoProfile` can be used by administrators and
legitimate automation, but they also appear frequently in malicious
PowerShell execution. The flags are suspicious by technique, not proof of
malware by themselves.

The investigation therefore needed to answer:

1. Which host and user executed the command?
2. What was the parent process?
3. What exact command line ran?
4. What child or follow-on processes appeared?
5. What file activity was associated with the command?
6. Was there network, credential, persistence, or destructive activity?

## Splunk Search Approach

```spl
index=* host=DESKTOP-3JKM5O9 EventCode=1
("powershell.exe" OR "cmd.exe" OR "notepad.exe" OR "phase12_inv1.txt")
| table _time host User Image ParentImage CommandLine ParentCommandLine ProcessId ParentProcessId
| sort _time
```

If the normalized fields were unavailable, the raw event could be searched
first:

```spl
index=* host=DESKTOP-3JKM5O9 EventCode=1
("ExecutionPolicy Bypass" OR "phase12_inv1.txt" OR "notepad.exe")
| table _time host _raw
| sort _time
```

Raw XML fields could then be extracted with `rex` for a cleaner timeline:

```spl
index=* host=DESKTOP-3JKM5O9 EventCode=1
("ExecutionPolicy Bypass" OR "phase12_inv1.txt" OR "notepad.exe")
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)"
| rex field=_raw "<Data Name='ParentImage'>(?<ParentImage>[^<]+)"
| rex field=_raw "<Data Name='CommandLine'>(?<CommandLine>[^<]+)"
| rex field=_raw "<Data Name='User'>(?<User>[^<]+)"
| table _time host User ParentImage Image CommandLine
| sort _time
```

## Evidence Observed

- Sysmon Event ID 1 process-creation telemetry was searchable in Splunk.
- PowerShell or command-shell execution appeared on the Windows endpoint.
- The command line included `-NoProfile` and `-ExecutionPolicy Bypass`.
- The harmless `phase12_inv1.txt` marker file was created.
- `notepad.exe` appeared as controlled follow-on execution.
- The timeline connected the initiating shell, PowerShell command, and
  follow-on activity.

## SOC Assessment

The evidence proved that suspicious-style PowerShell execution occurred. The
flags and process chain deserved review because they can be used to evade
normal script restrictions or reduce profile-based visibility.

The evidence did **not** prove:

- Malware execution.
- Credential theft.
- Command-and-control communication.
- Privilege escalation.
- Persistence.
- Unauthorized access.
- Endpoint compromise.

The known lab command, harmless marker file, and controlled follow-on process
explained the activity.

**Verdict:** Suspicious by technique, benign by controlled lab context.

## Containment Decision

**Lab decision:** Do not isolate the host. Close the activity as a benign
simulation after confirming the known command and expected artifacts.

**Production response:**

1. Validate whether the user or administrator was authorized to run the
   command.
2. Review the full parent-child process chain.
3. Search for encoded commands, downloads, script files, registry changes, and
   network connections.
4. Scope the same command line or hash across other endpoints.
5. Escalate and isolate if the activity is unexplained or accompanied by
   malicious follow-on evidence.

## Interview-Ready Explanation

> I investigated a suspicious PowerShell process chain using Sysmon process
> creation telemetry in Splunk. I identified the parent process, child process,
> command line, host and user context, and timeline, then made a containment
> decision based on the observed evidence and known lab context.

---

# 4. Investigation 2 - Scheduled Task Persistence-Style Behavior

## Scenario

I created a harmless scheduled task:

| Field | Value |
|---|---|
| Task name | `\Phase12\UpdaterCheck` |
| Action | `notepad.exe` |
| Trigger | `ONLOGON` |
| Run-as user | `MAJEED\Administrator` |
| Parent process | `powershell.exe` |

The task was queried after creation and removed during cleanup.

## Why the Behavior Matters

Scheduled tasks are used legitimately for maintenance and automation. They are
also a common persistence mechanism because they can run a command at logon,
startup, or a recurring time.

The task name, action path, trigger, user, parent process, and change-control
context determine whether the behavior is expected or suspicious.

## Host-Side Validation

The controlled activity used a command similar to:

```powershell
schtasks.exe /Create /TN \Phase12\UpdaterCheck /TR notepad.exe /SC ONLOGON /F
```

The task was then queried to confirm the registered action and trigger.

## Splunk Search Approach

```spl
index=* host=DESKTOP-3JKM5O9 EventCode=1
("schtasks.exe" OR "\\Phase12\\UpdaterCheck" OR "UpdaterCheck")
| table _time host User Image ParentImage CommandLine ParentCommandLine
| sort _time
```

A broader persistence search can include:

```spl
index=* host=DESKTOP-3JKM5O9 EventCode=1
(Image="*\\schtasks.exe" OR CommandLine="*/Create*" OR CommandLine="*/Change*")
| table _time host User ParentImage Image CommandLine
| sort - _time
```

## Evidence Observed

- `schtasks.exe` process creation was recorded by Sysmon and searchable in
  Splunk.
- The command line showed creation of `\Phase12\UpdaterCheck`.
- The action was `notepad.exe`.
- The trigger was `ONLOGON`.
- The parent process was `powershell.exe`.
- The user context was `MAJEED\Administrator`.
- Query activity confirmed the task existed with the expected configuration.
- Cleanup removed the task.

## Timeline

```text
PowerShell launched
  -> schtasks.exe created \Phase12\UpdaterCheck
  -> Task configured to run notepad.exe at logon
  -> schtasks.exe queried the task
  -> Analyst validated the configuration
  -> Task removed
  -> Cleanup verified
```

## SOC Assessment

The evidence proved that a scheduled task was created and queried. The
automatic logon trigger and administrative context made the event
persistence-like and worthy of investigation.

The evidence did **not** prove:

- A malicious payload.
- Credential theft.
- Unauthorized persistence.
- Successful attacker access.
- Endpoint compromise.

The action was harmless and the activity was generated intentionally.

**Verdict:** Persistence-style behavior observed; benign lab simulation.

## Containment Decision

**Lab decision:** Remove the scheduled task and verify that it no longer
exists.

**Production response:**

1. Confirm whether the task has an approved owner or change record.
2. Inspect the task action, arguments, working directory, trigger, and run-as
   account.
3. Review the referenced executable or script reputation and signature.
4. Search for similar tasks across endpoints.
5. Review adjacent process, file, registry, authentication, and network
   telemetry.
6. Remove or disable the task and escalate if it is unauthorized.

## Interview-Ready Explanation

> I investigated a persistence-style endpoint alert by reviewing Sysmon
> process creation telemetry in Splunk. I identified `schtasks.exe`, the parent
> PowerShell process, task name, trigger, action, user context, and cleanup
> evidence. I treated the behavior as suspicious by technique but benign based
> on controlled lab context.

---

# 5. Investigation 3 - Linux Cron Persistence Timeline

## Scenario

I created a harmless user-level cron simulation on the Ubuntu VM.

Script:

```text
/home/mmajeed/phase12_linux/update-check.sh
```

Cron entry:

```cron
* * * * * /home/mmajeed/phase12_linux/update-check.sh
```

Marker output:

```text
/tmp/phase12_linux_marker.log
```

The script wrote a timestamp to the marker file every minute, creating
repeatable execution evidence.

## Why the Behavior Matters

Cron provides normal Linux task scheduling, but attackers can also use user or
root crontabs for recurring execution and persistence. Analysts must establish
who created the entry, what command runs, where the script lives, who owns it,
its permissions, and what execution evidence exists.

## Artifact Review

The investigation reviewed:

```bash
crontab -l
ls -la /home/mmajeed/phase12_linux/
stat /home/mmajeed/phase12_linux/update-check.sh
cat /home/mmajeed/phase12_linux/update-check.sh
cat /tmp/phase12_linux_marker.log
```

Useful production pivots would also include:

```bash
sudo crontab -l
ls -la /etc/cron.d /etc/cron.daily /etc/cron.hourly
systemctl list-timers --all
journalctl -u cron
```

## Evidence Observed

- A user crontab referenced the `update-check.sh` script.
- The script path and content were reviewed.
- Ownership and permissions were available for validation.
- The marker log contained repeated timestamps from multiple executions.
- The repeated output established that the cron entry executed successfully.
- The cron entry, script directory, and marker file were removed during
  cleanup.

## Timeline

```text
Script created in the user's home directory
  -> User crontab updated
  -> Cron launched the script every minute
  -> Marker file received repeated timestamps
  -> Analyst validated path, ownership, permissions, and output
  -> Cron entry removed
  -> Script directory and marker file deleted
  -> Cleanup verified
```

## SOC Assessment

The cron entry, referenced script, and timestamped output proved that recurring
execution occurred. A script launched automatically every minute is
persistence-like and should be investigated when it is unexpected.

The evidence did **not** prove:

- Malware.
- Privilege escalation.
- Credential theft.
- Command-and-control traffic.
- Unauthorized access.
- Host compromise.

The script was harmless and the activity was controlled.

**Verdict:** Linux persistence-style behavior observed; benign lab simulation.

## Containment Decision

**Lab decision:** Remove the cron entry, delete the script and marker file, and
verify that all artifacts are gone.

**Production response:**

1. Review user and root crontabs.
2. Review `/etc/cron*` paths and systemd timers.
3. Inspect scripts in user-writable or temporary directories.
4. Validate file ownership, permissions, timestamps, hashes, and package
   provenance.
5. Review shell history, process telemetry, authentication logs, and network
   connections.
6. Disable the persistence mechanism and isolate the host if the activity is
   unauthorized or linked to malicious follow-on behavior.

## Interview-Ready Explanation

> I investigated Linux persistence by reviewing a user crontab, confirming the
> script path, validating repeated execution through a timestamped output file,
> and performing cleanup. I treated the behavior as suspicious by technique
> but benign based on controlled lab context.

---

# 6. Wazuh Verification and Visibility Limits

Wazuh agent availability was verified as part of the endpoint environment, but
the strongest evidence for the Windows process cases came from Sysmon Event ID
1 in Splunk. I documented that distinction instead of implying every platform
contained identical telemetry.

This matters in a SOC because:

- A connected agent does not guarantee every desired event is collected.
- Different products normalize fields differently.
- A missing alert is not proof that activity did not occur.
- Analysts should pivot to raw events, host artifacts, another sensor, or a
  different time range when visibility is incomplete.

The investigation conclusions therefore identify both the available evidence
and the visibility gaps.

---

# 7. Troubleshooting and How I Fixed It

## Normalized Sysmon Fields Were Incomplete

Some Sysmon values were available in raw XML but were not consistently exposed
as convenient Splunk fields.

**Fix:** I searched the raw event first, validated that the XML contained the
needed values, and used `rex` to extract `Image`, `ParentImage`, `CommandLine`,
and `User` for the investigation table.

## A Missing Field Initially Looked Like Missing Telemetry

Searching only a normalized field can return no results even when the raw event
exists.

**Fix:** I broadened the search to known strings such as the executable, task
name, marker filename, and command-line flags. I then inspected `_raw` before
concluding the event was missing.

## Wazuh and Splunk Did Not Present Identical Evidence

The platforms had different collection and normalization paths.

**Fix:** I documented which sensor supplied each piece of evidence. Splunk and
Sysmon were used for the detailed Windows process chains, while Wazuh agent
status was verified as a separate endpoint-visibility check.

## Linux Investigation Did Not Depend on a SIEM Alert

The Linux cron case used direct endpoint artifacts rather than claiming an EDR
alert existed.

**Fix:** I built the timeline from the crontab, script path, ownership,
permissions, and repeated marker output. This demonstrated a valid host-based
investigation while remaining honest about the evidence source.

## Cleanup Needed Verification

Deleting one file would not fully remove the persistence mechanism.

**Fix:** I removed the cron entry, script directory, and marker file, then
checked that the entry and artifacts were gone.

## General Troubleshooting Method

When endpoint evidence was incomplete, I checked:

1. The correct host and time range.
2. The expected event source and event ID.
3. Raw telemetry before normalized fields.
4. Exact command-line strings and artifact names.
5. Parent and child process relationships.
6. Host artifacts that could confirm execution.
7. Cleanup evidence after containment.

---

# 8. Endpoint Investigation Template

The following structure can be reused for future endpoint alerts.

## Alert Summary

- Alert or behavior:
- Host:
- User:
- First observed:
- Last observed:
- Evidence source:

## Process and Artifact Evidence

- Parent process:
- Process:
- Child process:
- Command line:
- File, registry, task, service, or cron artifact:
- Network activity:
- Hash or reputation:

## Evidence Assessment

- What the evidence proves:
- What the evidence does not prove:
- Known business or lab context:
- Visibility limitations:

## Analyst Decision

- Verdict:
- Severity:
- Containment action:
- Scope required:
- Escalation reason:
- Cleanup verification:

---

# 9. Skills Demonstrated

- Endpoint and EDR investigation workflow.
- Sysmon Event ID 1 process-creation analysis.
- Splunk raw XML review and `rex` field extraction.
- Parent-child process and command-line analysis.
- Suspicious PowerShell investigation.
- Scheduled-task persistence detection.
- Linux cron persistence review.
- Endpoint timeline construction.
- Evidence-safe SOC conclusions.
- Visibility-gap recognition and sensor pivoting.
- Containment and cleanup decisions.
- Documentation suitable for tickets, escalation, and interviews.

---

# 10. Interview-Ready Phase Summary

In Phase 12, I completed three endpoint investigations using Splunk, Sysmon,
Wazuh visibility, and Linux host artifacts. I investigated a suspicious
PowerShell process chain, scheduled-task persistence-style behavior, and a
Linux cron persistence simulation. I built timelines from process and artifact
evidence, reviewed parent-child relationships and command lines, documented
what the evidence did and did not prove, and selected containment or cleanup
actions. The main lesson was that suspicious techniques do not automatically
prove compromise, and missing normalized fields should trigger a pivot to raw
events or host artifacts rather than an unsupported conclusion.

---

# 11. Phase 12 Outcome

Phase 12 is complete.

I can now:

- Reconstruct a Windows endpoint process chain from Sysmon telemetry.
- Review suspicious PowerShell flags in context.
- Investigate scheduled-task persistence-style activity.
- Build a Linux persistence timeline from cron and file artifacts.
- Extract useful fields from raw Sysmon XML in Splunk.
- Separate suspicious behavior from confirmed compromise.
- Explain visibility limitations without treating missing data as proof.
- Choose and document containment, cleanup, scoping, or escalation actions.

## Up Next

**Phase 13 - IAM, Hardening, and Least Privilege**

The next phase uses the identity and endpoint findings from Track 3 to review
administrative exposure, stale accounts, service accounts, local and domain
privilege, logging gaps, and prioritized hardening recommendations.
