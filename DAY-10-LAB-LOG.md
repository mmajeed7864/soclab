# Phase 10 Detailed Lab Log - Active Directory Environment and Splunk Visibility

## Status

**Completed - June 9, 2026**

**Track 3 - Endpoint and Identity Investigation**

## Completion Summary

Phase 10 established the identity foundation for the next part of the SOC
roadmap. I built a working Active Directory lab with a Windows Server 2022
domain controller, private lab DNS, realistic organizational units, users, and
security groups. I joined the existing Windows 10 victim endpoint to the
domain, validated domain authentication, and forwarded DC01 Windows event logs
into Splunk for centralized identity investigation.

This phase focused on infrastructure, authentication visibility, and
evidence-safe analysis. Advanced Active Directory attacks were intentionally
left for Phase 11.

### Final Deliverables

| Requirement | Result |
|---|---|
| Domain controller | Windows Server 2022 host `DC01` |
| Lab domain | `majeed.local` |
| Domain services | Active Directory Domain Services and DNS |
| AD organization | `SOC-Lab`, `Users`, and `Workstations` OUs |
| Lab identities | Three domain users and three security groups |
| Domain endpoint | Windows 10 victim joined to `majeed.local` |
| Authentication validation | Successful login as `MAJEED\soc.analyst1` |
| Centralized logging | DC01 event logs forwarded into Splunk |
| Identity events reviewed | 4771, 4776, 4728, 4729, 4624, 4634, and 4672 |
| Analyst documentation | Two SOC-style mini-tickets |

---

# 1. Objective

The goal was to move from standalone endpoint monitoring into an
enterprise-style identity environment.

```text
Build domain services
  -> Create realistic identities
  -> Join a managed endpoint
  -> Centralize domain controller logs
  -> Generate controlled identity events
  -> Investigate the evidence in Splunk
  -> Write defensible analyst conclusions
```

The core question for this phase was:

```text
Can I build the identity infrastructure, confirm where authentication evidence
lives, and investigate that evidence without overclaiming what happened?
```

---

# 2. Lab Architecture

Only the systems required for the Active Directory build were powered on.
Elastic, Wazuh, and Kali remained off to preserve laptop resources.

| System | Role | Lab address |
|---|---|---|
| `DC01` | Windows Server 2022 domain controller and DNS | `192.168.56.10` |
| `DESKTOP-3JKM5O9` | Windows 10 domain-joined victim endpoint | `192.168.56.104` |
| `splunk-soc` | Splunk Enterprise server and forwarder receiver | `192.168.56.102` |

The VirtualBox design used:

- A host-only adapter for isolated lab communication.
- A NAT adapter when internet access was required.
- Static addressing for the domain controller.
- DC01 as the DNS server for the domain-joined Windows client.

This resource-conscious design kept the lab stable while still supporting
domain authentication and centralized logging.

---

# 3. Windows Server and Domain Controller Deployment

## Server Preparation

I created a Windows Server 2022 VM in VirtualBox and prepared it for domain
services:

- Renamed the server to `DC01`.
- Configured host-only and NAT networking.
- Assigned the host-only interface the stable IP `192.168.56.10`.
- Verified the server hostname and network configuration.

## Active Directory Domain Services

I installed Active Directory Domain Services, promoted DC01 to a domain
controller, and created the private lab domain:

```text
majeed.local
```

DNS was installed with the domain controller because domain clients depend on
AD-integrated DNS to locate domain services.

## Validation

I validated the completed promotion with Windows and PowerShell checks,
including:

```powershell
hostname
whoami
Get-ADDomain
```

The results confirmed that:

- The server was operating as `DC01`.
- The Active Directory domain was available.
- The server recognized `majeed.local`.
- Domain services and DNS were functioning.

---

# 4. Active Directory Organization and Identity Objects

## Organizational Units

I created a simple structure that separated users from managed workstations:

```text
majeed.local
└── SOC-Lab
    ├── Users
    └── Workstations
```

This was more realistic than leaving every object in a default container and
created a foundation for later Group Policy and identity investigation work.

## Domain Users

The following controlled lab accounts were created:

