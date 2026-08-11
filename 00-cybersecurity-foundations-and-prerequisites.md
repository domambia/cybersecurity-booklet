# Cybersecurity for CS Graduates — What You Need to Know and Have

**Audience:** Computer science graduate, beginner in security, aiming for employability.
**Purpose:** This is the *prerequisites and knowledge map* document — what you must know, what you must own/set up, and in what order. It is deliberately opinionated so you don't stall on choices.
**Last researched:** August 2026.

---

## 0. Read this first: the three honest truths

1. **Cybersecurity is not an entry-level field — it's a second-layer field.** Almost every security skill is "understanding a system well enough to break or defend it." You cannot secure what you don't understand. Your CS degree covers maybe 40% of the substrate; the other 60% (networking in practice, Windows internals, Active Directory, cloud IAM, log analysis) is usually not taught well in university.
2. **Demand is real, but junior hiring is narrow.** The workforce gap is genuine and information-security analyst employment is projected to grow ~30%+ through 2032, yet many organisations made *zero* entry-level security hires in recent years, and most "junior" postings quietly want 1–2 years of *something* adjacent (helpdesk, sysadmin, NOC, dev, cloud). Plan for a **bridge role or a portfolio strong enough to substitute for one**.
3. **Hands-on evidence beats certificates, but certificates get you past filters.** You need both: a cert for the HR/ATS screen, and a lab + writeups for the technical interview. Neither alone is enough.

---

## 1. Choose your direction early (but loosely)

You do not need to pick a specialism on day one, but knowing the destination changes what you study in months 4–12.

| Track | What you actually do | Good first job title | Fits you if… |
|---|---|---|---|
| **Blue team / SOC & detection** | Monitor, triage alerts, read logs, hunt threats, write detections | SOC Analyst (Tier 1), Security Analyst | You like pattern-finding, systems, being on defence. **Highest volume of beginner openings.** |
| **Offensive / pentest & AppSec** | Break web apps, networks, AD; write findings reports | Junior Pentester, AppSec Engineer | You like puzzles, deep dives, writing exploits. Fewer junior roles, higher bar. |
| **Cloud security** | Secure AWS/Azure/GCP, IAM, config, CI/CD | Cloud Security Engineer (usually needs 1–2 yrs first) | You like infrastructure, automation, IaC. **Top-2 demanded skill in 2026.** |
| **GRC / risk & compliance** | Policy, frameworks, audits, risk registers | GRC Analyst, Compliance Analyst | You write well, like structure. Hires beginners readily, less technical. |
| **Security engineering / DevSecOps** | Build guardrails, pipelines, tooling | Security Engineer | You can already code well. **Strongest natural fit for a CS grad.** |
| **AI/ML security** | Secure LLM apps, red-team models, agentic risk | AI Security Engineer | You have ML exposure. **Top-1 demanded skill in 2026, very few people qualified.** |

**Recommendation for a CS graduate:** aim at **Security Engineering / DevSecOps or Blue Team detection engineering**, with AI security as your differentiator. Your coding ability is a genuine advantage there and is *wasted* in pure GRC. Enter through a SOC/analyst or a junior devsecops role if needed.

---

## 2. Foundations you must know (non-negotiable)

This is the part people skip and then plateau. Treat each as "can I explain it to someone else and demonstrate it in a lab?"

### 2.1 Networking (the single highest-leverage foundation)
- OSI and TCP/IP models — and which layer a given attack lives at.
- IPv4/IPv6 addressing, **subnetting and CIDR by hand**, NAT, VLANs.
- TCP three-way handshake, flags, sequence numbers; UDP vs TCP tradeoffs.
- Core protocols and their weaknesses: **DNS, DHCP, ARP, HTTP/HTTPS, SMB, LDAP/Kerberos, SSH, SMTP, RDP, SNMP**.
- TLS: handshake, certificates, chains of trust, common misconfigurations.
- Routing/switching basics, firewalls, proxies, VPNs, WAF, IDS/IPS, load balancers.
- **Skill test:** capture your own traffic in Wireshark and explain every packet in a DNS lookup + TLS handshake.

