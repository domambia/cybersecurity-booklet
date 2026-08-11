# Appendix E · Self-Assessment Rubrics

> **How to use this:** After each part (and before applying for jobs), rate yourself honestly against these rubrics. The goal is not to reach "Expert" everywhere — it's to reach at least **Competent** on the fundamentals and on your chosen track before calling yourself job-ready. Rate by what you can *do unaided*, not what you've *read* (Chapter 1's illusion of knowledge). Where you fall short, the fix is always the same: go back and do more labs.

**Scale:**
- **1 · Novice** — needs the tutorial open; can't do it unaided.
- **2 · Advanced beginner** — can do it with hints/references.
- **3 · Competent** — can do it independently on standard cases. **← job-ready baseline**
- **4 · Proficient** — handles novel cases with good intuition.
- **5 · Expert** — deep, instant, teaches others.

---

## Rubric 1 · Foundations (Part 1) — everyone needs Competent here

| Skill | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Subnet by hand, explain CIDR |  |  |  |  |  |
| Explain a TLS handshake from a Wireshark capture |  |  |  |  |  |
| Operate Linux fluently (no googling basics) |  |  |  |  |  |
| Find & exploit a Linux privesc (SUID/sudo/cron) and explain it |  |  |  |  |  |
| Explain Kerberos auth and Pass-the-Hash |  |  |  |  |  |
| Read a Windows Security log and spot a suspicious logon |  |  |  |  |  |
| Write a Python tool that parses logs / scans ports |  |  |  |  |  |
| Build & isolate a home lab with snapshots |  |  |  |  |  |

**Gate:** ≥3 on all before Part 3. Foundations are load-bearing.

---

## Rubric 2 · Security Core (Part 2) — everyone

| Skill | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Threat-model an app (DFD + STRIDE + attack tree) |  |  |  |  |  |
| Distinguish vulnerability/threat/exploit/risk precisely |  |  |  |  |  |
| Choose the right crypto primitive; spot crypto misuse |  |  |  |  |  |
| Explain correct password storage (salt + Argon2/bcrypt) |  |  |  |  |  |
| Distinguish AuthN vs AuthZ; explain broken access control |  |  |  |  |  |
| Explain MFA hierarchy (why FIDO2 > SMS) |  |  |  |  |  |
| Map an attack to ATT&CK tactics/techniques |  |  |  |  |  |
| Explain the Pyramid of Pain and why TTPs matter |  |  |  |  |  |

**Gate:** ≥3 on all before specializing.

---

## Rubric 3 · Offensive track (Part 3)

| Skill | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Recon & OSINT: map an attack surface |  |  |  |  |  |
| nmap fluently; deep service enumeration |  |  |  |  |  |
| Run & triage a vuln scan; prioritize by real risk |  |  |  |  |  |
| Use Burp fluently; exploit the OWASP Top 10 |  |  |  |  |  |
| Get a shell; escalate on Linux and Windows |  |  |  |  |  |
| Execute an AD attack chain (Kerberoast/PtH/BloodHound) |  |  |  |  |  |
| Write a professional pentest report |  |  |  |  |  |
| State the *fix* for every vulnerability you find |  |  |  |  |  |

**Job-ready (junior pentest):** ≥3 on most, ≥3 on Burp/OWASP and reporting. Have a full report in your portfolio.

---

## Rubric 4 · Defensive track (Part 4)

| Skill | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Know key data sources & what each reveals |  |  |  |  |  |
| Query a SIEM fluently (one language deep) |  |  |  |  |  |
| Write & tune a behavioral detection (Sigma) |  |  |  |  |  |
| Validate detections (Atomic Red Team) |  |  |  |  |  |
| Walk the IR lifecycle; triage an alert end-to-end |  |  |  |  |  |
| Basic forensics: image, recover, timeline, Volatility |  |  |  |  |  |
| Triage a suspicious file (static + dynamic) |  |  |  |  |  |
| Run a hypothesis-driven threat hunt |  |  |  |  |  |

**Job-ready (SOC/detection):** ≥3 on SIEM querying, detection writing, and IR triage. Have an attack-and-detect + an investigation writeup.

---

## Rubric 5 · Modern platforms (Part 5) — pick per track

| Skill | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Explain shared responsibility; audit cloud IAM |  |  |  |  |  |
| Find & fix cloud misconfigurations (CSPM) |  |  |  |  |  |
| Explain container escape risks; scan images |  |  |  |  |  |
| Harden a Kubernetes cluster |  |  |  |  |  |
| Build a CI pipeline with security scanning |  |  |  |  |  |
| Explain the software supply chain & SBOM |  |  |  |  |  |
| Attack & secure an LLM app (prompt injection, agency) |  |  |  |  |  |

**Engineering/cloud/AI track:** ≥3 on your target area. The LLM row is a 2026 differentiator — aim high.

---

## Rubric 6 · GRC & professional (Part 6–7) — everyone

| Skill | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Explain "compliance ≠ security" |  |  |  |  |  |
| Build a risk register; compute ALE |  |  |  |  |  |
| Name the major frameworks and when each applies |  |  |  |  |  |
| Explain your jurisdiction's privacy law & breach rules |  |  |  |  |  |
| Communicate a technical finding to a non-technical audience |  |  |  |  |  |
| Have 4–6 polished portfolio projects public |  |  |  |  |  |
| Explain any project out loud in 3 minutes |  |  |  |  |  |

---

## The overall "job-ready" bar

You're ready to seriously apply for **junior roles in your track** when:
- [ ] **≥3 (Competent)** on all of Rubrics 1 and 2 (foundations + core).
- [ ] **≥3** on the core skills of your chosen track's rubric.
- [ ] **One cert passed** (Security+ or track-relevant) — Chapter 33.
- [ ] **4–6 polished portfolio projects** public — Chapter 32.
- [ ] **The flagship attack-and-detect project** done.
- [ ] You can **explain your work out loud, clearly**, in an interview.
- [ ] You understand the **honest market** and have a **two-track job strategy** — Chapter 34.

> You will **not** feel "fully ready" — nobody does (Chapter 34). If you hit the bars above, you're ready enough. Apply while continuing to learn. Waiting for a "5 everywhere" is a trap that keeps capable people unemployed.

---

## Re-assessment cadence
- **After each Part** — rate the relevant rubric; the low scores are your next labs.
- **Before applying** — the overall bar above.
- **Every 6–12 months on the job** (Chapter 36) — the deliberate-practice audit: where am I coasting? Push one skill toward Proficient/Expert.
