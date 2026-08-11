# Appendix D · 52-Week Study Schedule

> **How to use this:** This maps the booklet's 36 chapters, labs, certs, and portfolio work onto one year at the **standard pace (~12–15 hrs/week)**. It's a scaffold, not a straitjacket — adjust to your life and track (Chapter 1). The non-negotiables every week: **daily Anki (10–15 min)**, **a weekend lab block**, and **one polished writeup**. Intensive learners compress this to ~5–6 months; relaxed learners stretch to ~18–24. Do not skip the labs to "keep up" — the labs *are* the learning.

**Legend:** 📖 read · 🧪 lab · ✍️ writeup/portfolio · 🎓 cert milestone · 🎯 checkpoint

---

## Quarter 1 — Orientation & Foundations (Weeks 1–13)

| Wk | Focus | Key activities |
|---|---|---|
| 1 | **Ch 1** How to use this + **setup** | 🧪 Obsidian + Anki + safety contract + weekly plan. Read Oakley/*Learning How to Learn*. |
| 2 | **Ch 2** What security is | 📖 Map a real breach to the kill chain. Browse ATT&CK. Start *The Cuckoo's Egg*. |
| 3 | **Ch 3** Ethics & law | 📖 Learn your jurisdiction's law. ✍️ Write a rules-of-engagement doc. **Begin Security+ study (Prof Messer).** |
| 4 | **Ch 4** Computing fundamentals | 🧪 /proc/maps, buffer overflow crash, CyberChef decoding. |
| 5–6 | **Ch 5** Linux for security | 🧪 **OverTheWire Bandit 0–20**, log one-liners, GTFOBins privesc. ✍️ Persistence-hunt writeup. |
| 7 | **Ch 9** Build home lab (Stage 1) | 🧪 VirtualBox + Kali + Metasploitable, host-only, verify isolation, snapshots. ✍️ Lab README + diagram. |
| 8–9 | **Ch 6** Windows & Active Directory | 🧪 Event Viewer, install Sysmon, build mini-AD (Stage 3), TryHackMe AD basics. ✍️ Kerberos diagram from memory. |
| 10–11 | **Ch 7** Networking deep dive | 🧪 Subnet by hand, Wireshark TCP+TLS handshakes, TryHackMe networking. 🎓 **Sit Security+.** |
| 12 | **Ch 8** Scripting & automation | 🧪 Build your own port scanner + log parser + API enrichment tool. ✍️ Publish the scanner. |
| 13 | **Q1 review + catch-up** | 🎯 **Checkpoint:** Security+ passed, lab running, Bandit done, first tools/writeups public. Rubric self-check (App E). |

---

## Quarter 2 — Security Core & Offense begins (Weeks 14–26)

| Wk | Focus | Key activities |
|---|---|---|
| 14 | **Ch 10** Principles & threat modeling | 🧪 Full threat model (DFD + STRIDE + attack tree) of an app. ✍️ Portfolio piece. |
| 15 | **Ch 11** Applied cryptography | 🧪 ECB penguin, salt/bcrypt, crack weak hashes, inspect a cert, tamper a JWT. Cryptopals Set 1. |
| 16 | **Ch 12** Identity, AuthN/AuthZ | 🧪 Exploit IDOR, cookie flags, map an OAuth flow. Adopt password manager + hardware MFA. |
| 17 | **Ch 13** Adversary models & ATT&CK | 🧪 Map a breach to ATT&CK, build a Navigator heatmap. 🎯 **Choose your track.** |
| 18 | **Ch 14** Recon & OSINT | 🧪 OSINT yourself, CT-log subdomain hunt, Shodan. |
| 19 | **Ch 15** Scanning & enumeration | 🧪 Full nmap sweep + deep enum, run & triage a Nessus scan, CVE→exploit mapping. |
| 20–22 | **Ch 16** Web application hacking | 🧪 **Master Burp**, full OWASP Top 10 in DVWA, **40+ PortSwigger labs**, Juice Shop, API testing. ✍️ Web assessment writeup. |
| 23 | **Ch 17** Host & network exploitation | 🧪 First exploit + shell, Linux & Windows privesc, crack hashes. ✍️ Boot-to-root walkthrough. |
| 24–25 | **Ch 18** Active Directory attacks | 🧪 BloodHound, Kerberoast, PtH, DCSync/Golden Ticket — **all attack-and-detect** in your lab. ✍️ **Flagship attack-and-detect writeup.** |
| 26 | **Ch 19** Pentest & reporting + review | ✍️ Full mini pentest report. 🎓 **(Offensive track: sit eJPT.)** 🎯 Checkpoint + rubric. |

---

## Quarter 3 — Defense & Modern Platforms (Weeks 27–39)

| Wk | Focus | Key activities |
|---|---|---|
| 27 | **Ch 9** Lab Stage 4 + **Ch 20** SIEM | 🧪 Stand up Wazuh/ELK/Sentinel, ship logs, learn one query language, detect your own attack. |
| 28–29 | **Ch 21** Detection engineering | 🧪 Write behavioral detections, Sigma rules, tune false positives, validate with Atomic Red Team. ✍️ Detection portfolio + a SigmaHQ PR. |
| 30 | **Ch 22** Incident response | 🧪 LetsDefend/CyberDefenders investigation, write a playbook, full attack-to-response in lab. ✍️ Post-incident report. |
| 31 | **Ch 23** Digital forensics | 🧪 Image & verify, recover deleted files, Windows artifacts, Volatility memory analysis. |
| 32 | **Ch 24** Malware analysis | 🧪 REMnux/FLARE-VM setup, static triage, dynamic analysis, write YARA from findings. |
| 33 | **Ch 25** Threat intel & hunting | 🧪 Consume a real threat report, run a hypothesis-driven hunt in your lab. ✍️ Hunt writeup. |
| 34–35 | **Ch 26** Cloud security | 🧪 Free-tier setup (billing alert!), IAM least privilege, CSPM scan, **flaws.cloud/CloudGoat**, cloud detection. 🎓 (Track cert: CySA+/SAL1/BTL1 or AZ-500/AWS.) |
| 36 | **Ch 27** Containers & Kubernetes | 🧪 Image scanning, container escape (lab), Kubernetes Goat, harden a cluster with Falco. |
| 37 | **Ch 28** DevSecOps & supply chain | 🧪 Build a secure CI pipeline (SAST/SCA/secrets/IaC), SBOM, sign an image, policy-as-code. ✍️ Pipeline portfolio piece. |
| 38 | **Ch 29** AI & LLM security | 🧪 **Build → break → secure an LLM app**, OWASP LLM Top 10 walkthrough, Gandalf, ATLAS. ✍️ **AI-security differentiator writeup.** |
| 39 | **Q3 review + catch-up** | 🎯 Checkpoint: track cert progress, defensive + platform projects done. Rubric self-check. |

---

## Quarter 4 — GRC, Portfolio & Job Hunt (Weeks 40–52)

| Wk | Focus | Key activities |
|---|---|---|
| 40 | **Ch 30** GRC & risk | 🧪 Build a risk register, framework-map the booklet, quantitative risk (ALE), CIS self-assessment. |
| 41 | **Ch 31** Privacy & regulation | 🧪 Learn your jurisdiction's law, privacy threat model, breach-notification tabletop. |
| 42–43 | **Ch 32** Portfolio | ✍️ Polish 4–6 flagship projects, set up GitHub + blog, publish, make an open-source contribution. |
| 44 | **Ch 33** Certification strategy | 🎓 Finalize cert plan; schedule/complete your track practical cert. |
| 45–46 | **Ch 34** Job hunt | ✍️ Two-track target list, tailor résumé, optimize LinkedIn, network (chapter/BSides + online), start applying. |
| 47 | **Ch 35** First 90 days | 🧪 Draft a 90-day plan, question-asking + concern-raising scripts, on-the-job learning system. |
| 48 | **Ch 36** Competent → master | ✍️ 5-year vision, permanent learning system, deliberate-practice audit. |
| 49–52 | **Apply, interview, iterate** | 🎯 Run the job-hunt pipeline: applications, interviews (use the self-checks as prep), iterate on feedback. Keep building & learning in public. |

---

## The three eternal weekly habits (never skip)
1. **Daily Anki** — 10–15 min. The compounding foundation.
2. **Weekend lab block** — 3–4 hrs of hands-on. Where skill is actually built.
3. **One polished writeup/week** — your portfolio, built in real time.

> **When you fall behind:** protect the *labs*, cut reading depth if you must. A week with one lab done well beats a week of reading and no doing. And **start applying around Week 45 even if you don't feel ready** — you won't (Chapter 34). Certs + a couple of strong projects + a practical cert in progress is enough to begin.
