# Appendix C · Glossary

> Concise definitions of the key terms used throughout the booklet, with the chapter that covers each in depth. Use it to look things up; learn them in context.

---

**AAA** — Authentication, Authorization, Accounting. The trio of proving identity, granting rights, and recording actions. (Ch 12)

**ABAC** — Attribute-Based Access Control. Access decisions from attributes (user, resource, context). Underpins Zero Trust. (Ch 12)

**Active Directory (AD)** — Microsoft's directory service managing users, computers, and permissions across a Windows domain; the heart of most enterprise environments and attacks. (Ch 6, 18)

**AEAD** — Authenticated Encryption with Associated Data (e.g., AES-GCM). Provides confidentiality *and* integrity together. (Ch 11)

**ALE** — Annualized Loss Expectancy = SLE × ARO. The expected yearly cost of a risk; justifies controls. (Ch 30)

**APT** — Advanced Persistent Threat. A well-resourced, patient adversary (often nation-state). (Ch 2)

**ASLR** — Address Space Layout Randomization. Randomizes memory locations to hinder exploitation. (Ch 4)

**ATT&CK (MITRE)** — A knowledge base of real adversary tactics (why) and techniques (how); the industry's shared language. (Ch 13)

**Attack surface** — The sum of points where an attacker could interact with a system. (Ch 14)

**Authentication (AuthN)** — Proving who you are. (Ch 12) · **Authorization (AuthZ)** — Determining what you may do. (Ch 12)

**BloodHound** — Tool that maps Active Directory as a graph to reveal attack paths to Domain Admin. (Ch 18)

**Blue team** — Defenders (monitor, detect, respond). (Ch 2) · **Red team** — Offense (simulate attackers). · **Purple team** — Red + blue collaborating.

**Buffer overflow** — Writing past a buffer's bounds to corrupt adjacent memory (e.g., a return address) → code execution. (Ch 4)

**CIA triad** — Confidentiality, Integrity, Availability. The core properties security protects. (Ch 2)

**CIS Controls** — 18 prioritized, actionable security controls. (Ch 30) · **CIS Benchmarks** — secure-config guides.

**CSPM** — Cloud Security Posture Management. Tools that continuously find cloud misconfigurations. (Ch 26)

**CVE** — a unique ID for a specific known vulnerability. · **CWE** — the class/weakness type. · **CVSS** — a 0–10 severity score. · **KEV** — CISA's catalog of exploited-in-the-wild CVEs. · **EPSS** — probability of exploitation. (Ch 15)

**C2 (Command and Control)** — the channel attackers use to control compromised hosts. (Ch 2, 13)

**DAST** — Dynamic Application Security Testing (tests a running app). (Ch 28) · **SAST** — Static (tests source code). · **SCA** — Software Composition Analysis (scans dependencies).

**DCSync** — Abusing AD replication rights to extract password hashes (including krbtgt) from a DC. (Ch 18)

**Defense in depth** — Layering independent controls so no single failure is fatal. (Ch 10)

**DevSecOps** — Integrating security into the continuous development/deployment pipeline; "shift left." (Ch 28)

**DFIR** — Digital Forensics and Incident Response. (Ch 22, 23)

**Diamond Model** — Frames intrusions via Adversary, Capability, Infrastructure, Victim; enables pivoting. (Ch 13)

**DoS / DDoS** — Denial of Service; making a system unavailable. (Ch 2)

**EDR / XDR** — Endpoint (Extended) Detection and Response; endpoint telemetry + response. (Ch 20)

**Enumeration** — Methodically extracting details from a service/system to find weaknesses. (Ch 15)

**Exploit** — Code/method that takes advantage of a vulnerability. (Ch 10, 17)

**Federation** — Extending identity/trust across organizational boundaries (SSO). (Ch 12)

**GRC** — Governance, Risk, and Compliance. How security is governed, prioritized, and legally met. (Ch 30)

**Hashing** — One-way, keyless fingerprinting of data (integrity). Distinct from encryption. (Ch 11)

**IAM** — Identity and Access Management. The #1 cloud attack surface. (Ch 12, 26)

**IDOR** — Insecure Direct Object Reference; accessing others' objects by changing an ID (broken access control). (Ch 12, 16)

**IOC** — Indicator of Compromise (hash, IP, domain). Cheap for attackers to change. (Ch 13, 25)

**IR (Incident Response)** — The structured process of handling a breach: Prep, Identify, Contain, Eradicate, Recover, Lessons. (Ch 22)

**Kerberoasting** — Requesting a service ticket and cracking it offline to recover a service account's password. (Ch 6, 18)

**Kerberos** — AD's ticket-based authentication protocol (TGT, service tickets). (Ch 6)