| User | Intended role |
|---|---|
| `soc.analyst1` | SOC analyst account used for domain-login validation |
| `helpdesk.user1` | Help desk identity used for group-membership testing |
| `test.employee1` | Standard employee identity used for failed-login testing |

## Security Groups

The following role-based security groups were created:

- `SOC Analysts`
- `Helpdesk`
- `IT Admins`

Users were assigned to appropriate role groups. I also used membership changes
in `IT Admins` to generate controlled privileged-access evidence.

## Validation

Active Directory Users and Computers was used to verify:

- The OU hierarchy.
- User placement.
- Security group membership.
- The domain-joined computer object.

---

# 5. Windows 10 Domain Join and Authentication Validation

The existing Windows 10 victim VM was configured with host-only and NAT
adapters. The host-only adapter DNS setting was pointed to DC01:

```text
192.168.56.10
```

Before joining the domain, I validated communication and name resolution:

```powershell
ping 192.168.56.10
nslookup majeed.local
```

After confirming connectivity, I joined the endpoint to:

```text
majeed.local
```

The client was restarted and domain authentication was validated with:

```text
MAJEED\soc.analyst1
```

The computer object `DESKTOP-3JKM5O9` was then moved into:

```text
SOC-Lab\Workstations
```

This completed the managed endpoint portion of the AD build.

---

# 6. Splunk Universal Forwarder on DC01

## Existing Splunk Receiver

The Splunk server was available at:

```text
192.168.56.102
```

Splunk receiving was already enabled on TCP port:

```text
9997
```

Existing Windows 10 telemetry was checked first to confirm the Splunk server
and receiver were still operational.

## DC01 Forwarder Configuration

I installed and configured Splunk Universal Forwarder on DC01. The forwarder
was directed to the Splunk receiver and configured to collect:

- Windows Security
- Windows System
- Windows Application
- Directory Service
- DNS Server

## Validation

DC01 events became searchable in Splunk with:

```spl
index=* host=DC01
```

The centralized results included relevant Windows Security events such as:

- `4624` - successful logon
- `4634` - logoff
- `4672` - special privileges assigned to a new logon
- `4771` - Kerberos pre-authentication failed
- `4776` - domain controller attempted to validate credentials
- `4728` - member added to a security-enabled global group
- `4729` - member removed from a security-enabled global group

---

# 7. SPL Searches Used

## All DC01 Events

```spl
index=* host=DC01
```

## Kerberos Pre-Authentication Failures

```spl
index=* host=DC01 EventCode=4771
```

## Security Group Membership Changes

```spl
index=* host=DC01 EventCode=4728 OR EventCode=4729
```

## Combined Identity Investigation View

```spl
index=* host=DC01 (EventCode=4771 OR EventCode=4776 OR EventCode=4728 OR EventCode=4729)
| table _time host EventCode Account_Name Account_Domain Group_Name src_ip Workstation_Name
| sort - _time
```

These searches created a repeatable way to move from a broad host search into
specific authentication and privileged-group evidence.

---

# 8. Identity Investigation 01 - Kerberos Pre-Authentication Failure

I generated controlled failed domain authentication attempts for:

```text
MAJEED\test.employee1
```

DC01 recorded Kerberos pre-authentication failures using Event ID `4771`.

## Evidence Reviewed

| Field | Value |
|---|---|
| Account Name | `test.employee1` |
| Service Name | `krbtgt/MAJEED` |
| Client Address | `::ffff:192.168.56.104` |
| Failure Code | `0x18` |

Failure code `0x18` is consistent with incorrect password attempts. The source
mapped to the controlled domain-joined Windows 10 workstation.

## Analyst Interpretation

Event ID `4771` demonstrated that the most useful domain-controller-side
evidence was not necessarily a clean `4625`. The authentication protocol and
where the event is recorded determine which event IDs should be searched.

The activity was suspicious by technique but benign by known lab context.

---

# 9. Identity Investigation 02 - Privileged Group Membership Change

I added `helpdesk.user1` to the `IT Admins` global security group to generate a
controlled privileged-access change.

DC01 recorded Event ID `4728`.

## Evidence Reviewed

