# Chapter 3 · Ethics, Law, and the Professional Line

> **Why this chapter matters:** A penetration tester and a criminal run the same commands. The *only* difference between a security career and a criminal record is **authorization** — and the boundary is far easier to cross by accident than beginners think. This chapter is not filler before the fun stuff. It is the fun stuff's precondition. Read every word. Then reread §3.3 before every offensive lab for your first month.

> **By the end of this chapter you will:** understand exactly what "authorization" means legally and practically, know the categories of computer crime law and why intent isn't a defense, be able to write and read a rules-of-engagement document, and understand responsible disclosure. You'll also have internalized the professional ethics that make people trust you with access to their systems.

---

## 3.1 The uncomfortable truth: the tools are the same

Everything you're about to learn — scanning, exploiting, cracking passwords, escalating privileges, exfiltrating data — is exactly what criminals do. There is no separate "ethical" version of `nmap` or Burp Suite or hashcat. The tool doesn't know or care who's holding it.

What separates you from a criminal is **not** the technique, the intent in your heart, or your good character. It is one thing, and one thing only:

> **You have explicit, documented authorization from the owner of the system to do exactly what you are doing, within an agreed scope.**

That is the entire line. Everything in this chapter elaborates it. Internalize this now: *good intentions are not authorization.* "I was just curious," "I wasn't going to do anything bad," "I was trying to help them find their vulnerability" — none of these are legal defenses, and people have been prosecuted, fired, sued, and had careers ended for exactly these situations. The law in most countries cares about **unauthorized access**, not about what you did after or why.

---

## 3.2 The law: unauthorized access is the crime

You do not need to be a lawyer, but you must understand the shape of computer crime law, because it governs your entire career. The specific statutes vary by country; the *principle* is nearly universal.

**The core offense in almost every jurisdiction: accessing a computer system without authorization, or exceeding the authorization you were given.** Note what's *not* required for it to be a crime:

- **No damage is required.** Simply accessing a system you weren't authorized to access is typically the crime, even if you touched nothing and left no trace.
- **No malicious intent is required.** "Just looking" is still unauthorized access.
- **No profit is required.** You don't have to steal or sell anything.
- **A "helpful" motive is not a defense.** Finding and reporting a vulnerability you had no permission to look for can still be prosecuted as unauthorized access. This has happened to well-meaning researchers repeatedly.

