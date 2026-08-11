# Chapter 33 · Certification Strategy

> **Why this chapter matters:** Certifications are the most misunderstood part of breaking into security. Beginners either dismiss them ("just paper") or over-invest ("I'll get ten certs and then apply"). Both are wrong. The truth: **certs get you past automated résumé filters and HR screens; your portfolio and skills get you the offer** (Chapter 32). You need certs *strategically* — the right ones, in the right order, each paired with real skill. This chapter gives you a concrete, budget-aware plan tailored to a beginning CS graduate, and it saves you from the expensive "cert collector" trap.

> **By the end of this chapter you will:** understand what certs actually do (and don't), which to get in what order for your track, how to prepare efficiently, and how to sequence them with the portfolio work from Chapter 32.

> **Currency note:** exam codes, prices, and vendor lineups change. The *strategy* is durable; verify current specifics on the vendor's site (they're the authority — Chapter 0's currency note). Figures below are approximate 2026 USD.

---

## 33.1 What certifications actually do

Be clear-eyed:
- **They pass filters.** Many job applications are screened by ATS software and HR non-specialists who match keywords. "Security+" on the posting and your résumé gets you *seen*. Without the expected cert, a strong candidate can be auto-rejected before a human looks. This is the #1 real function of an entry cert.
- **They prove baseline knowledge** — a common vocabulary and foundation, signaling you're not starting from zero.
- **Some are required** — government/defense roles often *mandate* specific certs (e.g., DoD 8140 recognizes Security+). No cert, no job, regardless of skill.
- **They provide structure** — a syllabus to study against, useful for organizing your learning.

What they **don't** do:
- **They don't prove you can do the work.** A multiple-choice cert says nothing about whether you can get a shell or triage an incident. That's what your portfolio is for. (Practical/hands-on certs are the exception — see below.)
- **They don't substitute for skill.** The "cert collector" with ten certs and no hands-on ability fails technical interviews and is a known anti-pattern. Employers have seen it and are wary.

> **The rule from Chapter 0, restated:** pair every cert with a lab/portfolio project that proves it. A cert with nothing behind it fails the technical interview; a cert *plus* demonstrated skill is a complete package. One at a time, each earned *and* applied.

---

## 33.2 The certification landscape (by tier)

| Tier | Certs | Nature |
|---|---|---|
| **Entry / foundational** | CompTIA Security+, ISC2 CC, Google Cybersecurity Cert, Microsoft SC-900 | Broad knowledge, multiple-choice, no experience needed |
| **Networking/cloud foundation** | CompTIA Network+, Cisco CCNA; AWS Cloud Practitioner, AZ-900 | Fill specific gaps |
| **Practical / hands-on (respected)** | INE eJPT, TryHackMe SAL1, HTB CPTS/CDSA, BTL1, PNPT | *Prove you can actually do it* — increasingly valued |
| **Mid-level / specialist** | CompTIA CySA+ (blue), OSCP (offensive gold standard), AWS Security Specialty, AZ-500 (cloud), GIAC (GSEC/GCIH/GCIA, expensive/respected) | Deeper, role-specific |
| **Advanced / management** | CISSP, CCSP, CISM (require years of experience) | Senior/leadership; not for beginners |

Two things to internalize: (1) **practical certs are rising** — employers increasingly trust "I hacked/defended a real machine under exam conditions" (OSCP, CPTS, PNPT, CDSA, BTL1) over pure multiple-choice; these double as portfolio proof. (2) **Advanced certs (CISSP/CISM/CCSP) are experience-gated** — chasing them now is wasted effort; they're 3–5 years out.

---

## 33.3 Your recommended sequence (CS graduate, beginner)

One at a time, each paired with the relevant portfolio project. Adapt to your target track and budget.

**Step 1 (do this) — CompTIA Security+ (~$425, 2–3 months).** The default first cert. It appears in the majority of entry-level postings, is DoD-recognized, and passes the most filters. It also gives you a structured syllabus covering Parts 1–2, 6 of this booklet. *If you can afford one cert to start, this is it.* Pair with: your documented home lab (Chapter 9).

*Optional cheaper/faster starters* if budget is tight or you're cold: **ISC2 Certified in Cybersecurity (CC)** — exam often **free** via ISC2's One Million Certified program (~$50/yr upkeep after); and/or the **Google Cybersecurity Professional Certificate** (~$300) for a gentle labbed intro. These are fine *pre-*Security+ steps, not replacements for it.

**Step 2 — fill your biggest gap cheaply:**
- Weak networking? **CCNA** or **Network+** (networking pays back forever — Chapter 7).
- Targeting cloud? **SC-900** or **AWS Cloud Practitioner** (~$100, fast) to signal cloud literacy.

**Step 3 — a practical cert in your chosen track** (this is where you prove you can *do* it, and it's the biggest résumé differentiator):
- **Blue/SOC path:** **CompTIA CySA+**, **TryHackMe SAL1**, **BTL1** (Blue Team Level 1 — hands-on, well-regarded), or **HTB CDSA**. Pair with: your SIEM/detection + IR portfolio projects (Chapters 20–22, 32).
- **Offensive path:** **INE eJPT** (~$250, the cheapest proof you enjoy and can do offense — great first offensive cert) → **HTB CPTS** or **TCM PNPT** (~$400–500, credible *practical* OSCP alternatives, fully hands-on with a report). Pair with: your pentest report + web assessment (Chapters 16, 19, 32).
- **Engineering/cloud path:** **AZ-500** or **AWS Security Specialty** (~$165–300). Pair with: your DevSecOps pipeline + cloud project (Chapters 26, 28, 32).

**Step 4 (later, when ready) — the prestige practical cert:**
- **OSCP/OSCP+** (~$1,600+, 6–12 months) — still the offensive gold standard for pentest hiring. Only after CPTS/PNPT-level competence; it's demanding and expensive, so don't rush it.

**Much later (3–5 years, experience-gated):** CISSP/CCSP/CISM for senior/management tracks.

> **The meta-point:** you likely need only **2–3 certs** to land your first role — typically **Security+ + one track-specific practical cert** (+ maybe a cloud foundation). The rest of your credibility comes from the *portfolio*. Don't let cert-chasing crowd out the hands-on work and applications that actually get you hired.

---

## 33.4 How to prepare efficiently

- **Use one primary resource + practice exams.** For Security+, **Professor Messer** (free, excellent videos + free practice) plus a question bank is enough for most. Don't buy five courses for one exam.
- **Practice exams are the highest-signal prep** — they reveal what you don't know and acclimate you to the format. Aim to consistently pass practice tests comfortably before booking the real one.
- **Study for understanding, not memorization** (Chapter 1). Certs test recall, but if you *understand* (from doing the labs), recall follows and the knowledge lasts. Anki (Chapter 1) for the rote facts (ports, acronyms, Event IDs).
- **For practical certs, the prep IS lab work** — you pass CPTS/OSCP by *doing* boxes (HTB/PWK), which is also portfolio building. The prep and the portfolio merge — efficient.
- **Book the exam to create a deadline.** A scheduled date focuses study far better than "someday."
- **Budget-consciously:** many resources are free (Professor Messer, TryHackMe free tier, vendor docs), ISC2 CC is free, and student/voucher discounts exist. You don't need to spend thousands to start — Security+ + eJPT/SAL1 is an affordable, effective opening.

---

## 33.5 Sequencing certs with everything else

Don't do certs in isolation — interleave them with the 12-month plan and portfolio (Chapters 1, 32):
- **Months 1–4:** Foundations + **Security+** (paired with home lab documented).
- **Months 5–8:** Deep track work + a **practical cert** (eJPT/SAL1/CySA+) paired with track portfolio projects.
- **Months 9–12:** Second practical/cloud cert + portfolio polish + **start applying** (Chapter 34) — you don't wait for "all certs done."

Critically: **start applying before you feel fully certified.** You'll never feel ready (Chapter 2's mistake list). Security+ + a couple of solid projects + a practical cert in progress is enough to apply for junior roles. Certs and job-hunting overlap; they're not sequential phases.

---

## Common mistakes

- **Cert collecting with no hands-on work.** Fails technical interviews; a known red flag. Pair every cert with a project.
- **Over-investing before applying.** You need ~2–3 certs, not ten. Don't let cert-chasing delay applications and portfolio work.
- **Chasing CISSP/CISM/OSCP too early.** The first two are experience-gated; OSCP needs real prior competence. Sequence properly.
- **Buying multiple courses per exam.** One good resource + practice exams is usually enough. Save money for the track-specific practical cert.
- **Memorizing instead of understanding.** Understanding (from labs) makes recall durable and passes technical interviews; cramming passes only the multiple-choice.
- **Ignoring practical certs.** They prove ability *and* build portfolio — increasingly the best value. Don't skip them for more multiple-choice.
- **Treating "cert done" as "job-ready."** The portfolio and applications are what convert certs into offers.

---

## Labs

> **Lab 33.1 — Make your cert plan.** Based on your target track (Chapter 2), write your personal 3-cert sequence with target dates and budget: Security+ → (gap-filler if needed) → track practical cert. Tie each to a portfolio project. Put dates on it — deadlines drive completion.

> **Lab 33.2 — Start Security+ (or your first cert).** Begin Professor Messer's free Security+ course. Take a free practice exam *today* (cold) to baseline what you know from this booklet — you'll likely surprise yourself after Parts 1–2. Schedule the exam once you're consistently passing practice tests.

> **Lab 33.3 — Verify current details.** For each cert in your plan, check the vendor's site for current exam code, price, format, and any prerequisites (they change). Note discounts (ISC2 CC free program, student/voucher deals). Build a realistic budget.

> **Lab 33.4 — Align prep with portfolio.** For your chosen practical cert, confirm its prep *is* portfolio-building (HTB/THM boxes → writeups; a report requirement → a portfolio report). Plan so studying and portfolio-building are the same activity — maximum efficiency.

---

## References and further reading

- **Vendor sites are the authority** — CompTIA, ISC2, OffSec, INE, TCM Security, HTB, Microsoft Learn, AWS Training. Verify current details there.
- **Professor Messer** ([professormesser.com](https://www.professormesser.com)) — free, complete Security+ (and Network+/A+) training + practice. The standard free path.
- **Paul Jerimy's Security Certification Roadmap** ([pauljerimy.com/security-certification-roadmap](https://pauljerimy.com/security-certification-roadmap/)) — the well-known visual map of certs by domain and level. Great for orientation.
- **r/cybersecurity, r/CompTIA, and the certification subreddits** — real candidate experiences, current difficulty, and study tips (filter the hype).
- **TCM Security, INE, Hack The Box, TryHackMe** — for the practical cert paths (their prep and the cert overlap with portfolio work).
- **ISC2 "One Million Certified in Cybersecurity"** — the free CC exam program (verify it's still running).
- **The prerequisites document** ([../../00-cybersecurity-foundations-and-prerequisites.md](../../00-cybersecurity-foundations-and-prerequisites.md)) §4 — the cert sequencing table with 2026 costs.

---

## Self-check

1. What is the single most important real-world function of an entry-level certification?
2. Why is "cert collecting" a known anti-pattern, and what's the rule that prevents it?
3. For a beginning CS graduate, what's the recommended *first* cert and why? Name a cheaper/free alternative starting point.
4. Why are practical certs (OSCP, CPTS, SAL1, BTL1) increasingly valued, and how do they double as portfolio pieces?
5. How many certs do you realistically need to land a first role, and where does the rest of your credibility come from?

<details>
<summary>Answers</summary>

1. **Passing automated résumé filters / HR screens.** Many applications are keyword-matched by ATS software and non-specialist screeners, so the cert a posting expects (e.g., Security+) gets you *seen* by a human — without it, strong candidates get auto-rejected before evaluation.
2. Because certs prove baseline knowledge but *not* the ability to do the work, so someone with many certs and no hands-on skill fails technical interviews — a pattern employers recognize and distrust. The preventing rule: **pair every cert with a lab/portfolio project that proves it** (one at a time, earned *and* applied).
3. **CompTIA Security+** — it appears in most entry-level postings, is DoD-recognized, passes the most filters, and provides a structured foundational syllabus. Cheaper/free starting point: **ISC2 Certified in Cybersecurity (CC)**, whose exam is often free via ISC2's One Million Certified program (or the ~$300 Google Cybersecurity Certificate) — good *pre*-Security+ steps, not replacements.
4. Because they require *actually performing* the work under exam conditions (hacking/defending real machines, writing a report), so they prove ability, not just recall — which employers increasingly trust over multiple-choice. They double as portfolio pieces because the prep is hands-on lab/box work and the deliverables (e.g., a full pentest report) are exactly what goes in a portfolio.
5. Realistically **2–3** — typically Security+ plus one track-specific practical cert (and maybe a cloud foundation). The rest of your credibility comes from your **portfolio** (demonstrated projects, writeups, contributions — Chapter 32) and your interview performance, not from accumulating more certificates.

</details>

---

## What's next

With a portfolio and a cert or two (or in progress), it's time to actually get hired. [Chapter 34](34-the-job-hunt.md) is the job hunt: the honest realities of the entry-level market, how to find and target roles, résumé and LinkedIn, networking (where most jobs actually come from), and how to convert interviews — including the technical ones — into offers.
