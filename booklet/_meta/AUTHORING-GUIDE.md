# Authoring Guide (internal)

Every chapter of this booklet is written to this specification. Read it fully before writing.

## Audience

A computer science graduate, beginner in security, self-studying toward employment. They can program (Python/Java/C-ish), know basic data structures, have seen a shell, and have vaguely heard of TCP. They have **not** administered a Windows domain, read a packet capture, tuned a SIEM, or written a pentest report.

Write for a smart reader who lacks *this specific* knowledge. Never condescend; never assume.

## Voice and style

- Second person ("you"), present tense, direct. American English.
- **No filler openings.** Never "In today's interconnected digital landscape…". Start with the substance.
- No hype, no marketing tone, no exclamation marks.
- Short paragraphs. Aggressive use of tables, lists, and headings for scannability.
- Use they/them when referring to a person whose pronouns aren't stated.
- Explain *why*, not just *what*. A reader should finish able to reason about novel cases, not just recall facts.
- When something is genuinely contested or vendor-dependent, say so rather than picking a fake consensus.

## Length

- Standard chapter: **3,000–6,000 words**. Depth is the point of this project — do not write a summary. But do not pad: every paragraph must carry information.
- Appendices: as long as needed to be genuinely useful (the glossary and cheat sheets will be long).

## Required chapter structure

```markdown
# Chapter N · <Title>

> **Part X · Chapter N of 36**
> **Prerequisites:** <chapters, with relative links>
> **Time:** <realistic study estimate, e.g. "8–12 hours including labs">

## Why this chapter matters
2–4 sentences. Concrete stake: what breaks in the real world if you don't know this.

## What you'll be able to do
- 4–7 bullet outcomes, each observable/testable ("Explain X", "Build Y", "Detect Z").

---

## <Numbered body sections: N.1, N.2, …>

## Common mistakes
Table or list: mistake → why it happens → what to do instead. 5–8 entries.

## Labs
2–4 labs. Each with: **Objective**, **Setup**, **Steps** (numbered, concrete), **Success criteria** (> ✅), **Stretch goal**.

## Book and resource references
Annotated. See rules below.

## Self-check
6–10 questions, then a collapsed/clearly-separated **Answers** subsection.

## What's next
1–3 sentences pointing to the next chapter with a relative link.
```

## Content rules

**Examples are mandatory.** Every significant concept gets at least one of:
- a real command with realistic output,
- a real code snippet (correct, copy-pasteable, language-tagged),
- a worked walkthrough of a real scenario,
- a named real-world incident used illustratively.

**Commands must be real and correct.** Use actual flags for actual tools. If showing output, make it realistic and clearly representative. Never invent tool flags.

**Code must run.** Python 3, PowerShell 5.1+/7, Bash. Include imports. Prefer standard library and well-known packages.

**Diagrams.** Use fenced ```mermaid blocks (flowchart/sequenceDiagram) or ASCII art. At least one diagram per chapter where it aids understanding. Keep mermaid syntax simple and valid — quote labels containing special characters.

**Cross-reference other chapters** with relative paths, e.g. `[Chapter 7](../part-1-foundations/07-networking-deep-dive.md)`. Use the canonical filenames from `../README.md`.

**Safety callouts.** Any destructive, legally sensitive, or malware-adjacent instruction gets a `> ⚠️` block explaining the boundary. Offensive techniques are always framed as "in your lab or on an authorized target."

## Accuracy rules — important

- **Do not fabricate.** No invented statistics, CVE numbers, ISBNs, or URLs.
- **Book references must be real books** — correct title and author. Cite specific chapters/sections where you can. Do not invent editions or page numbers; if unsure of the edition, omit it.
- **URLs**: only canonical, stable, well-known domains (owasp.org, nist.gov, attack.mitre.org, portswigger.net, tryhackme.com, hackthebox.com, kernel.org, learn.microsoft.com, etc.). If unsure a specific deep link exists, link the site root or just name the resource.
- **Numbers**: if you cite a statistic, attribute it to a named source, or phrase it qualitatively instead. Prefer "the large majority of" over an invented percentage.
- **Currency**: written August 2026. Reference NIST CSF 2.0, OWASP Top 10 for LLM Applications 2026, MITRE ATT&CK, current cert exam codes where confident (e.g. Security+ SY0-701). Flag anything volatile as "check current vendor docs."

## Formatting

- Markdown, GitHub-flavored. `---` between major sections.
- Tables for anything comparative.
- Bold for the first use of a defined term.
- No emojis except `⚠️` and `✅` inside blockquote callouts.
- Do not include a `<!DOCTYPE>` or HTML wrapper — plain markdown files.

## Consistency anchors

The reader's lab, built in [Chapter 9](../part-1-foundations/09-building-your-home-lab.md), is:
- Hypervisor: VirtualBox or VMware Workstation (Proxmox as the upgrade path)
- Attacker: **Kali Linux**
- Linux target: **Metasploitable 2** and **Ubuntu Server**
- Web targets: **OWASP Juice Shop**, **DVWA** (Docker)
- Windows: **Windows Server 2022 eval** as domain controller `DC01` in domain `lab.local`, plus **Windows 10/11 eval** client `WS01`
- Blue stack: **Wazuh** or **Security Onion**, with **Sysmon** on Windows hosts
- Networking: host-only/internal network `192.168.56.0/24`, no bridged adapters on vulnerable VMs

Later chapters must use these names, IPs, and domain consistently. If a chapter needs a new VM, name it and note it as an addition to the Chapter 9 lab.

## What "detailed" means here

Bad: "Kerberoasting lets an attacker request service tickets and crack them offline."

Good: explains what an SPN is and why it exists, shows the exact `Rubeus`/`GetUserSPNs.py` invocation with realistic output, shows the resulting `$krb5tgs$23$*