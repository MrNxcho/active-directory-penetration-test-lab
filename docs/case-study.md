# Case study

## The problem I investigated

The assignment gave me an internal network range and a Windows domain with deliberately vulnerable systems and hidden artifacts. My task was to discover the environment, identify weaknesses, demonstrate meaningful access paths, and explain what the results meant for security. The report frames this as a fictional white-hat engagement by “One Piece”; the portfolio presentation identifies it as academic lab work.

I wanted the assessment to answer a practical question: **how could an initial weakness become access to other machines and information in the same environment?** That required connecting infrastructure findings to identity and privilege, rather than stopping after the first successful session.

## Environment and scope

The assigned range was `10.16.32.0/24`. Initial discovery recorded four responding addresses: `.10`, `.11`, `.12`, and `.75`. The report identifies Windows 10, Windows 7 SP1, Windows Server 2008 SP1, and Windows Server 2019 systems. Domain services included Active Directory, DNS, DHCP, and a legacy IIS web service.

| Asset in the report | Observed or reported role | Evidence boundary |
| --- | --- | --- |
| Viper | Windows 7 workstation | The successful exploit capture targets `10.16.32.11`. |
| Raptor | Windows 10 engineering workstation | OS inventory associates Windows 10 with `.10`; later sections document access and artifacts. |
| Galactica / BS75 | Windows Server 2019; domain controller, DNS and DHCP | Domain-discovery output identifies the controller at `.75`. |
| Pegasus / BS62 | Windows Server 2008; legacy server with IIS and directory services in the diagram | Initial inventory puts Server 2008 at `.12`, while later narrative calls the server BS62. The source does not fully reconcile these identifiers. |

This is a reconstruction of a historical report, not a fresh scan. Four responding hosts are an observation, not proof that discovery found every asset. The diagram’s shared switch also does not prove that every possible connection was unrestricted.

## 1. Establish the attack surface

I began with host discovery, then inspected services and OS information. SMB, RPC, RDP, Kerberos, LDAP, and HTTP were useful because they helped distinguish workstations from systems providing domain or server functions. HTTP banner inspection identified IIS 7.0 on a legacy server.

**Why this approach:** a list of IP addresses is not yet a useful target model. Service and host context helped me decide where to investigate next. For example, an exposed SMB service on an old Windows host and Kerberos/LDAP on another host point to very different follow-up questions.

**What I learned:** inventory should remain a maintained record throughout the test. Later host aliases in my report are not fully aligned with the initial table, so a future version should record address, hostname, OS, role, confidence, and evidence reference in one place.

## 2. Prioritize and validate a legacy-host finding

Nessus surfaced multiple findings on legacy Windows systems. I selected Viper, the Windows 7 host, for validation of MS17-010/EternalBlue. The terminal capture records a vulnerable-host check, the exploitation sequence, and a Meterpreter session opening against `.11`.

**Why I chose this path:** the scanner result and host context gave me a specific hypothesis to test. Validating that selected path turned a potential weakness into demonstrated access. It also made the consequence of missing maintenance more concrete than a severity label alone.

**What the evidence does not establish:** the session capture does not by itself document every later privilege or every vulnerability found by Nessus. Findings on other systems are not automatically proven exploitable because Viper was successfully accessed. I did not record a patch-and-retest cycle.

## 3. Shift from host access to domain identity

After the initial foothold, I identified the domain controller and investigated domain users. The report documents valid-account discovery through Kerbrute, a successful password-spraying result, requests for service-ticket material, and offline recovery of two account passwords.

**Why this mattered:** a foothold on one workstation is only part of an Active Directory assessment. Domain identities can cross machine boundaries, and a weak service-account password can have consequences beyond the host where I first obtained access.

I then checked the recovered accounts with NetExec rather than assuming equal access everywhere. One account had administrative rights on two servers; another did not have the same rights. That verification shaped which systems I could approach next.

**Technical distinction:** the report uses a filename containing “goldenticket” for saved material. That filename is not evidence of a forged Golden Ticket attack. The documented sequence is service-ticket collection and offline password recovery. Kerberoasting also does not inherently grant elevated access: the recovered account’s actual permissions determine the impact.

## 4. Follow privileges into additional systems

