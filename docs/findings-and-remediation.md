# Findings and remediation

The priorities below are a retrospective portfolio assessment of the submitted lab evidence. They are not formal CVSS scores, a new scan, or proof that the recommended controls were deployed.

## Finding summary

| Finding | Evidence in the report | Impact and priority |
| --- | --- | --- |
| Demonstrated MS17-010 path on Viper | Sections 3.5–3.6; a session opens against the Windows 7 host at `.11` | **Immediate:** a remotely reachable legacy host became an initial foothold. |
| Weak domain and service-account passwords | Sections 4.0 and 6.0–6.1; password-spraying and offline recovery results | **Immediate:** recovered identities enabled subsequent authenticated access. |
| Excessive or broadly usable administrative access | Sections 4.0–5.1; privilege checks and access to additional servers | **Immediate:** account compromise had consequences beyond one endpoint. |
| Sensitive lab artifacts accessible through privileged sessions | Sections 5.1 and 6.1–6.5 | **High:** administrative access enabled file discovery and controlled collection. |
| Legacy service and operating-system exposure | Sections 3.3–3.5 and 7.3; service banners and scanner findings | **High, verify individually:** exposure and scanner output warrant remediation, but each listed issue was not independently exploited. |
| Infrastructure and identity relationships visible after access | Sections 7.1–7.5; DNS, DHCP and BloodHound observations | **Contextual:** useful for understanding impact; visibility alone is not a separate proven vulnerability. |

## Break the demonstrated entry path

Patch systems affected by the relevant SMB vulnerabilities, retire or replace unsupported platforms, and remove SMBv1 where it is not required. Restrict SMB and administrative services to their intended management paths. Validate compatibility before disabling services in a real environment.

**Why:** this addresses the demonstrated Viper entry path and reduces the exposure of similar legacy systems. It does not fix account weaknesses elsewhere in the domain.

**Retest:** verify patch and protocol state, repeat the relevant vulnerability checks in the authorized lab, and confirm that the previous access path no longer succeeds. No such retest is included in this project.

Microsoft’s [MS17-010 bulletin](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010) provides the underlying vulnerability and update context.

## Reduce password and service-account risk

Use long, unique credentials and block common or compromised choices. Review service identities and their privileges; use group Managed Service Accounts where the service and environment support them. Monitor abnormal service-ticket activity and validate supported Kerberos encryption settings before changing them.

**Why:** collecting a service ticket and recovering its password are different events. Strong managed secrets reduce the practical value of ticket material, while least privilege limits the consequence if an identity is recovered.

**Retest:** review SPNs and service-account ownership, assess password-management configuration, confirm unnecessary administrative memberships are removed, and validate that monitoring can recognize a controlled test sequence.

The original report recommends periodic password changes. For the updated portfolio interpretation, NIST’s current [SP 800-63B guidance](https://pages.nist.gov/800-63-4/sp800-63b.html) emphasizes blocking common/compromised passwords and does not call for arbitrary periodic password changes. Rotation remains appropriate when compromise is suspected or confirmed. See also [Microsoft’s Kerberoasting mitigation guidance](https://www.microsoft.com/en-us/security/blog/2024/10/11/microsofts-guidance-to-help-mitigate-kerberoasting/).

## Limit the reach of recovered credentials

Review local and domain administrative memberships, separate privileged administration from routine user activity, restrict remote administration paths, and manage local administrator passwords with Windows LAPS on supported systems.

**Why:** the lab’s privilege checks showed that recovered accounts did not all have the same rights. Removing unnecessary administrative reach can interrupt lateral movement even when one password is compromised.

**Retest:** enumerate effective administration rights and test both an approved management path and a disallowed workstation-to-server path. LAPS manages local administrator passwords; it does not, by itself, correct excessive permissions held by a domain service account. See the [Windows LAPS overview](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview).

## Improve visibility and protect information

Centralize relevant authentication, service-creation, process, and administrative-access logs. Correlate unusual account activity with SMB/RPC traffic and remote execution. Review data permissions and management-plane access to critical infrastructure. Do not assume that the absence of a detection in this report proves a monitoring failure: detection coverage was not measured.

**Why:** the source demonstrates that a privileged identity could discover and retrieve assigned artifacts. The useful defensive question is which account, device, or behavior would make that access unusual, and whether the organization could investigate it.

**Retest:** execute a small authorized validation scenario, confirm telemetry arrives, verify a detection or documented investigation path, and review file-access boundaries. The joint [Detecting and mitigating Active Directory compromises](https://www.cyber.gov.au/business-government/detecting-responding-to-threats/detecting-and-mitigating-active-directory-compromises) guidance provides broader defensive context.

## Interpret secondary observations carefully

- **DNS and DHCP:** records and leases seen through privileged administration are infrastructure context. An information-disclosure finding would need evidence that an unintended identity or source could obtain restricted information.
- **Echo service:** the scanner identified an exposed service. The report does not show a denial-of-service test, so service exposure should not be described as demonstrated disruption.
- **Other scanner findings:** legacy platforms and banners support investigation, but a banner or plugin result is not equivalent to a validated exploit.
- **PsExec:** the relevant issue was credential and privilege misuse, not the administration tool’s mere presence.
- **BloodHound:** graph relationships guide investigation; each consequential path still needs validation.

## Limitations

The repository contains no patch validation, detection test results, formal risk acceptance, complete asset reconciliation, or proof that every potential path was assessed. The report’s unresolved artifact challenges remain unresolved in this case study.
