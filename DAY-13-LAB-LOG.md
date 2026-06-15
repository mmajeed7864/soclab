# Phase 13 Detailed Lab Log - IAM, Hardening, and Least Privilege

## Status

**Completed - June 15, 2026**

**Track 3 - Endpoint and Identity Investigation**

## Completion Summary

Phase 13 focused on defensive identity and access management work inside the
Active Directory home lab. I reviewed domain accounts, privileged access,
service-account settings, Kerberos-related exposure, password and lockout
policy, enabled test accounts, and local administrator membership on the
Windows endpoint.

The review produced 10 findings with risk ratings and remediation guidance. I
fixed the items that could be changed safely without disrupting lab access,
then documented the remaining items for controlled change. This distinction
matters in production: identifying risk does not automatically mean a setting
should be changed without confirming ownership, impact, alternate access, and
rollback.

No production systems or real user data were involved.

### Final Deliverables

| Requirement | Result |
|---|---|
| Active Directory account review | Domain users, privileged indicators, service accounts, and enabled test accounts reviewed |
| Kerberos and service-account review | Non-expiring passwords, pre-authentication settings, SPNs, and visible last-logon context reviewed |
| Domain policy review | Password length and account lockout controls assessed |
| Endpoint privilege review | Windows 10 local Administrators membership reviewed |
| Findings report | 10 findings documented with risk, evidence, recommendation, lab action, and status |
| Safe remediation | Five remediation areas completed and validated |
| Change-control judgment | Access-sensitive and service-dependent items documented instead of changed casually |

---

# 1. Objective

The goal was to practice the hardening workflow expected of a cybersecurity
analyst:

```text
Define the review scope
  -> Collect current identity and policy settings
  -> Identify unnecessary or risky access
  -> Assign risk based on likelihood and impact
  -> Separate safe fixes from disruptive changes
  -> Apply and validate approved lab changes
  -> Document remaining risk and production recommendations
```

This phase was a configuration and access review, not an attack simulation. It
did not attempt to prove that accounts had been abused or that the domain was
compromised.

---

# 2. Review Scope

## Active Directory

- Domain users and enabled status.
- Privileged groups and administrative indicators.
- Built-in Administrator configuration.
- Service accounts and ownership questions.
- `PasswordNeverExpires` settings.
- `DoesNotRequirePreAuth` settings.
- Service principal names.
- Visible `LastLogonDate` context.
- Default domain password policy.
- Account lockout policy.

## Windows Endpoint

- Local Administrators group membership.
- Domain-level groups receiving endpoint administrator access.
- Individually assigned local administrator accounts.
- Least-privilege and Windows LAPS recommendations.

## Evidence Boundaries

The review established configuration exposure and control gaps. It did not
prove password cracking, Kerberos ticket abuse, unauthorized administrator
use, or service-account compromise.

---

# 3. Finding Summary

| ID | Finding | Risk | Lab action |
|---|---|---:|---|
| IAM-01 | Built-in Administrator account enabled | Medium | Documented |
| IAM-02 | Administrator had `PasswordNeverExpires=True` | Medium | Documented |
| IAM-03 | `svc_sql` had `PasswordNeverExpires=True` | Medium | Remediated |
| IAM-04 | `svc_legacy` had `DoesNotRequirePreAuth=True` | High | Remediated |
| IAM-05 | `svc_web` had an HTTP SPN | Medium | Documented as legitimate exposure requiring management |
| IAM-06 | Account lockout threshold was 0 | High | Remediated |
| IAM-07 | Minimum password length was 7 | Medium | Remediated |
| IAM-08 | Enabled service accounts had no visible `LastLogonDate` | Medium | Documented for ownership review |
| IAM-09 | `test.employee1` remained enabled | Low/Medium | Remediated |
| IAM-10 | Broad local administrator membership on Windows 10 | High | Partially remediated |

---

# 4. Detailed Findings

## IAM-01 - Built-in Administrator Account Enabled

**Risk:** Medium

**Evidence:** The built-in Administrator account was enabled in Active
Directory.

**Why it matters:** Built-in administrator accounts are predictable,
high-value identities. Unrestricted or poorly monitored use can reduce
accountability and increase the impact of credential exposure.

