# Phase 11 Detailed Lab Log - AD Attack Simulation and Alert Triage

## Status

**Completed - June 11, 2026**

**Track 3 - Endpoint and Identity Investigation**

## Completion Summary

Phase 11 used the Active Directory environment built in Phase 10 to simulate
and investigate five identity-focused security scenarios. I generated
controlled authentication failures, created risky service-account conditions,
requested a Kerberos service ticket, changed privileged group membership, and
ran domain-enumeration commands from a joined Windows workstation.

I investigated the resulting DC01 and endpoint telemetry in Splunk, documented
what each event proved, identified the production context that would still need
validation, and assigned an evidence-safe verdict. The lab activity was
deliberate and controlled. No real compromise, password cracking, credential
theft, malware execution, or domain takeover occurred.

### Final Deliverables

| Requirement | Result |
|---|---|
| Password spray simulation | Investigated Kerberos pre-authentication failures across multiple users |
| AS-REP roastable condition | Created and documented `svc_legacy` with pre-authentication disabled |
| Kerberoast-relevant telemetry | Generated and investigated Event ID 4769 for a lab SPN |
| Privileged group change | Detected addition and removal of a user from Domain Admins |
| AD enumeration | Reviewed process and command evidence from the Windows workstation |
| Investigation platform | Splunk using DC01 and Windows endpoint telemetry |
| Analyst documentation | Five SOC-style assessments with verdicts and follow-up actions |

---

# 1. Objective

The goal was to move beyond building Active Directory and practice the work a
SOC analyst performs after identity alerts appear:

```text
Generate controlled identity activity
  -> Locate the correct telemetry
  -> Establish actor, target, host, and timeline
  -> Decide what the evidence proves
  -> Identify what remains unknown
  -> Assign severity and verdict
  -> Recommend validation or escalation steps
```

The phase goal was to detect, triage, and document five Active Directory
security scenarios.

---

# 2. Lab Environment

| System | Role | Key evidence |
|---|---|---|
| `DC01` | Windows Server 2022 domain controller and DNS | Security events, Kerberos events, account changes, group changes |
| `DESKTOP-3JKM5O9` | Windows 10 domain workstation | User activity, process creation, command-line evidence |
| `splunk-soc` | Splunk Enterprise | Central search, timeline review, field comparison, ticket evidence |
| `majeed.local` | Private lab domain | Controlled users, service accounts, groups, and authentication |

Important lab identities included:

- `MAJEED\Administrator`
- `MAJEED\soc.analyst1`
- `MAJEED\test.employee1`
- `MAJEED\svc_legacy`
- `MAJEED\svc_web`

The scenarios were intentionally generated inside the isolated lab. That
context affected the final verdict, but I still analyzed the behavior as if it
had appeared unexpectedly in a production SOC queue.

---

# 3. Scenario 1 - Password Spray Simulation

## Activity Generated

I generated failed authentication attempts against multiple MAJEED domain
users from the Windows 10 workstation. The intent was to create a pattern that
resembled password spraying: one source testing incorrect credentials across
several accounts within a short time window.

## Evidence Observed

- Host: `DC01`
- Event ID: `4771`
- Kerberos service: `krbtgt/MAJEED`
- Target users included `soc.analyst1` and `test.employee1`
- Failure code: `0x18`, consistent with an incorrect password
- Multiple domain identities were targeted during the controlled test

## Investigation Searches

```spl
index=* host=DC01 EventCode=4771
| table _time host EventCode Account_Name Account_Domain src_ip Failure_Code _raw
| sort - _time
```

```spl
index=* host=DC01 (EventCode=4771 OR EventCode=4776 OR EventCode=4625)
| table _time host EventCode Account_Name Account_Domain src_ip Workstation_Name Failure_Code _raw
| sort - _time
```

## SOC Assessment

Multiple failed Kerberos pre-authentication events against several users from
one source are suspicious because they resemble password spray behavior. The
event pattern justified investigation, but the known source, timing, and lab
test explained the activity.

**Verdict:** Suspicious by behavior, benign by controlled lab context.

**Production follow-up:** Confirm the source host and owner, count targeted
accounts, check for privileged users, review successful logons after the
failures, compare MFA or identity-provider logs, and look for follow-on
activity.

---

# 4. Scenario 2 - AS-REP Roastable Account Configuration

## Activity Generated

I created the service account `svc_legacy` and disabled Kerberos
pre-authentication for the account. I verified the resulting
`DoesNotRequirePreAuth=True` property and searched DC01 telemetry for account
enablement and modification evidence.