### 2.2 Operating systems
**Linux (primary):**
- Filesystem hierarchy, permissions (`rwx`, SUID/SGID, sticky bit), ownership, ACLs.
- Processes, systemd services, cron, users/groups, sudoers.
- Shell fluency: `grep`, `awk`, `sed`, `find`, `xargs`, pipes, redirection, `ss`, `netstat`, `lsof`, `ps`, `journalctl`.
- Logs: `/var/log/auth.log`, `syslog`, auditd.
- Package management, kernel modules, SELinux/AppArmor at a conceptual level.

**Windows (do not skip — most enterprise attacks happen here):**
- Registry, services, scheduled tasks, WMI.
- **Active Directory**: domains, forests, OUs, GPOs, Kerberos authentication, tickets, SPNs, trusts.
- Authentication internals: NTLM vs Kerberos, LSASS, hashes, tokens.
- **Event logs** and the security-relevant Event IDs (4624/4625 logons, 4688 process creation, 4768/4769 Kerberos).
- PowerShell fluency (both as admin tool and attacker tool).

### 2.3 Security core concepts
- **CIA triad**, AAA (authentication, authorization, accounting), non-repudiation.
- **Least privilege, defence in depth, zero trust, fail-secure, separation of duties.**
- **Threat modelling** — STRIDE, attack trees, trust boundaries, abuse cases.
- Risk = likelihood × impact; qualitative vs quantitative risk; residual risk.
- Vulnerability vs threat vs exploit vs risk vs asset (get the vocabulary exact).
- **CVE / CVSS / CWE / EPSS / KEV** — what each is and when to cite it.
- Attack lifecycle models: **Cyber Kill Chain**, **MITRE ATT&CK** (tactics → techniques → procedures), the Diamond Model.
- Security controls: preventive / detective / corrective, administrative / technical / physical.

### 2.4 Cryptography (applied, not academic)
- Symmetric (AES, modes: GCM vs CBC, why ECB is broken) vs asymmetric (RSA, ECC).
- Hashing (SHA-2/3), **password hashing** (bcrypt, scrypt, Argon2) and why they differ from fast hashes.
- HMAC, digital signatures, key exchange (Diffie–Hellman), PKI and certificate lifecycle.
- Randomness: CSPRNG vs PRNG; nonce/IV reuse failures.
- **JWT**, OAuth 2.0 / OIDC, SAML — token structure, validation, and their classic flaws.
- Practical failures: hardcoded keys, weak KDFs, padding oracles, downgrade attacks.
- **Rule:** never invent crypto; know how to use libraries correctly and how to spot misuse in code review.

### 2.5 Web and application security
- HTTP fully: methods, status codes, headers, cookies, `SameSite`, CORS, CSP, caching.
- Sessions, authn vs authz, MFA, SSO.
- **OWASP Top 10** in depth — broken access control, injection (SQLi, command, NoSQL), SSRF, XSS (reflected/stored/DOM), CSRF, insecure deserialization, XXE, IDOR, security misconfiguration.
- API security: REST and GraphQL specifics, rate limiting, mass assignment, BOLA/BFLA.
- Secure SDLC: SAST, DAST, SCA, secret scanning, dependency and **supply-chain** risk (SBOM, typosquatting, CI/CD compromise).
- Client-side: browser security model, same-origin policy, subresource integrity.

### 2.6 Programming and automation (your edge — use it)
- **Python** — the security lingua franca: `requests`, `scapy`, `impacket`, parsing logs, writing tooling, automating triage.
- **Bash + PowerShell** — for real work on the boxes you touch.
- **Go** — increasingly the language of modern security tooling.
- **C basics + memory model** — needed to understand buffer overflows, use-after-free, heap corruption; even if you never write C professionally.
- **Assembly (x86-64) reading ability** — for reverse engineering and exploit understanding.
- **Regex and SQL** — you will use both daily in log/data work.
- **Git**, CI/CD pipelines, Docker, Kubernetes basics, Infrastructure-as-Code (Terraform).