**Production recommendation:**

1. Use separate named administrative accounts.
2. Restrict and monitor built-in Administrator usage.
3. Enforce strong authentication and privileged-access controls.
4. Confirm an alternate recovery path before disabling or restricting access.

**Lab action:** Documented only. The account was not disabled because doing so
without validated alternate access could lock administrators out of the lab.

## IAM-02 - Administrator Password Never Expires

**Risk:** Medium

**Evidence:** The Administrator account showed
`PasswordNeverExpires=True`.

**Why it matters:** A static privileged credential can remain useful for a
long time if exposed.

**Production recommendation:** Use privileged access management, controlled
rotation, strong authentication, and monitored named admin accounts.

**Lab action:** Documented only to preserve known recovery access.

## IAM-03 - `svc_sql` Password Never Expires

**Risk:** Medium

**Evidence:** The SQL service account had
`PasswordNeverExpires=True`.

**Production recommendation:**

- Confirm service ownership.
- Rotate the credential through change control.
- Remove the non-expiring setting.
- Consider a group Managed Service Account when supported.

**Lab action:** Remediated. `PasswordNeverExpires` was changed to `False`.

**Validation:** The account was queried again after the change to verify the
updated value.

## IAM-04 - `svc_legacy` Did Not Require Kerberos Pre-Authentication

**Risk:** High

**Evidence:** The legacy service account showed
`DoesNotRequirePreAuth=True`.

**Why it matters:** Disabling Kerberos pre-authentication can create
AS-REP-roastable exposure. This is a configuration risk even when no
credential attack has occurred.

**Production recommendation:** Re-enable pre-authentication unless a
documented legacy requirement exists, then monitor and plan replacement of the
legacy dependency.

**Lab action:** Remediated. `DoesNotRequirePreAuth` was changed to `False`.

**Validation:** The account setting was queried after remediation.

## IAM-05 - `svc_web` Service Principal Name Exposure

**Risk:** Medium

**Evidence:** The web service account had the SPN
`HTTP/webapp.majeed.local`.

**Why it matters:** SPNs are legitimate when services require Kerberos, but
the associated accounts require strong credential management and limited
privilege because service tickets can become part of Kerberoasting
investigations.

**Production recommendation:**

- Confirm the service owner and business purpose.
- Use a strong managed credential or gMSA where possible.
- Keep the account out of privileged groups.
- Monitor service-ticket activity.

**Lab action:** Documented. The SPN was left in place because its presence is
not automatically a misconfiguration.

## IAM-06 - Account Lockout Threshold Disabled

**Risk:** High

**Evidence:** The default domain password policy showed
`LockoutThreshold: 0`.

**Why it matters:** A zero threshold means account lockout is disabled,
reducing resistance to repeated password guessing.

**Lab action:** Remediated:

- Lockout threshold: 5 attempts.
- Lockout duration: 15 minutes.
- Lockout observation/reset window: 15 minutes.

**Validation:** The default domain password policy was reviewed again after
the change.

## IAM-07 - Minimum Password Length Was 7

**Risk:** Medium

**Evidence:** The default domain policy showed
`MinPasswordLength: 7`.

**Production recommendation:** Increase minimum length and encourage long
passphrases. Align the final value with organizational policy and current
identity standards.

**Lab action:** Remediated. The minimum password length was changed to 12.

**Validation:** The domain password policy was re-queried.

## IAM-08 - Service Accounts Without Visible Last Logon

**Risk:** Medium

**Evidence:** `svc_sql`, `svc_legacy`, and `svc_web` were enabled and did not
show a visible `LastLogonDate` in the review output.

**Why it matters:** Missing sign-in context can indicate an unused account, a
data limitation, or a service that authenticates differently. It is not enough
evidence to disable an account by itself.

**Production recommendation:** Confirm owner, dependent service, expected
authentication method, credential age, and recent use before taking action.

**Lab action:** Documented for ownership review.

## IAM-09 - Enabled Test Account

**Risk:** Low/Medium

**Evidence:** `test.employee1` remained enabled after earlier identity labs.