## Evidence Observed

- Account: `MAJEED\svc_legacy`
- Risky property: `DoesNotRequirePreAuth=True`
- Event ID `4722`: user account enabled
- Event ID `4738`: user account changed
- Evidence source: `DC01`

## Investigation Search

```spl
index=* host=DC01 ("svc_legacy" OR "Legacy App Service")
| table _time host EventCode Account_Name Account_Domain _raw
| sort - _time
```

## SOC Assessment

Disabling Kerberos pre-authentication creates an AS-REP roastable condition.
The evidence proved that the account was placed in a risky configuration. It
did not prove that an attacker requested material, cracked a password, stole
credentials, or compromised the account.

**Verdict:** Risky configuration documented; benign lab simulation.

**Production follow-up:** Confirm whether the setting is required, re-enable
pre-authentication when possible, rotate the service-account password, verify
ownership and privileges, and monitor Kerberos requests involving the account.

---

# 5. Scenario 3 - Kerberoast-Relevant Service Ticket Request

## Activity Generated

I created the `svc_web` service account and registered the SPN:

```text
HTTP/webapp.majeed.local
```

After enabling and verifying Kerberos Service Ticket Operations auditing, I
used `klist` from the Windows workstation to request a service ticket for the
SPN. I then located the resulting ticket request on DC01.

## Evidence Observed

- Service account: `MAJEED\svc_web`
- SPN: `HTTP/webapp.majeed.local`
- Event ID: `4769`
- Evidence source: `DC01`
- A Kerberos service ticket was requested for the lab web application SPN

## Investigation Search

```spl
index=* host=DC01 EventCode=4769 ("HTTP/webapp.majeed.local" OR "svc_web" OR "webapp")
| table _time host EventCode Account_Name Service_Name Ticket_Encryption_Type Ticket_Options src_ip _raw
| sort - _time
```

## SOC Assessment

An SPN ticket request tied to a service account is Kerberoast-relevant
telemetry. Event 4769 proved that the ticket request occurred. It did not prove
ticket cracking, password recovery, credential theft, or compromise.

**Verdict:** Kerberoast-relevant telemetry detected; benign lab simulation.

**Production follow-up:** Validate the requesting user and host, look for bursts
of ticket requests across many SPNs, review encryption type, confirm whether
the service account is privileged, and search for suspicious authentication or
lateral movement after the request.

---

# 6. Scenario 4 - Domain Admins Group Membership Change

## Activity Generated

I added `test.employee1` to Domain Admins and removed the account immediately
afterward. This created a high-impact identity event with a known beginning and
end state.

## Evidence Observed

- Event ID `4728`: member added to a security-enabled global group
- Event ID `4729`: member removed from a security-enabled global group
- Actor: `MAJEED\Administrator`
- Target: `MAJEED\test.employee1`
- Group: `Domain Admins`
- Evidence source: `DC01`

## Investigation Search

```spl
index=* host=DC01 (EventCode=4728 OR EventCode=4729) "Domain Admins"
| table _time host EventCode Account_Name Account_Domain Group_Name Security_ID _raw
| sort - _time
```

## SOC Assessment

A Domain Admins membership change has high potential impact because it can
grant broad administrative control. Even when the membership is removed
quickly, the analyst must determine whether the change was authorized and
whether the account performed privileged actions while access was available.

**Verdict:** High severity by potential impact; benign lab simulation by known
context.

**Production follow-up:** Validate an approved change record, confirm actor
authorization, verify removal, review the target account's logons, and search
for administrative activity during the membership window.

---

# 7. Scenario 5 - AD Enumeration and Reconnaissance

## Activity Generated

From `DESKTOP-3JKM5O9`, I ran domain discovery and enumeration-style commands
as `MAJEED\soc.analyst1`. Commands included:

```text
whoami /all
nltest /domain_trusts
nltest /dsgetdc:majeed.local
setspn and LDAP-style enumeration attempts
```

Some commands did not return the expected domain data because of RPC, LDAP, or
tool behavior. They still produced useful process and command-line evidence
for endpoint investigation.

## Evidence Observed

- Host: `DESKTOP-3JKM5O9`
- User: `MAJEED\soc.analyst1`
- Process and command-line evidence for identity and domain discovery
- Commands queried user context, domain trusts, domain controller location,
  SPNs, or directory structure

## Investigation Search