### 2.7 Cloud (mandatory in 2026, pick one provider deep)
- Shared responsibility model.
- **IAM** — the number-one cloud attack surface: roles, policies, trust relationships, privilege escalation paths, federation.
- Networking in cloud: VPCs, security groups, NACLs, private endpoints.
- Storage exposure (public buckets), secrets management (KMS/Key Vault/Secret Manager).
- Logging & monitoring: CloudTrail / Azure Activity Log / GCP Audit Logs → SIEM.
- Container and serverless security; posture management (CSPM) concepts.
- Native security services (e.g. GuardDuty, Security Hub / Microsoft Defender for Cloud, Sentinel).

### 2.8 Defensive operations (blue team)
- **SIEM** — log ingestion, normalisation, correlation, writing queries (Splunk SPL, KQL, or Elastic DSL). Pick one query language and get genuinely good.
- **EDR/XDR** — telemetry, process trees, behavioural detection.
- **Detection engineering** — Sigma rules, YARA, tuning for false positives, detection-as-code.
- **Incident response lifecycle** — preparation, identification, containment, eradication, recovery, lessons learned (NIST SP 800-61).
- **Digital forensics basics** — disk/memory acquisition, chain of custody, artifacts (MFT, prefetch, registry hives, browser history), Volatility, Autopsy.
- **Threat intelligence** — IOCs vs TTPs, the pyramid of pain, feeds, attribution caution.
- Log sources you must be able to read cold: Windows Security log, Sysmon, firewall, proxy, DNS, cloud audit logs, auth logs.

### 2.9 Offensive operations (red team) — at least literacy, even if you go blue
- Methodology: recon → enumeration → exploitation → privilege escalation → persistence → lateral movement → exfiltration → reporting.
- **OSINT** and passive recon.
- Scanning and enumeration: `nmap`, service fingerprinting, directory bruteforcing.
- Web exploitation with **Burp Suite** (learn it properly — this is a core skill).
- Privilege escalation on Linux and Windows (misconfigs, kernel exploits, credentials at rest).
- **Active Directory attacks**: Kerberoasting, AS-REP roasting, pass-the-hash, DCSync, delegation abuse, BloodHound analysis.
- Password attacks: hashcat, John, wordlists, rules — and why they succeed.
- Post-exploitation frameworks conceptually (Metasploit, C2 concepts).
- **Report writing** — the actual deliverable of a pentest. Underrated, heavily judged in interviews.

### 2.10 Governance, risk, compliance and law
- Frameworks: **NIST CSF 2.0**, **NIST SP 800-53**, **ISO/IEC 27001/27002**, **CIS Controls v8**, SOC 2, PCI-DSS.
- Privacy/regulatory: **GDPR**, HIPAA, and your own jurisdiction's data protection act — know which applies to you.
- Policies, standards, procedures, baselines — the difference matters.
- Business continuity / disaster recovery, RTO/RPO.
- Third-party and vendor risk, security questionnaires.
- **Ethics and legality:** written authorisation, scope, rules of engagement, responsible disclosure. *Never test systems you don't own or have explicit written permission to test.* This is the line between a career and a criminal record.

### 2.11 AI and security (your 2026 differentiator)
- **OWASP Top 10 for LLM Applications (2026 edition)** — prompt injection, sensitive information disclosure, hidden context exposure (broadened from system-prompt leakage), unbounded consumption, supply-chain risks for models, insecure output handling, excessive agency.
- **OWASP Top 10 for Agentic Applications (2026)** — autonomous agent risks: tool misuse, goal manipulation, unbounded action loops.
- **MITRE ATLAS** — adversarial ML tactics and techniques.
- Core design principle from the 2026 guidance: *don't try to build a model that can't be fooled — harden the surrounding architecture so a fooled model can't cause damage.* Sandboxing, least-privilege tools, human-in-the-loop, output validation.
- Attacks on ML: data poisoning, model extraction, membership inference, adversarial examples, jailbreaks.
- Using AI de
fensively: alert triage assistance, log summarisation, code review — plus the risk of over-trusting it.
- Securing AI *in your own org*: RAG data leakage, MCP/tool permission boundaries, shadow AI usage.

---

## 3. What you need to have (equipment, accounts, setup)

