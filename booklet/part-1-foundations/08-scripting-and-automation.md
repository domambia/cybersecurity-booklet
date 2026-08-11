# Chapter 8 · Scripting and Automation

> **Why this chapter matters:** This is where your CS degree becomes a superpower most security professionals don't have. Plenty of analysts can *run* tools; far fewer can *build* them, automate the boring 80% of their job, parse a messy log at 2am, or write a custom exploit when the off-the-shelf one doesn't fit. Coding ability is the differentiator that pushes you toward the higher-paid engineering, detection, and AppSec lanes. Lean into it.

> **By the end of this chapter you will:** use Python for security tasks (networking, parsing, automation, API work), write effective Bash and PowerShell for the systems you operate, and understand where automation belongs in a security workflow.

---

## 8.1 The right language for the job

You don't need to master every language. You need the right few, used well.

| Language | Use it for | Why |
|---|---|---|
| **Python** | Tooling, automation, parsing, exploits, API work, data | The security lingua franca. Huge library ecosystem. Learn this deeply. |
| **Bash** | Gluing Linux commands, quick automation on servers | It's already there on every Linux box; irreplaceable for ops. |
| **PowerShell** | Windows automation, AD, and (as attacker/defender) post-exploitation | The language of Windows administration *and* Windows attacks. |
| **Go** | Fast, portable security tools and network services | Increasingly the language modern tooling is written in. Learn after Python. |
| **SQL** | Querying data and understanding injection | You'll read/write it constantly; it's also an attack surface. |
| **JavaScript** | Understanding web attacks (XSS), browser behavior | Read it fluently even if you don't write much. |

As a CS grad you already know how to program. This chapter is about applying that to security *idioms* — not learning to code from scratch.

---

## 8.2 Python for security

Python dominates because it's readable, batteries-included, and has a security library for everything. Core patterns you'll use constantly:

**Networking — talk to services directly:**
```python
import socket
# A minimal port scanner — understand every line rather than run someone else's
def scan(host, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    result = s.connect_ex((host, port))   # 0 == open (completed handshake)
    s.close()
    return result == 0

for port in [22, 80, 443, 445, 3389]:
    print(port, "OPEN" if scan("127.0.0.1", port) else "closed")
```
This connects to the TCP handshake you learned in Chapter 7. Building your *own* scanner — even a crude one — teaches more than a thousand `nmap` runs, because now you understand what nmap is doing.

**HTTP and APIs — the `requests` library:**
```python
import requests
r = requests.get("https://api.example.com/users", headers={"Authorization": "Bearer <token>"})
print(r.status_code, r.json())
```
You'll automate web testing, interact with security tool APIs (VirusTotal, Shodan, your SIEM), and script authenticated requests. Web hacking (Chapter 16) leans heavily on scripting HTTP.

**Parsing — turning chaos into answers:**
```python
import re
# Extract all IPs from a log file and count them
from collections import Counter
ip_re = re.compile(r'\b(?:\d{1,3}\.){3}\d{1,3}\b')
counts = Counter()
with open("access.log") as f:
    for line in f:
        counts.update(ip_re.findall(line))
for ip, n in counts.most_common(10):
    print(n, ip)
```
Log parsing, extracting IOCs, transforming data between formats — this is daily SOC and DFIR work, and Python makes it trivial. **Regular expressions** are non-negotiable here; get genuinely good at them (regex101.com is your practice ground).