**Why it matters:** Test and stale accounts expand the identity attack surface
and can outlive their intended purpose.

**Lab action:** Remediated. The test account was disabled.

**Validation:** The account's enabled status was checked after the change.

## IAM-10 - Broad Endpoint Local Administrator Membership

**Risk:** High

**Evidence:** The Windows 10 local Administrators group included:

- Local `Administrator`.
- `MAJEED\Domain Admins`.
- `mmajeed`.

**Why it matters:** Broad workstation administrator access increases the
impact of credential theft and weakens least privilege. Domain Admins should
not normally be used for routine workstation administration.

**Lab action:** Partially remediated. `mmajeed` was removed from the local
Administrators group. `MAJEED\Domain Admins` was documented but retained to
avoid disrupting convenient lab administration.

**Production recommendation:**

1. Use separate workstation admin accounts.
2. Remove Domain Admins from routine endpoint administration.
3. Manage membership through approved policy.
4. Deploy Windows LAPS or another controlled local-admin solution.
5. Monitor membership changes.

---

# 5. Safe Remediation Summary

The following changes were completed:

1. Changed `svc_sql` `PasswordNeverExpires` to `False`.
2. Changed `svc_legacy` `DoesNotRequirePreAuth` to `False`.
3. Disabled `test.employee1`.
4. Increased minimum domain password length from 7 to 12.
5. Configured a five-attempt account lockout.
6. Set lockout duration and observation window to 15 minutes.
7. Removed `mmajeed` from the endpoint local Administrators group.

These are seven individual configuration changes across five remediation
areas.

---

# 6. Items Requiring Controlled Change

The following items were intentionally not changed immediately:

| Item | Reason |
|---|---|
| Built-in Administrator enabled | Alternate administrative and recovery access should be validated first |
| Administrator password never expires | Privileged credential rotation requires planning to avoid access loss |
| `svc_web` SPN | SPNs can be legitimate and should be validated against service ownership |
| Service accounts without visible last logon | Ownership and dependent services must be confirmed before disablement |
| Domain Admins in endpoint Administrators | Removal should follow a tested least-privilege administration design |

This was an important lesson from the phase: a hardening report should not
encourage risky production changes without impact analysis, approval,
rollback, and validation.

---

# 7. Analyst Workflow and Reporting

Each finding was documented with:

- Finding ID and title.
- Risk rating.
- Observed configuration.
- Why the setting matters.
- Production remediation.
- Lab action.
- Validation or remaining dependency.

The report uses evidence-safe language:

- It states that exposure or weak configuration was observed.
- It does not claim that a credential was stolen.
- It does not claim that an account was abused.
- It does not claim that the domain was compromised.

---

# 8. Key Lessons Learned

1. Identity hardening requires both technical review and operational judgment.
2. Service accounts should have clear ownership, limited privilege, managed
   credentials, and monitored use.
3. Kerberos-related settings and SPNs are risk indicators, not automatic proof
   of malicious activity.
4. Password and lockout policy changes are easy to describe but still need
   impact testing in production.
5. Stale accounts and broad local administrator access are common,
   high-value review targets.
6. Some risks can be fixed immediately; others require change control and a
   safer replacement design.
7. Validation after remediation is part of the work, not an optional final
   step.

---

# 9. Interview-Ready Explanation

> In Phase 13, I performed an IAM and hardening review of my Active Directory
> lab. I reviewed domain users, privileged groups, service accounts, password
> policy, Kerberos-related service-account exposure, and local administrator
> membership on a Windows endpoint.
>
> I documented 10 findings and fixed the safe items: I corrected two risky
> service-account settings, disabled an unused test account, strengthened
> password and lockout policy, and removed unnecessary local administrator
> access. For access-sensitive or service-dependent findings, I documented the
> risk and the production remediation instead of making an unsafe change
> without ownership review, alternate access, and rollback.

---

# 10. Next Step

Track 3 now moves to Practice Checkpoint 01 and Capstone 02:

- Complete unfamiliar external cases.
- Triage a mixed queue using earlier alert types.
- Document incomplete-evidence decisions.
- Apply asset context to severity.
- Complete the Endpoint and Identity Investigation capstone.