### 3.1 Hardware
| Item | Minimum | Comfortable | Notes |
|---|---|---|---|
| RAM | 16 GB | **32 GB** | The real bottleneck. 16 GB = 2–3 VMs. 32 GB = a small AD lab. |
| CPU | Modern i5 / Ryzen 5, 4 cores | i7 / Ryzen 7, 8 cores | Virtualization extensions (VT-x/AMD-V) must be enabled. |
| Storage | 512 GB SSD | 1 TB NVMe | VMs and snapshots eat space fast. |
| Extra | — | Old desktop or mini PC as a dedicated lab host, USB Wi-Fi adapter with monitor mode | Optional; a repurposed desktop running Proxmox is the best value upgrade. |

Budget reality: a useful lab costs **$0–$500**. Cloud free tiers can substitute for hardware you don't have.

### 3.2 Software / lab stack
- **Hypervisor:** VirtualBox (free, simplest) or VMware Workstation; **Proxmox VE** if you have a dedicated machine (best snapshotting + virtual networking).
- **Attacker VM:** Kali Linux (or Parrot OS).
- **Target VMs:** Metasploitable 2/3, DVWA, OWASP Juice Shop, VulnHub images, a Windows Server evaluation VM + 1–2 Windows client VMs for an **Active Directory lab**.
- **Blue-team VM:** Security Onion, or an Elastic/Wazuh stack, plus Sysmon on the Windows boxes.
- **Networking:** put everything on a **host-only / internal lab subnet**, isolated from your home network and the internet. Snapshot before every experiment.
- **Core tools to install and actually learn:** Wireshark, nmap, Burp Suite Community, Metasploit, hashcat/John, BloodHound, Volatility, CyberChef, Autopsy, ffuf/gobuster, sqlmap, Nessus Essentials (free tier), Sysinternals suite.
- **Dev environment:** VS Code, Python 3 + venv, Git, Docker.

> ⚠️ **Safety rules:** malware and exploits stay inside isolated VMs with snapshots and no bridged networking. Never point tools at systems you don't own. Keep a separate email/identity for security platforms.

### 3.3 Accounts to create (mostly free)
- **TryHackMe** — best structured start (free tier + ~$14/mo or ~$100/yr premium).
- **Hack The Box / HTB Academy** — for when you're past fundamentals (~$14–20/mo).
- **PortSwigger Web Security Academy** — free, and the best web-security training that exists.
- **OverTheWire (Bandit)** — free Linux/shell wargames.
- **LetsDefend / Blue Team Labs Online / CyberDefenders** — defensive/SOC simulations.
- **picoCTF, CTFtime** — free CTFs, and a calendar of live competitions.
- **GitHub** — your portfolio lives here.
- **AWS / Azure / GCP free tier** — for cloud labs (set billing alerts on day one).
- **LinkedIn** — configured properly; security hiring runs through it heavily.
- **Cloud/community:** OWASP local chapter, a couple of Discord/Slack communities, r/cybersecurity, and 2–3 blogs/newsletters.

### 3.4 Career assets
- A **GitHub portfolio** with 3–5 real projects (see §6).
- A **writeup habit** — a blog or repo of lab/CTF writeups. This is the single most effective differentiator for a beginner.
- A **CV** framed around demonstrated skills, not coursework.
- A tracked **learning log** (what you did, what broke, what you learned).

---

## 4. Certifications: what to get, in what order

| Order | Cert | Cost (approx.) | Time | Why |
|---|---|---|---|---|
| Optional pre-step | **ISC2 Certified in Cybersecurity (CC)** | Exam often **free** via ISC2's One Million Certified program; ~$50/yr upkeep | 2–4 wks | Zero-risk vocabulary validation. |
| Optional pre-step | **Google Cybersecurity Professional Certificate** | under ~$300 | 4–8 wks | Gentle theory + labs; good if you're truly cold-starting. |
| **1 (do this)** | **CompTIA Security+ (SY0-701)** | ~$425 | 2–3 months | Appears in ~70% of entry-level postings; DoD 8140-recognised. **The default first cert.** |
| Networking gap-filler | **Cisco CCNA** or CompTIA Network+ | ~$300–350 | 2–3 months | Only if your networking is weak — but strong networking pays back forever. |
| Cloud gap-filler | **Microsoft SC-900** or **AWS Cloud Practitioner** | ~$100 | 2–4 wks | Cheap, fast, signals cloud literacy. |
| **2 — blue path** | **CompTIA CySA+**, or **TryHackMe SAL1**, or **BTL1** | ~$400 / cheaper alternatives | 2–3 months | Analyst-level detection and IR. Practical alternatives are cheaper and hands-on. |
| **2 — red path** | **INE eJPT** → **HTB CPTS** or **PNPT** | ~$250 / ~$500 | 3–6 months | eJPT is the cheapest proof you enjoy offence; **CPTS/PNPT are credible practical OSCP alternatives.** |
| **3 — red path** | **OSCP / OSCP+** | ~$1,600+ | 6–12 months | Still the gold standard for pentest hiring. Only after CPTS-level competence. |
| **3 — cloud** | **AZ-500** or **AWS Security Specialty** | ~$165–300 | 2–3 months | Where the money moves next. |
| Later (needs 5 yrs) | **CISSP**, **CCSP**, **CISM** | ~$750+ | — | Management/architecture tier. Do not chase these now. |