**Security-specific libraries to know they exist:**
- **`scapy`** — craft and dissect packets programmatically (custom scanners, spoofing in a lab, protocol analysis).
- **`impacket`** — the essential library/toolkit for Windows/AD protocols (SMB, Kerberos); powers many AD attacks in Chapter 18.
- **`pwntools`** — exploit development framework (binary exploitation, Chapter 4's world weaponized).
- **`beautifulsoup`/`lxml`** — parse HTML for recon/scraping.
- **`cryptography`** — do crypto *correctly* (Chapter 11) instead of rolling your own.

> **Habit to build:** when you find yourself doing a repetitive task by hand (checking a list of hosts, decoding a batch of strings, reformatting a report), stop and script it. Every script you write is a portfolio piece and a permanent time-saver. This instinct is exactly what makes a *security engineer*.

---

## 8.3 Bash for operations

Bash is the glue of Linux. You won't write large programs in it, but you'll write countless small automations directly where the work happens.

```bash
#!/usr/bin/env bash
# Quick host-liveness sweep across a /24 (concept demo — nmap is better in practice)
for i in $(seq 1 254); do
  ip="192.168.1.$i"
  ping -c1 -W1 "$ip" &>/dev/null && echo "$ip is up"
done
```

Key Bash skills: variables and quoting (the source of most bugs), loops, conditionals, exit codes (`$?`), command substitution `$(...)`, and — above all — **pipelines** combining `grep`/`awk`/`sed`/`sort`/`uniq` (Chapter 5). A one-line pipeline often replaces a 30-line script. Know when Bash is enough and when to reach for Python (rule of thumb: if you need real data structures, error handling, or it exceeds ~30 lines, switch to Python).

---

## 8.4 PowerShell for Windows

PowerShell is to Windows what Bash is to Linux — but more powerful, because it's *object-oriented* (commands pass structured objects, not just text). It's the primary tool for both Windows administration/defense and Windows attacks, which is why defenders monitor it so closely.

```powershell
# Find recent failed logons (Event ID 4625) — defensive triage
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} -MaxEvents 20 |
  Select-Object TimeCreated, @{N='User';E={$_.Properties[5].Value}}
```

```powershell
# Enumerate local admins — useful to attacker and defender alike
Get-LocalGroupMember -Group "Administrators"
```

Key concepts: cmdlets (`Verb-Noun` naming), the pipeline passes *objects* so you filter with `Where-Object` and shape with `Select-Object`, and you can reach deep into Windows/AD (`Get-ADUser`, `Get-WinEvent`). Because attackers love PowerShell, defenders enable **script block logging** and **transcription** — meaning as a red-teamer you must know it's watched, and as a blue-teamer you must know how to read those logs. Both sides, again.

---

## 8.5 Automation in the security workflow

Automation isn't just convenience — at scale it's the only way defense keeps up. Where it belongs:

- **SOC/detection:** auto-enrich alerts (look up an IP's reputation, pull the user's recent activity), auto-triage, and orchestrate response (**SOAR** — Security Orchestration, Automation, and Response). You'll meet this in Part 4.
- **Offense:** chain recon → scan → parse into a repeatable pipeline; write custom exploit/PoC scripts.
- **Engineering/DevSecOps:** security checks in CI/CD (Chapter 28) — every code push automatically scanned for vulns, secrets, and misconfigs.
- **Reporting:** generate findings tables and evidence from raw tool output.

**Principles for security automation:**
- **Idempotent and safe** — running it twice shouldn't cause harm; think hard before automating anything destructive.
- **Logged and auditable** — you must be able to reconstruct what your automation did.
- **Fails safe** — on error, stop; don't blindly continue (especially anything that acts on production).
- **Version-controlled** — your scripts live in Git, like all code. This also builds your portfolio.

> **A note on AI-assisted coding (2026):** AI assistants write security scripts quickly, but **verify every line** — they hallucinate flags, invent library functions, and produce subtly wrong regex. In security, a wrong script is a missed detection or a broken production system. Use AI to draft; use your CS judgment to verify. And never paste secrets, client data, or sensitive findings into a third-party AI service — that's a confidentiality breach (Chapter 3).

---

## Common mistakes

- **Collecting scripts you don't understand.** A copy-pasted tool you can't explain is a liability. Build small things yourself first.
- **Over-engineering.** A 10-line script that works beats a "framework" you never finish. Solve the actual problem.
- **Ignoring error handling in anything that touches real systems.** A scanner that crashes silently gives false confidence; an automation that half-runs on production is dangerous.
- **Not version-controlling your work.** Every script is a portfolio artifact — commit it.
- **Weak regex skills.** So much of security work is pattern-matching text. Practice regex deliberately.

---

## Labs

> **Lab 8.1 — Build your own port scanner.** Write the Python TCP scanner from §8.2, then extend it: accept a host and port range as arguments, add threading for speed, and print a clean summary. Compare its results to `nmap` against a lab target. Write up what nmap does that yours doesn't — and why. This is a strong portfolio piece.

> **Lab 8.2 — Log parser.** Take a real-ish log (generate web/SSH logs on your VM, or download a sample). Write a Python script that extracts the top talkers, flags failed logins, and outputs a summary table. This is genuine SOC work; keep it in your portfolio repo.

> **Lab 8.3 — API automation.** Get a free Shodan or VirusTotal API key. Write a script that takes an IP or file hash and returns a reputation summary. You've just built an alert-enrichment tool — exactly what SOAR does.

> **Lab 8.4 — Bash sweep + PowerShell triage.** Write a Bash script that sweeps your lab subnet for live hosts and open common ports. Then write a PowerShell script that pulls the last 20 failed logons and formats them. Two systems, two languages, one skill.

> **Lab 8.5 — Regex drills.** On regex101.com, complete exercises until you can confidently write patterns for: IPs, email addresses, URLs, base64 blobs, and timestamps. Save your patterns to your cheat-sheet notes.

---

## References and further reading

- **TJ O'Connor — *Violent Python*.** The classic "Python for security" book — scanners, forensics, network attacks, all in Python. Dated in specifics, excellent in approach. Reimplement its examples in modern Python 3.
- **Justin Seitz — *Black Hat Python* (2nd ed.).** Offensive tooling in Python: network tools, trojans, extending Burp. Great after *Violent Python*.
- **Automate the Boring Stuff with Python — Al Sweigart (free at automatetheboringstuff.com).** The best free ramp for practical, task-oriented Python.
- **PowerShell:** Microsoft Learn's PowerShell docs, and "Under the Wire" (powershell wargames, like Bandit for PowerShell).
- **regex101.com** — interactive regex builder/tester. Live in it until regex is second nature.
- **Impacket, Scapy, pwntools** — read their official docs/examples; you'll return to each in later chapters.
- **The Go tour (go.dev/tour)** — when you're ready to add Go for tool-building.

---

## Self-check

1. Your Python port scanner uses `connect_ex`. What does a return value of `0` mean, and which networking concept from Chapter 7 does that correspond to?
2. When should you reach for Python instead of Bash?
3. Why do defenders monitor PowerShell so heavily, and what two logging features enable it?
4. What does it mean for an automation to "fail safe," and why does it matter most on production systems?
5. Name three Python libraries a security practitioner uses and what each is for.

<details>
<summary>Answers</summary>

1. `0` means the TCP connection **succeeded** — the three-way handshake completed — so the port is **open**. It corresponds directly to the SYN → SYN-ACK → ACK handshake from Chapter 7.
2. When you need real data structures, robust error handling, external libraries/APIs, cross-platform behavior, or the script grows beyond ~30 lines of glue. Bash is best for short pipelines gluing existing Linux commands.
3. Because PowerShell is the primary tool for Windows post-exploitation (attackers use it for recon, lateral movement, persistence, and fileless attacks). **Script block logging** and **transcription** (plus module logging) record what PowerShell actually executed, so defenders can detect and investigate malicious use.
4. On any error or unexpected state, the automation **stops rather than continuing blindly**. It matters most on production because a partially-run or error-ignoring script can corrupt data, take systems down, or cause outages — the failure mode must be "halt," not "keep going."
5. Any three: `requests` (HTTP/APIs), `scapy` (packet crafting/analysis), `impacket` (Windows/AD protocols), `pwntools` (exploit dev), `cryptography` (correct crypto), `beautifulsoup` (HTML parsing).

</details>

---

## What's next

You can operate systems, read networks, and build tools. Now you need somewhere to *use* all of it safely and legally. [Chapter 9](09-building-your-home-lab.md) builds your practice environment — the isolated lab where everything in Parts 2–5 happens. It's the last foundation chapter, and the gateway to the hands-on core of the booklet.
