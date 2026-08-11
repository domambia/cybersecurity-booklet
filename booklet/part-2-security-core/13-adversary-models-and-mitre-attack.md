# Chapter 13 · Adversary Models and MITRE ATT&CK

> **Why this chapter matters:** Red teams, blue teams, and threat intel used to describe attacks in incompatible ways, so nobody could compare notes. MITRE ATT&CK fixed that: it's the shared language of the entire industry. Learn it and you can read any threat report, describe any attack precisely, map your detections to real adversary behavior, and speak fluently in interviews. This chapter is the connective tissue between Part 2 (theory) and Parts 3–4 (offense and defense) — every technique you learn from here on has an ATT&CK ID.

> **By the end of this chapter you will:** understand the kill chain, the Diamond Model, and — in depth — MITRE ATT&CK (tactics, techniques, procedures), the Pyramid of Pain, and how to use ATT&CK to drive both attack and defense.

---

## 13.1 Why models matter

An attack is a *sequence of actions*, not a single event (Chapter 2). To defend, investigate, or emulate it, you need a structured way to describe those actions. Models give you:
- A **shared vocabulary** (so red, blue, and intel understand each other).
- A **checklist** (so you don't miss attack stages or defensive gaps).
- A way to **measure coverage** ("which adversary techniques can we actually detect?").

Three models matter, at increasing resolution: the Kill Chain (strategic, linear), the Diamond Model (analytical), and ATT&CK (tactical, detailed — the one you'll use most).

---

## 13.2 The Cyber Kill Chain (recap + use)

From Chapter 2: seven linear stages — Reconnaissance → Weaponization → Delivery → Exploitation → Installation → Command & Control → Actions on Objectives. Its strength is **strategic clarity** and the insight that breaking *any* link stops the attack (defense in depth). Its weakness: it's linear and perimeter-focused, so it doesn't capture the messy, back-and-forth reality of modern intrusions (attackers loop, pivot, and live off the land inside the network for months). Use it for high-level communication; use ATT&CK for detail.

---

## 13.3 The Diamond Model (analytical)

The **Diamond Model** frames every intrusion event as four linked vertices:

```
        Adversary
        /        \
  Infrastructure  Capability
        \        /
         Victim
```

- **Adversary** — who's doing it.
- **Capability** — their tools/malware/techniques.
- **Infrastructure** — the C2 servers, domains, IPs they use.
- **Victim** — the target.

The value: **pivoting.** Knowing one vertex helps you discover others. Found a malicious IP (infrastructure)? Pivot to what capability it delivers and which other victims it hit. Found malware (capability)? Pivot to the infrastructure it contacts. This is how threat intel analysts *expand* from one indicator to a whole campaign, and it's the mental model behind the correlation work you'll do in Part 4. It complements ATT&CK (which describes *behavior*) by structuring *relationships*.

---

## 13.4 MITRE ATT&CK: the core of this chapter

**ATT&CK** (Adversarial Tactics, Techniques, and Common Knowledge) is a free, continuously-updated knowledge base of *real-world* adversary behavior, observed in actual intrusions and curated by MITRE. It's structured as a **matrix**. Bookmark it now: **attack.mitre.org**.

### The hierarchy: Tactics → Techniques → Procedures (TTPs)

- **Tactics = the WHY** (the adversary's goal for a step). These are the *columns* of the matrix — the phases of an attack. The Enterprise tactics, in rough order:

  | Tactic | Adversary goal |
  |---|---|
  | Reconnaissance | Gather info on the target |
  | Resource Development | Acquire infrastructure/tools |
  | Initial Access | Get in (phishing, exploit, valid accounts) |
  | Execution | Run malicious code |
  | Persistence | Survive reboots/logouts |
  | Privilege Escalation | Gain higher rights |
  | Defense Evasion | Avoid detection |
  | Credential Access | Steal passwords/hashes/tickets |
  | Discovery | Learn the environment from inside |
  | Lateral Movement | Move to other systems |
  | Collection | Gather target data |
  | Command and Control | Communicate with compromised hosts |
  | Exfiltration | Steal the data out |
  | Impact | Destroy/encrypt/disrupt (ransomware, wipers) |

- **Techniques = the HOW** (the method to achieve a tactic). Each has an ID like **T1566 (Phishing)** for Initial Access, or **T1003 (OS Credential Dumping)** for Credential Access — which is exactly the LSASS dumping from Chapter 6. Many have **sub-techniques** (T1566.001 = Spearphishing Attachment).

- **Procedures = the specific implementation** a particular adversary or tool uses to perform a technique (e.g., "APT29 uses this specific PowerShell command to dump credentials").

**Everything you've already learned has an ATT&CK ID.** Kerberoasting = T1558.003. Pass-the-Hash = T1550.002. SUID abuse for privesc = T1548.001. This is the point: ATT&CK is the index of the whole field. As you go through Parts 3–4, tag each technique with its ATT&CK ID in your notes — you're building a mental map of the matrix.

### What each technique page gives you
For any technique, MITRE lists: a description, **procedure examples** (real groups/malware using it), **mitigations** (how to prevent it), **detections** (data sources and analytics to catch it), and which **groups/software** use it. This is gold for defenders — it tells you exactly what to log and what to look for (you'll use this directly in Chapter 21, detection engineering).

### Related MITRE resources
- **ATT&CK Groups** — profiles of tracked threat actors (e.g., APT28, FIN7) and the techniques they use.
- **ATT&CK Navigator** — a free web tool to visualize the matrix, highlight techniques, and build **coverage heatmaps** (e.g., "here's what our detections cover — and the gaps in red").
- **D3FEND** — MITRE's defensive counterpart, mapping defensive techniques.
- **MITRE ATLAS** — the ATT&CK-style matrix for *adversarial machine learning / AI systems* (Chapter 29). Same idea, applied to attacks on ML.
- **MITRE Engage** — for adversary engagement/deception.

---

## 13.5 The Pyramid of Pain (why TTPs matter most)

David Bianco's **Pyramid of Pain** ranks indicators by how much *pain* it causes the adversary when you detect and block them — i.e., how hard they are for the attacker to change:

```
            ▲  Hardest for attacker to change (most valuable to detect)
   /  TTPs        \      ← behaviors; changing these means re-tooling their whole operation
  /  Tools         \     ← their malware/utilities; painful to swap
 /  Network/Host    \    ← C2 domains, registry keys; annoying to change
/  Artifacts         \
─ Domain Names ───────   ← easy-ish to change
─ IP Addresses ───────   ← trivial to change (rotate in minutes)
─ Hash Values ────────   ← trivial (recompile → new hash)
            ▼  Easiest for attacker to change (least valuable to rely on)
```

**The lesson:** detecting a specific file **hash** or **IP** is cheap for the attacker to defeat — they change it instantly. Detecting **TTPs** (the *behavior* — "PowerShell spawned from Word, then dumped LSASS") forces the attacker to fundamentally re-engineer *how they operate*, which is expensive and slow. So **mature defense targets behavior (TTPs), not just indicators.** This is *why* ATT&CK (a catalog of TTPs) is so central: it points your detection at the painful, durable layer. Cheap indicators still matter (they're easy wins), but they can't be your whole strategy.

---

## 13.6 Putting it together: how the industry uses these models

- **Threat intelligence** describes an adversary campaign in ATT&CK terms + Diamond relationships ("APT-X used T1566 for initial access, T1059 for execution, C2 to these domains").
- **Red teams** plan and report engagements as ATT&CK techniques ("we emulated this group's TTPs") — **adversary emulation**.
- **Blue teams / detection engineers** map their detections to ATT&CK, build a Navigator heatmap of coverage, and prioritize filling gaps for techniques their likely adversaries use — **threat-informed defense**.
- **Purple teams** (Chapter 2) use ATT&CK as the shared scorecard: red emulates a technique, blue checks if they detected it, gaps become new detections. The matrix is literally the meeting point.

This is the workflow you'll live in Parts 3 and 4. Learning ATT&CK now means those parts click into a coherent whole instead of a pile of disconnected tricks.

---

## Common mistakes

- **Trying to memorize the whole matrix.** You don't. You learn to *navigate* it and recognize the common techniques; the details are always a click away on attack.mitre.org.
- **Relying only on cheap indicators (hashes/IPs).** They're the bottom of the pyramid — attackers change them in seconds. Aim detection at behavior.
- **Treating the kill chain as the full picture.** It's a strategic summary; real intrusions are non-linear. Use ATT&CK for detail.
- **Learning techniques without their ATT&CK IDs.** Tagging everything with its ID is what builds the industry-standard mental map and makes you fluent.
- **Forgetting the models are descriptive, not prescriptive.** They organize reality; they don't replace understanding *how* each technique actually works (that's Parts 3–4).

---

## Labs

> **Lab 13.1 — Navigate ATT&CK.** On attack.mitre.org, look up five techniques you've already met in this booklet (e.g., Phishing, OS Credential Dumping, Kerberoasting, Valid Accounts, Command and Scripting Interpreter). For each, note its ID, tactic, and one listed detection. Add the IDs to your concept notes.

> **Lab 13.2 — Map a real breach to ATT&CK.** Take the breach you analyzed in Lab 2.2 (or find a detailed incident report). Map each attacker action to an ATT&CK technique and lay them out across the tactics (a mini kill-chain-through-ATT&CK). This is exactly how threat intel reports are written.

> **Lab 13.3 — Build a coverage heatmap.** Install/open the free ATT&CK Navigator. Create a layer and color the techniques you (a) understand and (b) could detect with your lab's logging. The red gaps are your Part 4 study list. Save it — revisit after Chapter 21 to see it fill in.

> **Lab 13.4 — Pyramid of Pain analysis.** Pick a piece of commodity malware (read a public report). List the indicators it exposes at each pyramid level. For each, write how easily the attacker could change it — internalizing why TTP detection is worth more.

> **Lab 13.5 — Diamond pivot.** Given a single indicator (say, a malicious domain from a public report), practice the Diamond Model pivot on paper: from that infrastructure, what capability and victims could you discover, and how? This is the analyst's core reasoning move.

---

## References and further reading

- **MITRE ATT&CK** — [attack.mitre.org](https://attack.mitre.org). The primary source. Explore it weekly; it's the field's index.
- **MITRE ATT&CK Navigator** — [mitre-attack.github.io/attack-navigator](https://mitre-attack.github.io/attack-navigator/). For coverage heatmaps.
- **"Getting Started with ATT&CK" — MITRE (free eBook).** A short, practical guide to using ATT&CK for detection, threat intel, and red teaming. Read it after this chapter.
- **David Bianco — "The Pyramid of Pain" (blog post).** The original; short and foundational. Search "Pyramid of Pain Bianco."
- **Sergio Caltagirone et al. — "The Diamond Model of Intrusion Analysis" (paper).** The source; free online.
- **Lockheed Martin Cyber Kill Chain whitepaper** — the original kill chain.
- **MITRE ATLAS** — [atlas.mitre.org](https://atlas.mitre.org) — for AI/ML adversary techniques (preview for Chapter 29).
- **The DFIR Report** ([thedfirreport.com](https://thedfirreport.com)) — real intrusions mapped to ATT&CK in exquisite detail. Read one report per week; it's the best free way to see these models in action.

---

## Self-check

1. Distinguish tactic, technique, and procedure in ATT&CK, with an example of each.
2. Why does mature detection target TTPs rather than file hashes and IP addresses? Reference the Pyramid of Pain.
3. What is the Diamond Model's key analytical value, and give an example of a pivot.
4. Give the ATT&CK tactic for each: Kerberoasting, phishing to get in, dumping LSASS, encrypting files for ransom.
5. How do red and blue teams both use ATT&CK, and how does that enable purple teaming?

<details>
<summary>Answers</summary>

1. A **tactic** is the adversary's goal/phase (the *why*), e.g., Credential Access. A **technique** is the method to achieve it (the *how*), e.g., T1003 OS Credential Dumping. A **procedure** is a specific implementation, e.g., "this group uses Mimikatz to dump LSASS with a particular command."
2. Because hashes and IPs sit at the bottom of the Pyramid of Pain — the attacker changes them trivially (recompile for a new hash, rotate an IP in minutes), so detections based on them are easily evaded. TTPs sit at the top: detecting *behavior* forces the attacker to re-engineer how they operate, which is expensive and slow, making those detections durable and painful to defeat.
3. It structures an intrusion as relationships among Adversary, Capability, Infrastructure, and Victim, enabling **pivoting** — knowing one vertex helps you discover others. Example: from a malicious C2 IP (infrastructure) you pivot to the malware it delivers (capability) and other hosts contacting it (victims), expanding one indicator into a whole campaign.
4. Kerberoasting → **Credential Access**; phishing for entry → **Initial Access**; dumping LSASS → **Credential Access**; encrypting files for ransom → **Impact**.
5. Red teams plan/report engagements as ATT&CK techniques (adversary emulation); blue teams map detections to ATT&CK and measure coverage (threat-informed defense). Because both use the same framework, purple teaming works: red emulates a technique, blue checks whether they detected it, and each gap becomes a new detection — ATT&CK is the shared scorecard.

</details>

---

## What's next

**Part 2 is complete.** You have the security core: principles, threat modeling, cryptography, identity, and the adversary language that ties it all together. You can now reason like a security professional.

Now the booklet splits into the two great disciplines. [Part 3 · Offensive Security](../part-3-offensive-security/14-reconnaissance-and-osint.md) teaches you to *think and act like an attacker* — legally, in your lab — starting with reconnaissance. Even if you aim for a defensive career, you must understand offense to defend against it. Chapter 14 begins the hands-on attack chain you've been building toward since page one.