| Field | Value |
|---|---|
| Actor | `MAJEED\Administrator` |
| Target member | `MAJEED\helpdesk.user1` / Helpdesk User One |
| Group modified | `MAJEED\IT Admins` |
| Event ID | `4728` |

I then removed the member and confirmed Event ID `4729`.

## Analyst Interpretation

In the lab, both changes were authorized testing. In a production environment,
adding a help desk identity to a privileged IT group would require validation
against an approved request because the change may grant elevated access.

This investigation reinforced the difference between:

```text
Security-relevant activity
```

and:

```text
Confirmed unauthorized activity
```

The event was worth investigating, but the available evidence and known lab
context supported a benign verdict.

---

# 10. Mini-Ticket 01 - Failed Kerberos Authentication

## Alert

**Title:** Failed Kerberos Authentication for `MAJEED\test.employee1`

**Severity:** Low

**Event source:** DC01 Security Log forwarded into Splunk

**Event ID:** `4771` - Kerberos pre-authentication failed

## Evidence

DC01 recorded Kerberos pre-authentication failures for `test.employee1`. The
event identified service `krbtgt/MAJEED`, client address
`::ffff:192.168.56.104`, and failure code `0x18`, which is consistent with an
incorrect password attempt.

## Assessment

The activity matched controlled incorrect-password testing from the
domain-joined Windows 10 workstation. Volume was low and the activity was
expected in the lab.

## Decision

```text
Close as benign lab activity.
```

## Production Follow-Up

In a real SOC, I would:

- Confirm whether the user reported password issues.
- Review a wider time window for repeated failures.
- Determine whether a successful authentication followed.
- Validate the source IP and workstation.
- Check for suspicious post-authentication activity.

---

# 11. Mini-Ticket 02 - Privileged Group Membership Change

## Alert

**Title:** Privileged Group Membership Change - `helpdesk.user1` Added to `IT Admins`

**Severity:** Medium

**Event source:** DC01 Security Log forwarded into Splunk

**Event ID:** `4728` - member added to a security-enabled global group

## Evidence

DC01 recorded `MAJEED\Administrator` adding Helpdesk User One /
`MAJEED\helpdesk.user1` to the `MAJEED\IT Admins` group.

## Assessment

The change affected a privileged-access-related group. It was expected
controlled testing in the lab, but the same event in production would require
authorization and change-ticket validation.

## Decision

```text
Close as benign lab activity.
```

## Production Follow-Up

In a real SOC, I would:

- Validate the change against an approved ticket or change request.
- Confirm the actor was authorized.
- Confirm the target account still required the access.
- Review privileged logons and remote access after the change.
- Search for additional user or group modifications.
- Recommend removal if the access was unauthorized or no longer needed.

---

# 12. Troubleshooting and Lessons Learned

## DC01 Does Not Contain Every Endpoint Event

The domain controller records domain identity and authentication activity.
Endpoint process, Sysmon, file, registry, and local application evidence still
lives on the Windows client unless that endpoint telemetry is also forwarded.

```text
Domain identity evidence -> Domain controller
Endpoint behavior evidence -> Endpoint
Central investigation -> Search both relevant data sources
```

## Failed Password Evidence Was Not a Clean DC-Side 4625

The controlled bad-password attempt did not initially appear as the expected
clean `4625` on DC01. The useful domain-controller evidence appeared as:

- `4771` Kerberos pre-authentication failure
- `4776` credential validation activity

The lesson was to check `4625`, `4771`, and `4776` together and understand
which system and authentication path produced the evidence.

## Forwarder Had a Destination but No Inputs

Splunk Universal Forwarder initially had the correct `outputs.conf`
destination, so DC01 knew where to send data. It did not yet have the required
`inputs.conf`, so it had not been told which logs to collect.

I created the Windows event log input stanzas, restarted the forwarder, and
confirmed DC01 logs were searchable.

```text
Correct destination does not guarantee useful telemetry.
Outputs tell the forwarder where to send.
Inputs tell the forwarder what to collect.
```

## DNS Was Required for the Domain Join

The Windows client needed to use DC01 as its DNS server on the lab adapter.
Basic IP connectivity alone was not enough; AD domain discovery depended on
correct DNS resolution.

## Resource Management Mattered