The report documents authenticated access to server systems with a recovered privileged identity, including the domain-controller context and Pegasus. PsExec provided a remote execution path where the account was permitted to administer the target.

**Why I checked privileges first:** knowing a password does not mean an account is an administrator on every endpoint. Access testing prevented me from treating account recovery as universal compromise and helped explain why one path worked while another required a different identity.

**What this finding means:** PsExec is a legitimate administration mechanism. In this project, the relevant issue was that a recovered identity had powerful remote access. The public write-up does not treat the presence of PsExec as a standalone software vulnerability or describe the access as unauthenticated.

## 5. Recover and investigate lab artifacts

The exam included evidence-recovery and decoding tasks. On Pegasus, I found artifacts in another user’s desktop directory. The remote shell was inconvenient for directly examining the files, so I relocated copies into an accessible user directory before retrieving them for analysis.

**Why I adapted the workflow:** the goal was to inspect the assigned artifacts, not remain tied to a particular shell. Directory permissions and the capabilities of the access method determined the practical collection path. The important security implication was that administrative access allowed me to reach information outside the compromised account’s own working directory.

This was controlled recovery of designated lab files. It does not prove internet-scale exfiltration, evasion of a monitored egress control, or undetected movement; the source includes no such monitoring measurements.

I used different approaches for different artifact types:

| Artifact type | Approach documented | Reasoning |
| --- | --- | --- |
| Encoded text | CyberChef and cipher identification; Morse and Bacon-cipher investigation | Test the representation of the data before assuming the file was unreadable. |
| QR code and barcode | Decoding tools | Read the encoded content rather than treat the image as only a visual artifact. |
| Apparently empty text | Select-all and Notepad++ whitespace inspection | A blank-looking file can still contain tabs, spaces, or control characters. |
| Deeply nested archives | A shell loop for repeated extraction | Automate a repetitive task instead of manually repeating thousands of operations. |
| Password-protected Office document | Office hash extraction and password recovery with John/Johnny | Identify the file’s protection mechanism and test the password weakness. |
| Image content | Metadata inspection followed by image steganography tools | Move to another hypothesis when metadata inspection did not reveal useful information. |

Several challenges were only partially solved or unresolved, particularly the Pegasus medium/hard material and a Raptor hard challenge without a documented solution. I retain those limitations instead of presenting the artifact work as a complete solve.

## 6. Connect the access paths to defensive priorities

I inspected DNS and DHCP information and used BloodHound to understand high-value objects and relationships. This added context to the account and host findings: identities, server roles, and exposed services all influenced the consequences of an initial compromise.

**Why this is more useful than a tool list:** the remediation needs to interrupt the chain. Patching closes the demonstrated legacy-host entry point. Stronger service-account secrets make offline recovery less practical. Tighter administrative rights reduce what a recovered account can reach. Segmentation and logging can reduce and expose subsequent movement.

Directory and infrastructure visibility still need careful interpretation. Seeing DNS records or DHCP leases after obtaining privileged access is not, on its own, proof of a separate information-disclosure vulnerability. Likewise, BloodHound relationships suggest paths worth validating; the graph alone does not prove successful exploitation of every path.

## Outcomes and lessons

The report documents a successful Viper session, credential recovery, authenticated administrative access to additional systems, and recovery of multiple assigned artifacts. Those results demonstrate a chain of host and identity weaknesses in this lab. They do not establish full coverage of every system or a completed remediation exercise.

My strongest lesson was to connect **evidence → permission → consequence → corrective control**. The most valuable part of the project was explaining why a finding changed the next decision, and why a defender would need more than one control to interrupt the chain.

For another assessment, I would:

1. Reconcile asset names and addresses as soon as aliases appear.
2. Record the identity and exact privilege level at each successful session.
3. Maintain an evidence register with timestamps, hashes, and sanitized publication copies.
4. Separate attempted techniques, scanner findings, verified access, and unresolved questions.
5. Define retest criteria before remediation and record whether each path is actually closed.

## Source and scope of the reasoning

The chronology and outcomes above are based on [the submitted report](../report/FinaExamDocument.docx). The “why” paragraphs explain the technical purpose of the documented choices in portfolio form. Lessons and next steps are retrospective analysis, not additional tests claimed to have occurred during the exam. See the [evidence index](evidence-index.md) for the original section references.