**Sequencing rule:** one cert at a time, each paired with a lab project that proves it. A cert with no lab behind it fails the technical interview.

---

## 5. A realistic 12-month study plan

Assume **10–15 hours/week**. Adjust, don't compress.

**Months 1–2 — Foundations**
Networking + Linux + Windows basics. Build the lab (hypervisor, Kali, one target). Finish TryHackMe's beginner path. Do OverTheWire Bandit end-to-end. Start Security+ study. *Deliverable:* lab documented in a GitHub README with a network diagram.

**Months 3–4 — Security core + Security+**
Security concepts, cryptography applied, MITRE ATT&CK, risk basics. Pass **Security+**. Begin PortSwigger Web Security Academy. *Deliverable:* Security+ passed; 20+ PortSwigger labs done.

**Months 5–6 — Pick your lane and go deep**
*Blue:* Security Onion or Wazuh + Sysmon, ingest logs, write 10 detections, do LetsDefend/CyberDefenders investigations, learn one query language (KQL or SPL) properly.
*Red:* Burp Suite deep dive, finish PortSwigger, HTB Academy modules, 10+ retired HTB boxes with writeups.
*Deliverable:* 5 published writeups.

**Months 7–8 — Cloud + automation**
One cloud provider deep: IAM, logging, misconfiguration hunting. Terraform a deliberately insecure environment then fix it. Write Python tooling that automates something real in your workflow. *Deliverable:* a cloud security project repo + a Python tool.

**Months 9–10 — Active Directory + AI security**
Build a small AD lab (DC + 2 clients). Attack it, then detect the attacks in your SIEM. Work through the OWASP LLM Top 10 2026 and MITRE ATLAS; build and then break a small LLM app (prompt injection, tool-abuse containment). *Deliverable:* an "attack + detection" paired writeup, and one AI-security writeup.

**Months 11–12 — Specialise, certify, apply**
Second cert (CySA+ / SAL1 / eJPT / CPTS path). Polish CV and LinkedIn around your portfolio. Enter 2–3 CTFs. Apply broadly — including bridge roles (helpdesk, NOC, junior sysadmin, junior dev with security interest) if security roles don't land. Contribute to an open-source security tool. *Deliverable:* applications out, portfolio complete.

---

## 6. Portfolio projects that actually get you hired

Pick 4–5. Document each with a README: goal, architecture, what you did, what you found, what you'd do differently.

1. **Isolated home lab with network diagram** — the baseline expectation.
2. **SIEM + detection project** — ingest Windows/Sysmon/cloud logs, write Sigma rules, show a detected attack chain with screenshots.
3. **Attack-and-detect pair** — perform an AD attack (e.g. Kerberoasting), then show the exact log evidence and the detection rule that catches it. *This single project demonstrates both red and blue thinking and interviews extremely well.*
4. **Web app security assessment** — assess Juice Shop or a deliberately vulnerable app and write a **professional-format pentest report** (executive summary, findings, CVSS, remediation).
5. **Security automation tool** — Python/Go: log parser, IOC enricher, cloud misconfiguration scanner, CI security gate.
6. **Cloud security hardening** — insecure Terraform environment → findings → remediated version, with before/after evidence.
7. **LLM app red-team writeup** — build a small RAG or agentic app, attack it per OWASP LLM Top 10 2026, then implement architectural containment.
8. **Malware analysis (static + dynamic)** — a safe sample in an isolated VM, with a full report. Only if you're careful and interested.