Only DC01, the Windows 10 client, and Splunk were required. Keeping Elastic,
Wazuh, and Kali powered off reduced resource pressure and made the build more
stable.

---

# 13. Skills Practiced

- Windows Server 2022 deployment in VirtualBox
- Active Directory Domain Services installation
- Domain controller promotion
- AD-integrated DNS dependency awareness
- Static IP planning for domain services
- OU, user, and security group administration
- Role-based group membership
- Windows domain join configuration
- Domain authentication validation
- Splunk Universal Forwarder configuration on a domain controller
- Windows Security, Directory Service, and DNS Server log collection
- SPL searches for identity events
- Kerberos pre-authentication failure interpretation
- Privileged security group change investigation
- Evidence-safe severity and verdict reasoning
- SOC ticket documentation and production follow-up planning

---

# 14. SOC and Interview Mapping

Phase 10 provides practical evidence that I can explain and investigate
enterprise identity fundamentals:

- Built a Windows Active Directory domain from scratch in a home lab.
- Configured DNS and a stable address for the domain controller.
- Created users, groups, OUs, and a managed workstation structure.
- Joined a Windows endpoint to the domain and validated domain login.
- Forwarded domain controller logs into Splunk.
- Investigated Kerberos pre-authentication failures.
- Investigated privileged security group membership changes.
- Distinguished domain-controller identity evidence from endpoint-local
  evidence.
- Wrote evidence-safe conclusions without claiming compromise.

## Interview Summary

> I built an Active Directory lab with a Windows Server domain controller,
> DNS, domain users, groups, OUs, and a domain-joined Windows client. I
> configured the client to use the domain controller for DNS, validated domain
> login, and forwarded DC01 Security logs into Splunk with Universal
> Forwarder. I investigated Kerberos pre-authentication failures and security
> group membership changes using events such as 4771, 4776, 4728, and 4729 to
> identify the account, actor, source, and group affected. The main lesson was
> that domain identity evidence lives on the domain controller, endpoint
> behavior lives on the client, and a SOC analyst has to search the correct
> data source before assigning a verdict.

---

# 15. Scope Boundaries

Phase 10 did **not** claim completion of advanced Active Directory attack
simulation. The following are reserved for later phases:

- Password spraying at attack-simulation scale
- Kerberoasting
- AS-REP roasting
- BloodHound collection and attack-path analysis
- Pass-the-Hash
- DCSync
- Group Policy abuse
- Credential dumping
- EDR investigation of identity-driven endpoint compromise

This phase built and validated the foundation required to perform those labs
safely and investigate their evidence accurately.

---

# 16. Completion Checklist

| Requirement | Status |
|---|---|
| Windows Server 2022 VM deployed | Complete |
| Server renamed to DC01 | Complete |
| Stable host-only IP configured | Complete |
| Active Directory Domain Services installed | Complete |
| DC01 promoted to domain controller | Complete |
| `majeed.local` domain and DNS operational | Complete |
| SOC-Lab, Users, and Workstations OUs created | Complete |
| Domain users created | Complete |
| Security groups created | Complete |
| Windows 10 endpoint joined to domain | Complete |
| Domain authentication validated | Complete |
| Computer object moved to Workstations OU | Complete |
| Splunk Universal Forwarder configured on DC01 | Complete |
| DC01 event logs searchable in Splunk | Complete |
| Kerberos failure evidence investigated | Complete |
| Privileged group change evidence investigated | Complete |
| Two analyst mini-tickets documented | Complete |

---

# 17. Final Phase 10 Outcome

Phase 10 completed the core Active Directory foundation for Track 3. The lab
now has a functioning Windows Server domain controller, private domain DNS,
organized identities and systems, a domain-joined Windows endpoint, and
centralized DC01 event visibility in Splunk.

The most important professional lesson was understanding where identity
evidence lives:

```text
Know the authentication path
  -> Search the correct system
  -> Review the correct event IDs
  -> Add account, source, actor, and change context
  -> Assign a defensible verdict
```

## Up Next - Phase 11

Begin controlled identity attack and detection practice against the AD lab.
The next phase will focus on safely investigating password attacks, suspicious
domain authentication, Kerberos-related activity, and identity attack paths
without overclaiming compromise.
