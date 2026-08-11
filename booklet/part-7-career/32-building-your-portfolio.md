# Chapter 32 · Building Your Portfolio

> **Why this chapter matters:** Knowledge that no one can see doesn't get you hired. The security job market is competitive at the entry level (Chapter 2's honest truth), and the thing that separates people who get interviews from people who don't is **demonstrated, visible skill**. A portfolio is proof — it turns "I studied cybersecurity" into "here is what I can *do*." For a career-changer or new grad without work experience, a strong portfolio is the single most effective substitute for that experience. This chapter shows you how to build one deliberately, from the labs you've been doing all along.

> **By the end of this chapter you will:** know what a security portfolio is, which projects to build (and which you've already started), how to write compelling writeups, how to present it all, and how "learning in public" compounds into opportunity.

---

## 32.1 Why a portfolio beats a résumé bullet

Anyone can write "familiar with penetration testing" on a résumé. A portfolio *proves* it: here's a full engagement writeup, here's the tool I built, here's the detection I engineered and the attack it catches. In a field where employers are burned by people who have certs but can't do the work (Chapter 2's "certs alone" myth), evidence is gold. A portfolio:
- **Substitutes for experience** you don't yet have.
- **Demonstrates communication** — the skill employers say is most lacking (Chapter 19).
- **Shows genuine passion** — you did this because you wanted to, not because a class required it.
- **Gives interviews their content** — "walk me through this project" is a far better interview than abstract trivia, and you control the material.
- **Is discoverable** — recruiters and hiring managers find good public work.

> **The key reframe:** you've been building portfolio material since Chapter 1. Every lab writeup, every attack-and-detect exercise, every tool you scripted is a potential portfolio piece. This chapter is about *polishing and presenting* what you've already done — which is exactly why the booklet told you to write everything up as you went.

---

## 32.2 The projects that actually get you hired

Depth beats breadth: 4–6 *polished, well-documented* projects beat 20 half-finished ones. Choose projects aligned with your target track (Chapter 2's job map), and lead with the highest-impact ones. Here are the strongest, most of which you've already started in the labs:

**Universally strong:**
1. **Home lab with documentation** (Chapter 9) — the baseline expectation. A network diagram, the isolation model, the machines and their roles, reproducibly documented. Shows you can build and reason about environments.
2. **The "attack-and-detect" project** (Chapters 18, 21, 22) — perform an attack (e.g., Kerberoasting) *and* show the exact telemetry, the detection you wrote, and how a defender catches it. **This is the single most impressive beginner project** because it demonstrates both offensive and defensive thinking (purple-team value) in one artifact. If you build only one thing, build this.

**Offensive-track:**
3. **Full penetration test report** (Chapter 19) — a complete, professional-format report from a HTB/THM box or your lab: executive summary, findings, severity, remediation, attack-path narrative. Proves you can *communicate*, not just exploit — the rare and valued skill.
4. **Web app security assessment** (Chapter 16) — assess Juice Shop/DVWA and document the OWASP Top 10 findings with fixes.
5. **CTF writeups** — thoughtful walkthroughs of challenges you solved (the *reasoning*, not just the answer).

**Defensive-track:**
6. **SIEM + detection engineering project** (Chapters 20–21) — ingest logs, write and tune Sigma rules, show a detected attack chain with evidence.
7. **Incident investigation writeup** (Chapters 22–23) — a full DFIR investigation (from LetsDefend/CyberDefenders or your lab) with timeline and analysis.
8. **Threat hunt / detection contribution** — a documented hunt (Chapter 25) or an accepted Sigma rule PR to SigmaHQ.

**Engineering-track (strong CS-grad fit):**
9. **Security tool** (Chapter 8) — a Python/Go tool that solves a real problem (log parser, IOC enricher, cloud misconfig scanner, CI security gate). Clean code, tests, README.
10. **DevSecOps pipeline** (Chapter 28) — a repo with a CI pipeline running SAST/SCA/secret/IaC scanning that catches introduced vulnerabilities.
11. **Cloud security project** (Chapter 26) — an insecure Terraform environment → findings → remediated version, with before/after evidence.
12. **AI/LLM security project** (Chapter 29) — build an LLM app, attack it (prompt injection, excessive agency), then secure it. **Almost no beginner has this — it's a standout differentiator in 2026.**

**Malware/RE-track:**
13. **Malware analysis report** (Chapter 24) — static + dynamic analysis of a sample with IOCs and a YARA rule.

Pick 4–6 across your target track plus the two universal ones. Each should be *complete and polished*.

---

## 32.3 How to write a writeup that stands out

A project is only as good as its documentation — the writeup *is* the portfolio piece (the lab is invisible; the writeup is what people read). Structure each around a clear narrative:

```markdown
# Project Title — one-line description

## Objective / Problem
What was I trying to do or find? Why does it matter?

## Environment / Approach
Setup, tools, methodology. (Show you're systematic.)

## What I did
The steps, with commands, screenshots, and — crucially — MY REASONING.
Why each step, what I expected, what actually happened, how I adapted.

## Findings / Results
What I found or built, with evidence.

## The "why" / mechanism
Explain WHY it worked (offensive) or how the detection catches the behavior
(defensive). This is where you prove understanding, not just execution.

## Defense / Remediation
(On offensive projects, ALWAYS include this — it doubles the value and shows
purple-team thinking.)

## What I learned / would do differently
Reflection shows growth mindset — employers love this.
```

**What makes a writeup exceptional:**
- **Show reasoning, not just steps.** "I tried X, it failed because Y, so I pivoted to Z" demonstrates problem-solving — the actual skill. A list of commands demonstrates copying.
- **Explain the mechanism.** *Why* did the exploit work? *How* does the detection catch the behavior? This is what separates understanding from tutorial-following (Chapter 1's whole point).
- **Include the defense angle** on offensive work (the habit from Chapter 1). It doubles the value and signals maturity.
- **Clear writing.** Communication is a core security skill (Chapter 19). Typos and rambling undercut technical credibility. Edit.
- **Honest about limits.** "I couldn't crack the last box; here's my hypothesis about why" is more credible than false completeness.

---

## 32.4 Where your portfolio lives

- **GitHub** — the hub. A clean profile with well-organized repos (each project with a strong README), a pinned selection of your best work, and a green contribution graph (consistent activity signals dedication). This is the first thing technical interviewers check.
- **A blog** — where writeups shine. Free options: GitHub Pages, a static site (Hugo/Jekyll), Medium, or dev.to. A blog is *discoverable* (SEO, sharing) in a way a private folder isn't, and it demonstrates communication.
- **LinkedIn** — your professional front door (Chapter 34). Link your projects; post about what you're learning.
- **A personal "hub" page** — optional, but a simple site linking your GitHub, blog, key projects, certs, and CV makes you easy to evaluate at a glance.

Keep it professional: a separate identity from personal social media, no illegal or edgy content (you're demonstrating *trustworthiness* for a role with privileged access — Chapter 3), and everything reproducible and honest.

---

## 32.5 Learning in public: the compounding advantage

The highest-leverage career habit in security (introduced in Chapter 1, now the strategy): **share your learning publicly and consistently.** Post writeups, publish tools, tweet/toot what you figured out, contribute to open source, answer questions in communities. Why it compounds:
- **Visibility** — opportunities find people who are visible. Many first jobs come through someone who saw your work.
- **Network** — you build relationships with practitioners who become referrals, mentors, and colleagues (Chapter 34 — most jobs come through people).
- **Reinforcement** — teaching/writing cements your own learning (Feynman, Chapter 1).
- **Reputation** — a body of public work is a reputation that speaks before you do and grows while you sleep.
- **Feedback** — the community corrects and sharpens you.

It feels uncomfortable to share "beginner" work publicly. Do it anyway — everyone starts somewhere, the community is largely supportive, and the person one step behind you learns from exactly the content you feel is too basic. Consistency beats brilliance: one solid writeup a week for a year is a *body of work* that will astonish you (and employers) in hindsight.

> **Contribution ideas that build reputation:** submit a Sigma detection rule (Chapter 21), a tool improvement, a documentation fix to a security project, a well-researched writeup of a technique, a helpful answer in a community. Open-source contributions are especially credible — they're peer-reviewed proof you can work with others' code.

---

## Common mistakes

- **Waiting until you're "good enough" to build a portfolio.** You build it *while* learning, from your labs. Start now.
- **Breadth over depth.** Six polished projects beat twenty abandoned ones. Finish and polish.
- **Undocumented work.** An amazing lab no one can see isn't a portfolio piece. The writeup is the product.
- **Steps without reasoning.** Command lists look like copying. Show your thinking and the *why*.
- **Omitting the defense angle on offensive projects.** It's the maturity signal that sets you apart.
- **Sloppy writing.** Communication *is* the skill; poor writing undercuts technical credibility.
- **Never learning in public.** Invisible skill doesn't convert to opportunity. Share consistently.
- **Anything illegal or unethical in your public work.** You're proving you can be *trusted*. One bad artifact can end that.

---

## Labs

> **Lab 32.1 — Audit what you already have.** List every lab writeup you've produced through this booklet. Pick your 4–6 strongest (aligned to your target track + the two universal projects). This is your portfolio backbone — you've already built it.

> **Lab 32.2 — Set up your GitHub.** Create/clean a professional GitHub profile. Create a well-structured repo for each chosen project with a strong README (use the §32.3 template). Pin your best. Write a profile README introducing yourself and your focus.

> **Lab 32.3 — Polish your flagship: attack-and-detect.** Take your Chapter 18/21 attack-and-detect exercise and turn it into a showcase writeup: the attack, the telemetry, the detection rule (with ATT&CK mapping), test evidence, and defense narrative. Make this your best piece — it's the highest-impact one.

> **Lab 32.4 — Launch a blog.** Set up GitHub Pages (or dev.to/Medium). Publish your first polished writeup. Commit to a cadence (one per 1–2 weeks) you can sustain. Consistency is the whole game.

> **Lab 32.5 — Make one open-source contribution.** Submit something real: a Sigma rule to SigmaHQ, a doc fix or small feature to a security tool, or a helpful PR. Even a small merged contribution is credible, verifiable proof of skill and collaboration.

> **Lab 32.6 — Build the AI-security differentiator.** If you haven't, do Chapter 29's build-break-secure LLM project and write it up thoroughly. In 2026 this single project can set you apart from every other junior candidate.

---

## References and further reading

- **"The Cyber Plumber's Handbook" & TCM Security career content** — practical guidance on building a security career and portfolio.
- **John Hammond, IppSec, LiveOverflow (YouTube)** — see how respected practitioners present technical work; model your writeups on clear explainers.
- **The "How to write a good CTF writeup" guides** (many free) — the craft of the technical writeup.
- **GitHub's "profile README" and Pages documentation** — for setting up your hub and blog.
- **SigmaHQ, Atomic Red Team, and open-source security tools' CONTRIBUTING guides** — where to make your first contributions.
- **"Show Your Work!" — Austin Kleon** — the short, motivating case for learning in public. Read it early.
- **The DFIR Report, PortSwigger research, bug-bounty disclosure writeups** — examples of excellent technical communication to learn from.
- **dev.to #security, r/cybersecurity, security Discords** — communities to share in and learn the norms.

---

## Self-check

1. Why does a portfolio beat a résumé bullet, especially for someone without security work experience?
2. Which single project is the highest-impact for a beginner, and why?
3. What separates an exceptional writeup from a mediocre one?
4. Why include a defense/remediation section on *offensive* portfolio projects?
5. Why is "learning in public" described as compounding, and what's one concrete contribution that builds reputation?

<details>
<summary>Answers</summary>

1. Because a résumé bullet is an unverifiable claim, while a portfolio is *demonstrated, visible proof* of skill — it substitutes for missing work experience, shows communication ability, evidences genuine passion, gives interviews concrete material you control, and is discoverable by recruiters. In a market where employers are wary of "certs but can't do the work," evidence wins.
2. The **attack-and-detect project** — performing an attack (e.g., Kerberoasting) *and* showing the telemetry, the detection you wrote, and how it's caught. It's highest-impact because it demonstrates both offensive and defensive (purple-team) thinking and real understanding of mechanism in a single artifact, which few beginners can show.
3. Showing **reasoning and mechanism**, not just steps: explaining why each step was taken, what failed and how you adapted, *why* the exploit worked or how the detection catches the behavior — plus clear writing, evidence, honesty about limits, and (on offensive work) a defense angle. A command list looks like copying; reasoning proves problem-solving and understanding.
4. Because it demonstrates you understand the *mechanism* (you can defend what you can attack), signals professional maturity and purple-team thinking, and doubles the artifact's value — showing you think about fixing problems, not just finding them, which is exactly what employers want.
5. Because visibility, network, reputation, and reinforcement build on each other over time — consistent public work attracts opportunities, relationships, and referrals while also cementing your own learning, and a growing body of work speaks for you continuously. A concrete reputation-building contribution: submitting an accepted **Sigma detection rule to SigmaHQ** (or another merged open-source security contribution) — verifiable, peer-reviewed proof of skill and collaboration.

</details>

---

## What's next

Your portfolio proves what you can do; certifications get you past the automated filters that guard the interviews. [Chapter 33](33-certification-strategy.md) lays out a concrete certification strategy — which exams, in what order, how to pass them efficiently, and how to avoid the "cert collector" trap — tailored to a beginning CS graduate.