---

## 7. Habits and community

- **Learn in public.** Post writeups. It compounds faster than anything else you can do.
- **Read primary sources**, not just courses: NIST publications, OWASP guides, CIS Controls, MITRE ATT&CK/ATLAS, SANS reading room, vendor threat reports, CVE advisories.
- **Follow current events weekly** — one newsletter, one podcast. Security knowledge decays fast.
- **Do CTFs** — picoCTF to start, then live events via CTFtime. They build the debugging instinct courses don't.
- **Join a community** — OWASP local chapter, DEF CON group, a Discord. Most first security jobs come through people, not job boards.
- **Take notes in a system** (Obsidian/Notion). You will re-learn everything you don't write down.
- **Keep the ethics line bright.** Authorisation in writing, always. Responsible disclosure, always.

---

## 8. Prerequisite self-assessment checklist

Tick these before calling yourself job-ready for a junior role.

**Networking**
- [ ] Subnet by hand without a calculator
- [ ] Explain a TLS handshake packet by packet in Wireshark
- [ ] Explain how DNS resolution can be attacked at each step

**Operating systems**
- [ ] Comfortable in a Linux shell without googling basics
- [ ] Escalate privileges on a misconfigured Linux box and explain why it worked
- [ ] Explain Kerberos authentication in an AD domain
- [ ] Read a Windows Security event log and identify a suspicious logon

**Security core**
- [ ] Threat-model a simple web app (STRIDE, trust boundaries)
- [ ] Map an attack to MITRE ATT&CK tactics and techniques
- [ ] Explain the difference between vulnerability, threat, exploit, and risk precisely

**Crypto**
- [ ] Choose the right primitive for a given task and justify it
- [ ] Spot three crypto misuses in a code review

**Web/AppSec**
- [ ] Find and exploit each of the OWASP Top 10 in a lab
- [ ] Use Burp Suite fluently (proxy, repeater, intruder, scanner)
- [ ] Explain a fix, not just an exploit, for each finding

**Code/automation**
- [ ] Write a Python tool that parses logs and flags anomalies
- [ ] Read C well enough to spot a buffer overflow
- [ ] Use Git, Docker, and one CI pipeline confidently

**Cloud**
- [ ] Audit IAM policies for privilege escalation paths
- [ ] Route cloud audit logs into a SIEM and query them

**Defence**
- [ ] Write a working detection rule and tune its false positives
- [ ] Walk through the incident response lifecycle on a real scenario
- [ ] Triage an alert end-to-end and know when to escalate

**AI security**
- [ ] Explain and demonstrate prompt injection and its architectural mitigations
- [ ] Explain the OWASP LLM Top 10 2026 and MITRE ATLAS at a working level

**Professional**
- [ ] One cert passed
- [ ] 4+ documented portfolio projects
- [ ] 5+ public writeups
- [ ] A professional-format security report you'd hand to a client
- [ ] Can explain your lab and one finding out loud, clearly, in 3 minutes

---

## 9. Common failure modes to avoid

- **Cert collecting with no hands-on work.** Fails the technical interview every time.
- **Tool-driven learning.** Knowing `nmap` flags ≠ understanding scanning. Learn the protocol, then the tool.
- **Skipping Windows and Active Directory** because Linux is more fun. Most enterprise compromise happens in AD.
- **Skipping networking** because it's boring. It's the foundation of everything else.
- **Tutorial hell / path-hopping.** Finish one path before starting another.
- **Only offence.** Defence has far more jobs and teaches you offence anyway.
- **Not writing anything down or publishing anything.** Invisible work doesn't get hired.
- **Waiting to feel "ready" before applying.** You won't. Apply while learning.
- **Testing systems without written authorisation.** Career-ending and illegal.

---

## Sources