**Kill chain** — Lockheed Martin's 7-stage model of an attack; breaking any link stops it. (Ch 2, 13)

**Lateral movement** — Moving from one compromised host to others (e.g., Pass-the-Hash). (Ch 17, 18)

**Least privilege** — Granting the minimum access needed; the most important design principle. (Ch 10)

**LLM security** — Securing large-language-model applications (prompt injection, excessive agency, etc.). (Ch 29)

**LSASS** — Windows process holding credentials in memory; the most-attacked process (credential dumping). (Ch 6, 18)

**MFA** — Multi-Factor Authentication (factors from different categories). FIDO2/passkeys are phishing-resistant. (Ch 12)

**MITRE ATLAS** — ATT&CK-equivalent for adversarial machine learning. (Ch 29)

**NTLM** — Older Windows challenge-response auth; enables Pass-the-Hash. (Ch 6)

**OSINT** — Open-Source Intelligence; recon from public sources. (Ch 14)

**OWASP Top 10** — The standard list of top web application risks (broken access control, injection, etc.). (Ch 16) · **OWASP LLM Top 10** — for AI apps. (Ch 29)

**Pass-the-Hash (PtH)** — Authenticating with a stolen NT hash without cracking it. (Ch 6, 18)

**PKI** — Public Key Infrastructure; CAs, certificates, and trust enabling HTTPS. (Ch 11)

**Prompt injection** — Manipulating an LLM via crafted text (direct or indirect); the #1 LLM risk, unpatchable at the model layer. (Ch 29)

**Privilege escalation** — Turning limited access into higher (vertical) or peer (horizontal) access. (Ch 4, 17)

**Purple team** — See Blue/Red team. Collaboration to turn attacks into detections. (Ch 2, 13)

**RBAC** — Role-Based Access Control; permissions via roles. The enterprise standard. (Ch 12)

**RCE** — Remote Code Execution; running attacker code on a target. (Ch 16, 17)

**Reverse shell** — The target connects back to the attacker's listener (evades inbound firewalls). (Ch 17)

**Risk** — Likelihood × Impact of a threat exploiting a vulnerability. Managed, never eliminated. (Ch 10, 30)

**RTO / RPO** — Recovery Time / Point Objective; how fast to recover / how much data loss tolerable. (Ch 30)

**SBOM** — Software Bill of Materials; inventory of all components. Critical for supply-chain response. (Ch 28)

**Shared responsibility model** — The split of security duties between cloud provider and customer. (Ch 26)

**Shift left** — Moving security earlier in the SDLC (cheaper, safer). (Ch 28)

**SIEM** — Security Information and Event Management; centralizes logs, correlates, alerts. (Ch 20)

**Sigma** — Vendor-neutral detection rule format; converts to any SIEM. (Ch 21)

**SOAR** — Security Orchestration, Automation, and Response; automates SOC workflows. (Ch 8, 20)

**SOC** — Security Operations Center; where analysts monitor and triage. (Ch 2, 20)

**SQL injection (SQLi)** — Injecting SQL via unsanitized input; fixed with parameterized queries. (Ch 16)

**SSRF** — Server-Side Request Forgery; making the server fetch attacker-chosen URLs (e.g., cloud metadata). (Ch 16, 26)

**STRIDE** — Threat-modeling mnemonic: Spoofing, Tampering, Repudiation, Info disclosure, DoS, Elevation of privilege. (Ch 10)

**SUID** — Unix bit making a program run as its owner; a classic privilege-escalation vector. (Ch 5, 17)

**Supply chain (software)** — All the code/tools/infra you depend on but didn't write; a top attack vector. (Ch 28)

**Threat / Threat actor** — A potential cause of harm / the who behind it. (Ch 2, 10)

**Threat hunting** — Proactively searching for undetected attackers ("assume breach"). (Ch 25)

**Threat intelligence (CTI)** — Evidence-based knowledge of adversaries to inform defense. (Ch 25)

**Threat modeling** — Systematically finding what can go wrong before it does. (Ch 10)

**TLS** — Transport Layer Security; encryption + integrity + authentication under HTTPS. (Ch 7, 11)

**TTPs** — Tactics, Techniques, and Procedures; durable behavioral intelligence (top of the Pyramid of Pain). (Ch 13, 25)

**Vulnerability** — A weakness that can be exploited. (Ch 10)

**XSS** — Cross-Site Scripting; injecting JavaScript that runs in others' browsers. Fixed with output encoding. (Ch 16)

**YARA** — Rule format for identifying files/malware by patterns. (Ch 21, 24)

**Zero Trust** — "Never trust, always verify"; no implicit trust from network location. (Ch 7, 10)
