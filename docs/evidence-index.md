# Evidence index

The primary source is [FinaExamDocument.docx](../report/FinaExamDocument.docx), Tyree Payne’s original final-exam report. Section numbers below refer to its written headings rather than page numbers, because Word pagination can vary between viewers.

## Selected screenshots

| Published image | Original embedded image | What it supports | What it does not prove |
| --- | --- | --- | --- |
| [Host discovery](../assets/host-discovery.png) | `word/media/image2.png` | Four responding addresses observed in the assigned lab subnet | Exhaustive host coverage |
| [MS17-010 scanner finding](../assets/ms17-010-finding.png) | `word/media/image14.png` | A Nessus plugin result used during prioritization | Successful exploitation of every scanned host |
| [Viper session excerpt](../assets/viper-session.png) | `word/media/image18.png`, cropped to the result lines | A session opened against `.11` in the documented test | Complete post-exploitation privilege or remediation validation |
| [Domain-controller discovery](../assets/domain-controller.png) | `word/media/image29.png` | Reported domain/controller identification at `.75` | Arbitrary access to every domain resource |

The images were extracted from the supplied DOCX. The session image is a labeled crop for readability, not a new execution. The public narrative does not repeat password strings, credential hashes, or challenge answers; the preserved original submission contains the original lab evidence.

## Claim-to-source map

| Topic | Original sections | Assessment |
| --- | --- | --- |
| Educational setting and fictional engagement scenario | Title; 1.0–1.2 | Source context |
| Assigned scope and initial host discovery | 1.1; 3.1 | Documented inventory |
| Service and OS enumeration | 3.2–3.4 | Reported results and screenshots |
| Scanner prioritization and Viper access | 3.5–3.6 | Scanner findings plus documented successful session |
| Domain discovery and service-account investigation | 3.9; 4.0 | Documented discovery, password spraying and ticket/password recovery |
| Effective privilege checks | 4.0; 4.1 | Differing administrative rights reported for recovered identities |
| Authenticated remote administration | 4.1; 5.0–5.1 | Reported access and collection workflow |
| Artifact decoding and Office password recovery | 3.7–3.9; 4.1; 5.2–5.4; 6.1–6.5 | Mixed outcomes; some challenges unresolved |
| DNS, DHCP, IIS and identity relationships | 7.1–7.5 | Supporting infrastructure context |
| Original recommendations | 2.1; 9.0 | Recommendations, not implementation evidence |

## Editorial distinctions retained

1. The artifact is an academic exam report, even though its narrative uses client-engagement language.
2. The initial discovery table and later BS62 naming are not fully reconciled.
3. A filename containing “goldenticket” is not proof of Golden Ticket forgery.
4. A legitimate remote-administration tool is not itself the vulnerability demonstrated by credentialed access.
5. Controlled lab-file recovery does not prove unobserved or unrestricted exfiltration.
6. Statements about stealth, segmentation, or full domain compromise are bounded by the available evidence; the case study does not claim a complete independent audit of those properties.
7. Retrospective recommendations and proposed retests are distinguished from work actually documented in the exam.