Key legal regimes you should be aware of (know your own jurisdiction's version — this is not exhaustive and is not legal advice):

| Region | Principal law | Core prohibition |
|---|---|---|
| United States | **Computer Fraud and Abuse Act (CFAA)** | Accessing a computer "without authorization" or "exceeding authorized access." |
| United Kingdom | **Computer Misuse Act 1990** | Unauthorized access; unauthorized access with intent; unauthorized modification. |
| European Union | **Directive 2013/40/EU** (implemented per-member-state) | Illegal access, system interference, data interference. |
| Many others | National equivalents | Almost universally center on *unauthorized access*. |

Related laws you can trip over even with authorization to test:
- **Data protection / privacy law** (GDPR in the EU, and national equivalents) — if you access personal data during a test, you may fall under privacy obligations. Handle it accordingly.
- **Wiretapping / interception laws** — capturing network traffic can be regulated even on networks you administer.
- **Export controls** — some security tools and cryptography are export-restricted in some countries.

> ⚠️ **The scary part for a learner:** the boundary is crossable from your keyboard at home in seconds. Running a scan against a website "just to see" is unauthorized access to their system. Trying a default password on a device that isn't yours is unauthorized access. Following a tutorial against a live target that isn't a designated practice target is unauthorized access. The internet makes it feel casual; the law does not treat it casually.

---

## 3.3 What "authorization" actually requires

"Authorization" is not a vibe or a verbal "sure, go ahead." For professional work it is a specific, documented thing. Here is what real authorization looks like and what each part is for.

**1. Written permission from someone who actually owns the system or has the authority to grant access to it.**
- Your manager saying "test our stuff" is not enough if your manager doesn't own the systems or the risk. Permission must come from the right authority.
- A third party cannot authorize you to test a system they don't control. If a client uses a cloud provider, testing may also require the *provider's* permission (many now have standard policies; check them).

**2. A defined scope — exactly which systems, addresses, applications, and accounts are in bounds, and which are explicitly out.**
- Scope is stated as specific IP ranges, domains, URLs, or asset lists. "Everything" is never the scope.
- Out-of-scope systems are off-limits even if you can reach them and even if they're clearly vulnerable. Straying out of scope turns authorized testing into unauthorized access instantly.

**3. Rules of engagement (RoE) — what you may do and what you may not.**
- Allowed techniques and forbidden ones (e.g., "no denial-of-service testing," "no social engineering of staff," "no testing during business hours").
- Data handling rules (what you may access, must not exfiltrate, must delete afterward).
- Timing windows.
- A "get out of jail" contact and an emergency stop procedure if something breaks or if you find evidence of a *real* ongoing breach.

**4. A defined time window** — authorization is not open-ended. It starts and ends on specific dates.

**5. Signatures from both parties** — this is the document that proves, if anyone ever asks, that you were authorized. It protects *you* as much as the client.

> **Practical rule for your whole career:** *If you cannot point to a signed document that says you may do this specific thing to this specific system in this specific window, you are not authorized. Stop.*

For your learning right now, this is why the [safety contract in Chapter 1](01-how-to-use-this-booklet.md) restricts you to exactly four places: your own lab, targets you install, permission-granting platforms (whose *terms of service* are your authorization), and cloud accounts you own. Those are the systems for which your authorization is unambiguous.

---

## 3.4 A rules-of-engagement document (annotated example)

You will read and write these in real work. Here's a simplified, annotated version so the structure is concrete. This is a teaching artifact, not a legal template — real engagements use lawyer-reviewed contracts.

```
PENETRATION TEST AUTHORIZATION & RULES OF ENGAGEMENT

1. PARTIES
   Client: Acme Corp (owner of the systems below)
   Tester: <your company / your name>
   Authorizing signatory: Jane Doe, CTO, Acme Corp  ← must have authority to grant this

2. AUTHORIZATION STATEMENT
   Acme Corp authorizes Tester to perform security testing against the
   systems listed in Scope, during the Window, using the methods in RoE.
   ← THIS is the sentence that makes it legal.

3. SCOPE (in-bounds)
   - Web application: https://app.acme.example  (production)
   - API: https://api.acme.example
   - IP range: 203.0.113.0/28
   OUT OF SCOPE (explicitly forbidden):
   - All other Acme systems, subdomains, and IPs
   - Any third-party/cloud-provider infrastructure
   - Physical premises and staff (no social engineering)
   ← Out-of-scope is as important as in-scope. Everything not listed is forbidden.

4. RULES OF ENGAGEMENT
   - Allowed: automated + manual web testing, authenticated testing with
     provided test accounts.
   - Forbidden: denial-of-service / load testing; modifying or deleting
     production data; accessing real customer PII (use seeded test data).
   - Testing window: 09:00–17:00 UTC, Mon–Fri only.
   - If real customer data or an active third-party breach is discovered,
     STOP and notify the emergency contact immediately.

5. WINDOW
   Start: 2026-09-01 00:00 UTC   End: 2026-09-14 23:59 UTC
   ← Authorization does not exist before the start or after the end.

6. CONTACTS
   Emergency contact: <name, phone, email> — reachable during testing.
   ← The "get out of jail" line if something breaks.

7. DATA HANDLING
   All findings and any accessed data are confidential, encrypted at rest,
   and destroyed within 30 days of report delivery.

8. SIGNATURES
   Client signatory: __________  Date: ______
   Tester:           __________  Date: ______
   ← No signatures = no authorization = do not start.
```

Read that structure until it's second nature. When you eventually do paid work, the presence or absence of each element is what stands between you and serious legal exposure.

---

## 3.5 Responsible disclosure: what to do when you find something you weren't looking for

Sooner or later you'll stumble on a real vulnerability in a system you don't own — a website you use, an app on your phone, an IoT device. What you do next matters enormously, legally and ethically.

**The spectrum of disclosure:**

- **Full disclosure** — publish the vulnerability publicly and immediately. Pressures vendors to fix, but arms attackers before a patch exists. Generally irresponsible unless a vendor is ignoring a serious, actively exploited flaw.
- **Non-disclosure** — tell no one. The flaw festers.
- **Coordinated (responsible) disclosure** — privately report to the vendor, give them reasonable time to fix (commonly 90 days), then optionally publish once patched. **This is the professional standard.**

**How to do it safely:**
1. **Check for a policy first.** Look for a `security.txt` file (at `https://site/.well-known/security.txt`), a "responsible disclosure" or "vulnerability disclosure program (VDP)" page, or a **bug bounty program** on platforms like HackerOne or Bugcrowd. If one exists, it is your authorization and your instructions — follow it exactly.
2. **If a bug bounty program exists and lists the asset in scope**, you have permission to test *within that scope* and may even get paid. Stay strictly inside the defined scope — bug bounty scope rules are legally binding boundaries, not suggestions.
3. **If there's no program**, report through official channels (a security contact, not a random support form if you can avoid it), describe the issue clearly and minimally, and do **not** access more than the minimum needed to demonstrate it. Don't download customer data to "prove" it — proving you *could* is enough and staying minimal keeps you defensible.
4. **Do not extort.** "Pay me or I'll publish" is a crime, not disclosure.
5. **Get the timeline in writing** and be patient and professional.

> ⚠️ **The researcher's trap:** the moment you go *beyond* what's needed to confirm a bug — pulling a database, accessing other users' accounts, "just checking how deep it goes" — you've likely committed unauthorized access, and a "responsible disclosure" framing won't necessarily protect you. Confirm minimally, report, stop. Researchers have been threatened with prosecution for exactly this overreach, even acting in good faith.

**Bug bounty programs are the safest place to earn while you learn**, because they grant explicit, scoped authorization in writing. Once you're through Part 3, they're a legitimate way to practice offense legally and build a reputation. Read a program's scope and rules as carefully as you'd read a contract — because that's what they are.

---

## 3.6 Professional ethics beyond the law

Legal is the floor, not the ceiling. Security professionals are given extraordinary access — to systems, to data, to the keys of the kingdom. That access is granted on **trust**, and trust is the actual currency of this career. Some principles that go beyond what any law requires:

- **Confidentiality of what you find.** When you test a client's systems, you learn their weaknesses, their data, sometimes their secrets. You protect that as if it were your own — during the engagement, after it, forever. A tester who gossips about a client's flaws is finished.
- **Do no harm.** Even authorized, you act to minimize disruption. You don't run a risky exploit on production if a safer proof exists. You don't access more data than needed. You leave systems as you found them.
- **Honesty in findings.** You don't exaggerate a low-risk finding to justify your fee, and you don't quietly bury a critical one because it's embarrassing to report or hard to explain. Your report is only valuable if it's true.
- **Stay within scope even when you *can* go further.** The ability to do something is not permission to do it. This discipline, repeated, is what makes you trustworthy.
- **Competence and honesty about your limits.** Don't take on work you can't do safely. Say "I don't know" when you don't. In security, confident wrongness gets people breached.
- **Consider the human impact.** Behind every system is a person whose data, safety, or livelihood may be affected. Security is ultimately about protecting people, not scoring points.

Several bodies publish formal codes of ethics you'll encounter — **(ISC)² Code of Ethics**, **EC-Council**, **SANS/GIAC**. They differ in wording but share a spine: protect society and infrastructure, act honorably and legally, provide diligent competent service, and advance the profession. When you certify, you'll formally agree to one. Read it as a real commitment, not a checkbox.

---

## 3.7 The gray areas (and how to stay out of them)

Some situations feel ambiguous. Resolve every one of them conservatively — the cost of caution is a missed learning opportunity; the cost of overstepping is your career.

- **"It's my friend's website, they said it's fine."** Get it in writing, and make sure they actually own it (not a hosting provider's shared infrastructure that also hosts others). Verbal permission from a friend has failed people in court.
- **"The company has a bug bounty, so I can test anything of theirs."** No — only what the program lists *in scope*. Out-of-scope assets are unauthorized even for that same company.
- **"I found an open database on the internet, I'll just look."** That's unauthorized access. Report the exposure through proper channels (or a CERT) without accessing the contents.
- **"I'm scanning the whole internet for research."** Even scanning can be treated as unauthorized/hostile activity and violate ISP terms; academic internet-scale scanning is done under careful ethical and legal review, not casually.
- **"My employer told me to hack a competitor."** Illegal instructions are still illegal. "I was told to" is not a defense. Refuse, document, and get advice.
- **"I want to test the login of a site I have an account on."** Having an account authorizes normal *use*, not security testing. Different thing entirely.

> **The one-question test for any gray area:** *Can I produce written authorization from the system's owner covering exactly this action, right now?* If the honest answer is anything other than a clear yes, don't do it. There is always a legal practice target for the same skill.

---

## Common mistakes this chapter is trying to prevent

- **Treating this chapter as a formality to skim before the real content.** This *is* the content that keeps you employable and free.
- **Assuming good intentions are a legal shield.** They are not. Authorization is.
- **Practicing against live targets from tutorials.** Tutorials sometimes point at real sites. Never follow one against anything but a designated practice target.
- **Overreaching during disclosure** — turning a good-faith find into unauthorized access by "checking how far it goes."
- **Confusing "I have an account / access" with "I'm authorized to test."** Use is not testing.
- **Assuming scope is flexible.** It is a hard legal boundary, in both pentests and bug bounties.

---

## Labs

> **Lab 3.1 — Learn your jurisdiction.** Look up the primary computer-misuse law in your country (e.g., CFAA in the US, Computer Misuse Act in the UK, your national statute elsewhere). Write a one-paragraph summary in your notes of what it prohibits and what the penalties are. You should know the law that governs your own career. *(This is research, not legal advice — but every practitioner should know their local statute's shape.)*

> **Lab 3.2 — Write a rules-of-engagement document.** Using the template in §3.4, write a complete RoE for a fictional engagement: you're testing a fictional company's web app. Invent the scope, rules, window, and contacts. This is a real professional skill and a good portfolio artifact.

> **Lab 3.3 — Find real disclosure policies.** Visit three companies you use and look for their vulnerability disclosure policy: check `https://<their-domain>/.well-known/security.txt` and search "`<company>` responsible disclosure" or "`<company>` bug bounty." Note what you find. Notice which companies make it easy to report and which make it impossible — that itself is a security signal.

> **Lab 3.4 — Read a code of ethics.** Read the (ISC)² Code of Ethics in full (it's short). Write, in your own words, which of its canons you think will be hardest to uphold under real-world pressure, and why. This reflection matters more than it looks.

---

## References and further reading

- **Your national computer-crime statute** — read the actual text once. It's more readable than you'd expect and it governs your work.
- **(ISC)² Code of Ethics** — [isc2.org](https://www.isc2.org) → Code of Ethics. Short, foundational, and you'll formally agree to it if you certify.
- **EFF (Electronic Frontier Foundation)** — [eff.org](https://www.eff.org) — writes clearly about the CFAA, security research law, and the legal risks researchers face. Read their coverage of security-research prosecutions to understand how good-faith researchers have gotten into trouble.
- **`security.txt` standard** — [securitytxt.org](https://securitytxt.org) — the standardized way organizations publish security contacts.
- **disclose.io** — open-source standards and templates for vulnerability disclosure and safe-harbor language.
- **HackerOne and Bugcrowd** — [hackerone.com](https://www.hackerone.com), [bugcrowd.com](https://www.bugcrowd.com) — read a few real bug bounty program scopes to see how authorization is defined in practice.
- **The CFAA and *Van Buren v. United States*** — if you're in the US, read a summary of this Supreme Court case; it narrowed "exceeds authorized access" and is genuinely relevant to what researchers can and can't do.

---

## Self-check

1. Two people run the identical `nmap` command against the identical server. One is committing a crime and one isn't. What is the single thing that distinguishes them?
2. Your manager says "go ahead and test the payment system." Name two reasons this might still not constitute valid authorization.
3. You find a serious vulnerability in a website you use. Walk through, in order, the responsible steps — and name the specific action that would turn your good-faith find into a likely crime.
4. Why is "out of scope" as important as "in scope" in a rules-of-engagement document?
5. A company has a bug bounty program. Does that authorize you to test any system the company owns? Explain.

<details>
<summary>Answers</summary>

1. **Authorization** — one has explicit, documented permission from the system's owner covering that exact action; the other doesn't. Nothing about the command, the tool, or the intent differs.
2. (a) Your manager may not have the authority to grant permission for that system or to accept that risk; (b) valid authorization needs to be documented with defined scope, rules, and a time window and come from someone who actually owns/controls the system — a casual verbal go-ahead lacks all of that.
3. Check for a disclosure policy / `security.txt` / bug bounty program; if one exists, follow it; confirm the bug with the *minimum* access needed; report privately and clearly through official channels; give reasonable time; don't extort; optionally coordinate public disclosure after a fix. The crime-turning action: **going beyond minimal confirmation** — e.g., downloading customer data or accessing other users' accounts to "see how deep it goes" — which is unauthorized access regardless of good intent.
4. Because everything not explicitly in scope is forbidden, and straying onto an out-of-scope system — even the same client's — instantly converts authorized testing into unauthorized access. Naming out-of-scope assets removes ambiguity and protects both parties.
5. No. It authorizes testing only of the specific assets the program lists **in scope**, under the program's rules. Company-owned systems not listed in scope are out of bounds and testing them is unauthorized.

</details>

---

## What's next

Part 0 is complete: you know how to study, what the field is, and where the legal and ethical lines are. Now the real technical work begins. [Chapter 4](../part-1-foundations/04-computing-fundamentals.md) goes under your code — memory, processes, how a program actually runs — because you cannot exploit or defend what you don't understand at the machine level. Part 1 is the longest and most important part of this booklet. Take your time with it.
