# Chapter 29 · AI and LLM Security

> **Why this chapter matters:** In 2026, AI/ML security is the single most in-demand *and* least-saturated skill in the field — organizations are deploying LLMs and autonomous agents everywhere and have almost nobody who can secure them. This is your differentiator. AI changes security on three fronts at once: it creates **new systems to secure** (AI applications), gives **attackers new capabilities**, and gives **defenders new tools**. This chapter builds directly on the whole booklet — prompt injection is Chapter 4's "context confusion," AI supply chain is Chapter 28, agentic risk is least privilege (Chapter 10) — reapplied to a new and fast-moving domain.

> **By the end of this chapter you will:** understand the AI attack surface, the OWASP LLM Top 10 (2026) and MITRE ATLAS, why prompt injection is (currently) unsolvable at the model layer, how to secure LLM and agentic applications, and how AI is changing both offense and defense.

> **Currency note:** this is the fastest-moving area in security. Concepts here are durable; specific tools, models, and rankings change fast. The OWASP LLM Top 10 2026 edition (released August 2026) is the current reference.

---

## 29.1 The three ways AI intersects security

1. **Securing AI systems** — LLM apps, agents, and ML models are new software with new vulnerability classes. This is the bulk of the *new* work and this chapter's focus.
2. **AI as an attacker's tool** — AI lowers the skill floor and raises the volume of attacks: more convincing phishing at scale, faster vulnerability discovery, automated recon, malware assistance, deepfakes for social engineering. The 2026 reality is *more* attacks, *better* crafted.
3. **AI as a defender's tool** — AI assists alert triage, log summarization, detection writing, code review, and threat intel synthesis. Powerful, but with a critical caveat: **it hallucinates and can be manipulated**, so it augments analysts, never replaces judgment (Chapter 1's AI-study caution, professionalized).

The scarce, valuable skill is fluency across all three — and especially #1, which almost no one has yet.

---

## 29.2 The AI/LLM attack surface

An LLM application is more than a model — it's a system: the model, its **prompts** (system + user), its **training/fine-tuning data**, its **context sources** (RAG — retrieval-augmented generation, pulling in documents), its **tools/plugins** (functions the model can call), and the surrounding app. Each is attack surface.

The foundational insight (straight from Chapter 4): **LLMs cannot reliably distinguish instructions from data.** Everything — the system prompt, the user's message, a retrieved document, a tool's output — arrives as *text in the same context*, and the model may treat any of it as an instruction. This is **context confusion** at the semantic level, and it's why the entire field of LLM security exists. You cannot fully "patch" it in the model; you must architect around it.

---

## 29.3 The OWASP Top 10 for LLM Applications (2026)

The **OWASP Top 10 for LLM Applications** is the industry-standard risk list (2026 edition released at Black Hat, notably now weighted partly by *real-world incident data*). The key risks:

- **Prompt Injection (still #1)** — attacker input manipulates the model into ignoring its instructions or doing something unintended. **Direct** ("ignore previous instructions and...") and — more dangerous — **indirect**: malicious instructions hidden in content the model *retrieves* (a web page, a document, an email the agent reads), so the attacker doesn't even talk to the model directly. This is the defining LLM vulnerability. It maps directly to injection (Chapter 16) and context confusion (Chapter 4).
- **Sensitive Information Disclosure (#2)** — the model leaks secrets: training data, other users' data, system prompts, or data from its context/RAG. Includes the model revealing PII or credentials it shouldn't have access to.
- **Hidden Context Exposure** (broadened in 2026 from "System Prompt Leakage") — leakage of *any* non-user-visible context: system instructions, RAG schemas, hidden policy logic. The lesson: never rely on the system prompt being secret, and never put secrets/authorization logic in it.
- **Unbounded Consumption** (rose sharply in 2026) — resource/financial denial of service: expensive queries, "extended-thinking" abuse, and cost-exhaustion attacks against inference. An availability *and* cost problem unique to AI (recall CIA — availability, Chapter 2).
- **Supply chain** — compromised models, poisoned datasets, malicious model files (a downloaded model can execute code on load), vulnerable ML dependencies. This is Chapter 28's supply chain applied to AI — plus model-specific risks like backdoored weights.
- **Insecure Output Handling** — trusting the model's output and passing it unsanitized into other systems (rendering it as HTML → XSS, executing it as code, using it in a SQL query → injection). **Treat LLM output as untrusted user input** — a critical, often-missed principle.
- **Excessive Agency** — giving the model/agent too much power (tools, permissions, autonomy) so that a successful prompt injection causes real-world damage (sends emails, moves money, deletes data). This is **least privilege** (Chapter 10) for AI, and it's the crux of agentic risk (below).
- Plus: training-data poisoning, improper output/plugin design, and misinformation/over-reliance.

---

## 29.4 The core defensive philosophy (the 2026 insight)

The most important idea in modern AI security, emphasized in the 2026 guidance:

> **Don't try to build a model that "cannot be fooled." Assume the model *will* be fooled, and architect the surrounding system so that a compromised model can't cause harm.**

Because prompt injection has no reliable model-layer fix (you cannot guarantee the model won't be manipulated by cleverly crafted text), security must live in the **architecture around** the model — exactly the defense-in-depth and least-privilege thinking from Chapter 10:

- **Least privilege for AI agency** — give the model/agent the *minimum* tools and permissions; assume any tool it can call, an attacker can trigger via injection. An agent that can only read one folder is far safer than one with admin.
- **Human-in-the-loop for consequential actions** — require human approval before the model does anything irreversible or high-impact (spend money, send external email, delete data).
- **Sandboxing and output validation** — run model-triggered actions in constrained environments; validate/sanitize outputs before they touch other systems (treat output as untrusted — see Insecure Output Handling).
- **Privilege separation** — the model should not hold credentials or authorization it can be tricked into misusing; enforce authorization *outside* the model, in code (the model requests, the system authorizes — never trust the model to enforce access control).
- **Input/output filtering and monitoring** — guardrails, prompt-injection detection (imperfect but useful defense-in-depth), and logging model interactions for detection (Part 4).
- **Isolation of untrusted content** — clearly separate and constrain retrieved/third-party content the model processes (the indirect-injection vector).

The pattern is identical to every hard security problem in this booklet: you can't make the vulnerable component perfect, so you *contain the blast radius*.

---

## 29.5 Attacks on ML models themselves

Beyond LLM apps, classical adversarial ML attacks apply (and **MITRE ATLAS** — the ATT&CK for AI, Chapter 13 — catalogs them):
- **Adversarial examples** — inputs crafted to fool a model (imperceptible changes that make a classifier misidentify an image; evasion of ML-based malware/spam detectors).
- **Data poisoning** — corrupting training data to create backdoors or degrade the model (ties to supply chain).
- **Model extraction/stealing** — querying a model enough to replicate it (IP theft).
- **Model inversion / membership inference** — recovering training data or determining whether specific data was in the training set (privacy attacks — Chapter 31).
- **Evasion of AI-based defenses** — since defenders use ML (spam filters, EDR), attackers specifically craft inputs to slip past them.

**MITRE ATLAS** ([atlas.mitre.org](https://atlas.mitre.org)) is your ATT&CK-equivalent map for these — learn to navigate it as you did ATT&CK.

---

## 29.6 Securing AI in your own organization (the practical mandate)

Real orgs are deploying AI faster than they can secure it — which is the job. The practical checklist:
- **Inventory AI usage** (including **shadow AI** — employees pasting sensitive data into public chatbots, a major real data-leak vector — Chapter 3's confidentiality).
- **Data governance** — what data goes into prompts, RAG, and training? Prevent sensitive data from leaking into or out of AI systems.
- **Secure the AI supply chain** — vet models and datasets; scan model files (they can be malicious); treat model provenance like any dependency (Chapter 28).
- **Apply the architectural defenses** from §29.4 to every LLM/agent feature.
- **Red-team the AI** — test your LLM apps for prompt injection, data leakage, and excessive agency (offensive AI security — a growing role). Tools/frameworks for LLM red-teaming exist (e.g., Garak, PyRIT, promptfoo) and the OWASP Agentic Top 10 guides agent testing.
- **Monitor and log** AI interactions for abuse and detection (Part 4 applied to AI).
- **Policy** — acceptable-use rules for AI, aligned to emerging AI regulation (the EU AI Act and others — Chapter 31).

> **Why this is your biggest career lever:** the demand is enormous, the supply is tiny, and *you* — a CS graduate who's just learned the entire security stack — are unusually positioned to bridge AI and security. Almost no one can both build an LLM app and threat-model it. Being that person in 2026 is close to a career cheat code. Build an AI app, break it, secure it (Lab 29.x), and write it up — few beginners have this, and everyone's hiring for it.

---

## Common mistakes

- **Thinking prompt injection is patchable at the model layer.** It isn't (yet). Assume the model will be fooled; secure the architecture around it.
- **Trusting LLM output.** Treat it as untrusted user input — sanitize before rendering/executing/querying, or you reintroduce XSS/injection (Chapter 16).
- **Giving agents excessive agency.** Least privilege for AI — an over-empowered agent + prompt injection = real-world damage.
- **Putting secrets or authorization logic in the system prompt.** It leaks; enforce authz in code outside the model.
- **Ignoring indirect injection.** The dangerous vector is malicious instructions in *retrieved* content, not just direct user input.
- **Ignoring shadow AI / data governance.** Employees leaking data into public models is a real, common breach path.
- **Assuming AI defensive tools are trustworthy oracles.** They hallucinate and can be manipulated; augment judgment, don't replace it.

---

## Labs

> **Lab 29.1 — Build then break an LLM app.** Build a small LLM-powered app (e.g., a RAG chatbot over some documents, using an API). Then attack it: attempt **direct prompt injection** (override its instructions), **indirect injection** (hide an instruction in a document it retrieves), and **data/system-prompt leakage**. Document each. **This build-and-break project is a standout portfolio piece almost no beginner has.**

> **Lab 29.2 — Excessive agency demo.** Give your app a "tool" (even a mock one that "sends an email" or "deletes a file"). Show how a prompt injection can trigger it. Then implement the fixes: least-privilege tools, human-in-the-loop approval, output validation. Write up the before/after architecture.

> **Lab 29.3 — OWASP LLM Top 10 walkthrough.** For each of the 2026 OWASP LLM Top 10 risks, write a one-paragraph explanation and a concrete mitigation, mapping each to a principle from earlier chapters (injection→Ch16, excessive agency→Ch10, supply chain→Ch28). This synthesis cements the chapter.

> **Lab 29.4 — Gandalf and prompt-injection games.** Play **Lakera's Gandalf** (a free prompt-injection challenge) and similar LLM CTFs. They build intuition for how models get manipulated. Then read a few real prompt-injection writeups.

> **Lab 29.5 — Explore MITRE ATLAS.** Navigate ATLAS as you did ATT&CK (Chapter 13). Pick three techniques (e.g., an adversarial-example evasion, a data-poisoning case) and note the real-world case studies. Map how ATLAS complements ATT&CK.

> **Lab 29.6 — LLM red-teaming tools.** Try a free LLM red-teaming framework (**garak**, **promptfoo**, or **PyRIT**) against a small model/app. Understand how automated AI red-teaming works. This is directly job-relevant for the emerging AI-security roles.

---

## References and further reading

- **OWASP Top 10 for LLM Applications (2026) & OWASP Top 10 for Agentic Applications (2026)** — [genai.owasp.org](https://genai.owasp.org). **The** primary references. Read both fully; they're the field's syllabus.
- **MITRE ATLAS** — [atlas.mitre.org](https://atlas.mitre.org) — the adversarial-ML knowledge base. Learn it like ATT&CK.
- **NIST AI Risk Management Framework (AI RMF)** — the authoritative governance framework for trustworthy AI.
- **Lakera Gandalf** ([gandalf.lakera.ai](https://gandalf.lakera.ai)) — free, addictive prompt-injection practice.
- **garak, PyRIT (Microsoft), promptfoo, Giskard** — free LLM red-teaming/testing tools.
- **Simon Willison's blog** — the best ongoing, practical writing on prompt injection and LLM security (search "Simon Willison prompt injection"). Read regularly to stay current.
- **"The AI Attack Surface" resources, Embrace The Red blog (Johann Rehberger)** — real, creative LLM/agent attack research.
- **Anthropic, OpenAI, Google safety/security documentation** — provider guidance on building safe AI applications (prompt-injection mitigations, tool-use safety).

---

## Self-check

1. Why can't prompt injection be reliably fixed at the model layer, and what is the resulting core defensive philosophy?
2. Distinguish direct and indirect prompt injection, and explain why indirect is more dangerous.
3. What is "excessive agency," which earlier principle addresses it, and how?
4. Why must you treat LLM output as untrusted, and what earlier vulnerability class does mishandling it reintroduce?
5. What is MITRE ATLAS, and name three attacks on ML models it would catalog.

<details>
<summary>Answers</summary>

1. Because an LLM processes system instructions, user input, and retrieved content as text in the *same context* and cannot reliably distinguish instructions from data (semantic context confusion) — so cleverly crafted text can always potentially manipulate it, with no guaranteed model-layer fix. The resulting philosophy: **assume the model will be fooled and architect the surrounding system** (least privilege for agency, human-in-the-loop, sandboxing, output validation, authorization enforced in code) so a compromised model can't cause harm.
2. **Direct** injection is malicious instructions in the user's own input to the model ("ignore your instructions and..."). **Indirect** injection hides instructions in *content the model retrieves* (a web page, document, or email an agent reads). Indirect is more dangerous because the attacker never interacts with the model directly — they just plant malicious content the victim's AI will later ingest — making it stealthier and able to target other users/agents.
3. Excessive agency is giving the model/agent too many tools, permissions, or autonomy, so a successful prompt injection can cause real-world damage (send money, delete data, email externally). **Least privilege** (Chapter 10) addresses it: grant the agent the minimum tools/permissions needed, require human approval for consequential actions, and enforce authorization outside the model — shrinking the blast radius of any manipulation.
4. Because the model can be manipulated (via prompt injection) or simply hallucinate, so its output may contain attacker-controlled or malicious content. If you pass that output unsanitized into other systems, you reintroduce classic **injection/XSS** (Chapter 16) — e.g., rendering it as HTML (XSS), executing it as code, or using it in a SQL query. So LLM output must be validated/sanitized like any untrusted user input.
5. MITRE ATLAS is the ATT&CK-equivalent knowledge base of adversarial tactics and techniques against AI/ML systems. Three cataloged attacks: **adversarial examples** (crafted inputs that fool a model), **data poisoning** (corrupting training data to backdoor/degrade the model), and **model extraction/inversion or membership inference** (stealing the model or recovering/identifying its training data).

</details>

---

## What's next

**Part 5 is complete.** You now cover the modern platforms where security is most in demand — cloud, containers, DevSecOps, and AI. Combined with your offensive and defensive skills, you're a rare and valuable profile. Now the booklet turns to how security is *governed, funded, and made legal*. [Part 6 · Governance, Risk, and Compliance](../part-6-governance-risk-compliance/30-grc-frameworks-and-risk-management.md) — even deeply technical practitioners must understand this to be effective and to advance. Chapter 30 covers the frameworks and risk management that run every security program.