```spl
index=* host=DESKTOP-3JKM5O9 earliest=-30m ("whoami" OR "nltest" OR "setspn" OR "domain_trusts" OR "dclist" OR "objectClass")
| table _time host EventCode Image CommandLine ParentImage User _raw
| sort - _time
```

## SOC Assessment

Domain discovery can be normal administrative work, but it becomes suspicious
when a standard user runs it unexpectedly or when it appears alongside other
identity attack behavior. The observed commands resembled early AD
reconnaissance. This phase did not claim that BloodHound was fully deployed.

**Verdict:** Suspicious by behavior, benign by controlled lab context.

**Production follow-up:** Validate the user's role, inspect the parent process,
identify all related commands, and search for subsequent Kerberos requests,
privileged changes, PowerShell activity, credential access, or lateral
movement.

---

# 8. Troubleshooting and How I Fixed It

## Authentication Failures Did Not Always Appear as Event 4625

Searching only for 4625 on the domain controller missed important evidence.
The failed domain authentication path produced Kerberos pre-authentication
events such as 4771 and, depending on the path, credential-validation events
such as 4776.

**Fix:** I broadened the search across 4771, 4776, and 4625, then followed the
fields and raw event data instead of assuming one event ID would contain the
entire story.

## Windows Workstation Had DC, DNS, and RPC Issues

Some enumeration and domain commands failed or returned incomplete results.

**Fix:** I used `nltest` to validate domain controller discovery and confirmed
that the workstation could locate DC01 at `192.168.56.10`. I also rechecked
that the workstation used DC01 for domain DNS.

## Multiple DC01 Network Adapters Affected Registration

The NAT adapter could interfere with name and IP registration while the
host-only adapter was the address intended for the private domain.

**Fix:** I corrected DNS registration so domain clients resolved and reached
the host-only DC01 address.

## Event 4769 Was Initially Missing

The expected service ticket event did not appear until the correct auditing
and ticket-generation path were in place.

**Fix:** I enabled Kerberos Service Ticket Operations auditing, generated a
fresh request with `klist`, and searched the correct time window on DC01.

## Failed Enumeration Commands Still Produced Evidence

A command can fail to retrieve data and still be security-relevant.

**Fix:** I reviewed endpoint process and command-line telemetry. This preserved
evidence of user intent and behavior even when RPC, LDAP, or tool execution did
not return the desired result.

## General Lesson

When an expected event is missing, I now verify:

1. The correct data source and host.
2. The audit policy required to create the event.
3. The search time range.
4. The authentication or execution path.
5. The raw event before relying on normalized fields.

---

# 9. Analyst Decision Framework Used

For each scenario, I answered the same questions:

1. What activity occurred?
2. Which user, host, account, service, or group was involved?
3. What does the evidence directly prove?
4. What does the evidence not prove?
5. Why could the behavior be malicious?
6. What known lab context explains it?
7. What would a production SOC validate next?
8. Should the event be closed, monitored, tuned, or escalated?

This prevented the lab write-up from treating every suspicious event as a
confirmed incident.

---

# 10. Interview-Ready Summary

In Phase 11, I used my Active Directory lab to simulate and investigate five
identity-focused security scenarios. I generated password-spray style failed
authentication, created AS-REP roastable and Kerberoast-relevant
service-account conditions, performed a controlled Domain Admins group
membership change, and ran AD enumeration commands from a domain workstation.
I investigated the activity in Splunk using DC01 and Windows endpoint
telemetry, focusing on Windows Security events such as 4771, 4776, 4769, 4728,
and 4729. For each scenario, I documented what the evidence proved, what it did
not prove, the severity, likely false-positive context, and what a real SOC
would validate before escalation or closure.

---

# 11. Phase 11 Outcome

Phase 11 is complete.

I can now:

- Recognize password-spray patterns in domain authentication telemetry.
- Explain the risk of an AS-REP roastable account without claiming compromise.
- Investigate Kerberoast-relevant Event 4769 activity.
- Triage privileged group membership changes using 4728 and 4729.
- Review AD discovery commands using endpoint process evidence.
- Separate suspicious behavior from a confirmed security incident.
- Write evidence-safe verdicts and practical production follow-up steps.

## Up Next

**Phase 12 - Endpoint / EDR Investigation with Wazuh, Sysmon, and
Defender-Style Telemetry**

The next phase shifts from domain identity events to process trees, device
timelines, persistence evidence, endpoint scoping, and containment decisions.