- [Cybersecurity Roadmap: A Beginner's Guide 2026 — WsCube Tech](https://www.wscubetech.com/blog/cyber-security-roadmap/)
- [Cyber Security Roadmap for Beginners 2026 — EICTA, IIT Kanpur](https://www.eicta.iitk.ac.in/knowledge-hub/cyber-security/cyber-security-roadmap-for-beginners)
- [Cybersecurity Certification Roadmap 2026 — EC-Council](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/cybersecurity-certification-roadmap/)
- [Cybersecurity Learning Roadmap — Coursera](https://www.coursera.org/resources/cybersecurity-learning-roadmap)
- [Top 10 Cybersecurity Certifications in 2026 — Nucamp](https://www.nucamp.co/blog/top-10-cybersecurity-certifications-in-2026-security-gsec-ceh-pentest-and-more)
- [Cybersecurity Certifications 2026: Costs, Paths & Picks — HackerDNA](https://hackerdna.com/blog/cybersecurity-certifications)
- [Top Entry-Level Cyber Security Certifications — StationX](https://www.stationx.net/entry-level-cyber-security-certifications/)
- [Which Cybersecurity Certification Should Beginners Start With in 2026?](https://thecybersecuritytrail.com/guide/which-cybersecurity-certification-should-beginners-start-with-in-2026/)
- [TryHackMe vs HackTheBox 2026 — HackerDNA](https://hackerdna.com/blog/tryhackme-vs-hackthebox)
- [TryHackMe vs HackTheBox 2026: Honest Comparison — CertCompass](https://certcompass.org/tryhackme-vs-hackthebox/)
- [Build a Cybersecurity Home Lab in 2026 — Nucamp](https://www.nucamp.co/blog/build-a-cybersecurity-home-lab-in-2026-a-beginner-setup-that-actually-works)
- [Cybersecurity Lab Guide: Setup and Best Practices for 2026 — Lars Birkeland](https://larsbirkeland.com/cybersecurity-lab/)
- [How to Build a Cybersecurity Practice Lab in 2026 — Cyber Desserts](https://blog.cyberdesserts.com/cybersecurity-practice-lab-setup/)
- [Top 10 Entry-Level Cybersecurity Jobs in 2026 — Nucamp](https://www.nucamp.co/blog/top-10-entry-level-cybersecurity-jobs-in-2026-roles-pay-signals-and-skills)
- [Cybersecurity Job Market Statistics and Trends 2026 — StationX](https://app.stationx.net/articles/cybersecurity-job-market-statistics)
- [How to Get Into Cybersecurity in 2026 (The Honest Guide) — InfoSec Job Board](https://www.infosecjobboard.com/blog/how-to-get-into-cybersecurity-2026)
- [Cybersecurity Career Paths and Job Market Outlook 2026 — Iron Circle](https://www.ironcircle.com/insights/cybersecurity-career-paths-job-market-outlook-2026/)
- [OWASP GenAI LLM Top 10 2026 — OWASP Gen AI Security Project](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
- [OWASP 2026 LLM Top 10: "The model will be fooled" — Help Net Security](https://www.helpnetsecurity.com/2026/08/06/owasp-2026-llm-top-10-released/)
- [OWASP Top 10 for Agentic Applications 2026 — DeepTeam](https://www.trydeepteam.com/docs/frameworks-owasp-top-10-for-agentic-applications)
- [Cybersecurity blue team jobs in 2026 — Hack The Box](https://www.hackthebox.com/blog/cybersecurity-blue-team-jobs-roles-salaries-skills)
- [SEC450: SOC Analyst Training — SANS Institute](https://www.sans.org/cyber-security-courses/blue-team-fundamentals-security-operations-analysis)
- [Cloud Security Career Path 2026 — Certland](https://certland.net/blog/cloud-security-career-path-ccsp-2026/)
- [AWS vs Azure vs Google Cloud Security Certs 2026 — Whizlabs](https://www.whizlabs.com/blog/aws-vs-azure-vs-google-cloud-security-certs/)
- [15 Best Free Cybersecurity Resources to Learn in 2026 — Hackers Online Club](https://hackersonlineclub.com/free-cybersecurity-resources/)
- [Top 10 Free Resources for Learning Cybersecurity Skills — LetsDefend](https://letsdefend.io/blog/top-10-free-resources-for-learning-cybersecurity-skills)
