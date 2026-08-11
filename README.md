# The Cybersecurity Practitioner's Booklet

**A complete, hands-on self-study cybersecurity course — from computer-science graduate to security professional.**

📖 **Read it online → https://kevin-fincruse.github.io/cybersecurity-booklet/**

---

## What this is

A book-length (~95,000-word) course that takes a beginner with a CS background from "what is cybersecurity?" all the way to an employable practitioner with a portfolio, a specialism, and a plan for mastery. **36 chapters across 7 parts, plus 6 appendices.**

Every chapter follows the same structure: **why it matters → what you'll be able to do → the material with worked examples → common mistakes → hands-on labs → book references → self-check with answers → what's next.**

## Contents

| Part | Focus |
|---|---|
| **0 · Orientation** | How to study, what the field is, ethics & law |
| **1 · Foundations** | Computing internals, Linux, Windows/Active Directory, networking, scripting, home lab |
| **2 · Security Core** | Principles & threat modeling, applied cryptography, identity, MITRE ATT&CK |
| **3 · Offensive Security** | Recon, scanning, web hacking, exploitation, AD attack paths, pentest reporting |
| **4 · Defensive Security** | SIEM, detection engineering, incident response, forensics, malware analysis, threat hunting |
| **5 · Modern Platforms** | Cloud, containers/Kubernetes, DevSecOps & supply chain, AI/LLM security |
| **6 · Governance, Risk & Compliance** | Frameworks & risk management, privacy & regulation |
| **7 · Career** | Portfolio, certifications, job hunt, first 90 days, path to mastery |
| **Appendices** | Cheat sheets, annotated bibliography, glossary, 52-week schedule, self-assessment rubrics, lab build files |

## Repository layout

```
.
├── index.html      # the full booklet as a self-contained website (GitHub Pages entry point)
├── booklet/        # the source Markdown (one file per chapter + appendices, organized by part)
├── 00-cybersecurity-foundations-and-prerequisites.md   # the prerequisites / roadmap doc
└── build-reader.js # Node script that generates index.html from the Markdown
```

You can read the raw Markdown in [`booklet/`](booklet/) (start with [`booklet/README.md`](booklet/README.md)) or use the rendered website.

## Regenerating the website

`index.html` is generated from the Markdown by `build-reader.js` (Node, no dependencies). It renders every chapter into one navigable single-page reader with a sidebar, light/dark themes, and Mermaid diagrams.

> ⚠️ **Ethics & legality.** This course teaches offensive techniques for **authorized, educational, and defensive** purposes only. Practice exclusively on systems you own or are explicitly authorized to test (your own lab, or permission-granting platforms like TryHackMe, Hack The Box, and PortSwigger). Never test systems without written permission — see **Chapter 3: Ethics, Law, and the Professional Line**.

## Currency

Written August 2026; reflects that landscape (e.g. the OWASP Top 10 for LLM Applications 2026 edition and NIST CSF 2.0). Concepts endure; tool versions, prices, and exam codes change — verify specifics against vendor documentation.

---

_Content © its author. Shared for education. Learn ethically; protect people._
