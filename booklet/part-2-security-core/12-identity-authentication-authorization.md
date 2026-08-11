# Chapter 12 · Identity, Authentication, and Authorization

> **Why this chapter matters:** "Who are you, and what are you allowed to do?" is the question at the center of nearly every breach. Broken access control is the #1 web application risk. Stolen credentials are among the most common breach causes year after year. Identity and Access Management (IAM) is the #1 cloud attack surface. If you deeply understand authentication (proving identity) and authorization (granting rights), you understand where most real attacks succeed — and how to stop them.

> **By the end of this chapter you will:** distinguish authentication from authorization precisely, understand factors and MFA, SSO/federation and the protocols behind them, access-control models (RBAC/ABAC), and the classic identity attacks (credential stuffing, session hijacking, privilege escalation, broken access control).

---

## 12.1 The core distinction (never confuse these)

- **Authentication (AuthN)** — *proving who you are.* Logging in.
- **Authorization (AuthZ)** — *determining what you're allowed to do* once identified.
- **Accounting/Auditing** — *recording what you did.*

Together: **AAA** (Authentication, Authorization, Accounting). A door analogy: authentication is proving it's your key; authorization is which rooms your key opens; accounting is the log of every door you went through.

**Why the distinction matters for security:** they fail differently and are attacked differently. An **authentication** failure lets an attacker *become* a user (credential theft, weak passwords, MFA bypass). An **authorization** failure lets an already-authenticated user *do more than they should* (access others' data, perform admin actions) — this is **broken access control**, the most common serious web flaw. Interviewers *love* to check that you don't blur these.

---

## 12.2 Authentication: proving identity

Authentication relies on **factors** — categories of evidence:

1. **Something you know** — password, PIN, security question. Weakest; reusable, guessable, phishable, stealable.
2. **Something you have** — a phone (for a code), a hardware token/security key, a smart card.
3. **Something you are** — biometrics (fingerprint, face, iris).
4. (Sometimes) **Somewhere you are** — location; **something you do** — behavioral biometrics.

**Multi-Factor Authentication (MFA)** = requiring factors from *two or more different categories*. Two passwords aren't MFA (same category); a password + a phone code is. MFA is one of the highest-impact defenses in existence — it stops the vast majority of credential-based attacks, because a stolen password alone is no longer enough.

**But not all MFA is equal:**
- **SMS codes** — better than nothing, but vulnerable to SIM-swapping and interception.
- **Authenticator apps (TOTP)** — time-based codes; solid, but **phishable** (a fake site relays the code in real time).
- **Push notifications** — convenient, but vulnerable to **MFA fatigue** (spamming prompts until a tired user approves).
- **FIDO2 / WebAuthn hardware security keys / passkeys** — the gold standard: *phishing-resistant* because the cryptographic proof is bound to the real domain and never leaves the device. This is where the industry is heading; know why passkeys are fundamentally stronger.

**Password reality:** despite decades of trying to kill them, passwords persist. Best practice has shifted: **long passphrases over complex short ones**, no forced periodic rotation (it causes weaker, predictable passwords), screening against known-breached-password lists, and MFA on top. Password *managers* (unique strong password per site) plus MFA is the practical defense you should personally use and recommend.

---

## 12.3 How authentication is attacked

Map each attack to the factor/mechanism it defeats:

- **Password guessing / brute force** — trying many passwords against one account. Mitigated by rate limiting, lockouts, strong passwords.
- **Password spraying** — trying *one* common password against *many* accounts (evades per-account lockouts). A top real-world technique, especially against enterprise/cloud logins.
- **Credential stuffing** — replaying username/password pairs leaked from *other* breaches, exploiting reuse. Why password reuse is so dangerous and why breach-list screening matters.
- **Phishing** — tricking the user into handing over credentials (and often the MFA code) on a fake site. Defeated most reliably by phishing-resistant MFA (FIDO2/passkeys).
- **MFA bypass** — real-time phishing (relay the code), MFA fatigue (push spam), SIM swap (SMS), or stealing session tokens *after* MFA (see next section).
- **Session attacks** — stealing the post-login session rather than the password (below).
- **Default/weak credentials** — devices/services left on `admin/admin`. Still rampant, especially IoT.

> **The pattern:** as passwords get MFA protection, attackers move "downstream" to steal the *session* that exists after successful authentication. Understanding this shift is key to modern defense.

---

## 12.4 Sessions and tokens: staying logged in (and getting hijacked)

HTTP is stateless, so after you log in, the server issues a **session identifier** (a session cookie, or a token like a JWT) that your browser sends with each request to prove "I already authenticated." That token is now a credential — and a target.

- **Session hijacking** — stealing the session token (via XSS, network sniffing on unencrypted connections, or malware) to impersonate the user *without their password or MFA*. This is why post-MFA session theft is such a big deal.
- **Session fixation** — forcing a known session ID onto a victim, then riding it after they log in.
- **Defenses:** HTTPS everywhere (so tokens aren't sniffed); cookie flags **`HttpOnly`** (JavaScript can't read it — blunts XSS theft), **`Secure`** (HTTPS only), **`SameSite`** (limits cross-site sending — blunts CSRF); short session lifetimes; regenerating the session ID on login; binding sessions to context; and revocation on logout. You'll test these in Chapter 16.

**JWTs as session tokens** (from Chapter 11): stateless and scalable, but *hard to revoke* (they're valid until they expire) and dangerous if the signature isn't verified or secrets are weak. Trade-offs matter.

---

## 12.5 SSO and federation: identity at scale

Enterprises don't want a separate password per app. **Single Sign-On (SSO)** lets a user authenticate once to an **Identity Provider (IdP)** and access many applications (Service Providers) without re-entering credentials. **Federation** extends this across organizational boundaries.

The protocols (you met the crypto side in Chapter 11):
- **SAML 2.0** — XML-based, dominant in enterprise SSO. The IdP sends the app a signed **assertion** vouching for the user.
- **OAuth 2.0** — *authorization delegation* ("let app X access my data at Y"). Issues **access tokens** with limited **scopes**. Not authentication on its own.
- **OpenID Connect (OIDC)** — authentication layered on OAuth 2.0; issues an **ID token** (a JWT) proving identity. This is modern "Sign in with Google/Microsoft."

**Why this matters for security:** SSO concentrates risk — compromise the IdP or a user's IdP session, and you get *many* applications at once ("keys to the kingdom"). Federation and OAuth misconfigurations (over-broad scopes, redirect-URI flaws, token leakage, consent phishing) are a rich and growing attack area, especially in cloud environments. Golden SAML (forging SAML assertions after compromising the IdP's signing key) is the cloud analog of the Golden Ticket from Chapter 6.

---

## 12.6 Authorization: what you're allowed to do

Once identity is established, authorization decides access. The models:

- **DAC (Discretionary Access Control)** — resource owners set permissions (classic file permissions — Chapter 5). Flexible, error-prone (users over-share).
- **MAC (Mandatory Access Control)** — the system enforces central policy that users can't override (military classification levels; SELinux/AppArmor). Rigid, strong.
- **RBAC (Role-Based Access Control)** — permissions attach to **roles**, users get roles. The enterprise standard: manageable, auditable. "Finance role can read finance data." Most systems you'll secure use RBAC.
- **ABAC (Attribute-Based Access Control)** — decisions based on **attributes** (user department + resource sensitivity + time + location + device posture). More granular and dynamic; underpins Zero Trust. "Allow if user.dept == resource.dept AND device.is_managed AND time.is_business_hours."
- **ReBAC (Relationship-Based)** — access based on relationships (Google Docs "shared with"); increasingly common in apps.

**Principle of least privilege (Chapter 10) is authorization's north star**: grant the minimum, review regularly, remove access when roles change (deprovisioning is a chronically neglected control — orphaned accounts and stale admin rights are everywhere).

---

## 12.7 Broken access control: the #1 web risk

This is authorization *failing*, and it's the most common serious web vulnerability. The main forms (you'll exploit these in Chapter 16):

- **IDOR (Insecure Direct Object Reference)** — the app uses a client-supplied identifier (`/invoice?id=1234`) to fetch a resource *without checking the requester owns it*. Change the ID → read someone else's data. Trivial to attempt, devastating, everywhere.
- **Missing function-level authorization** — the "admin" endpoint isn't linked in the UI for normal users, but it's not actually *protected* server-side. Guess/force the URL → admin actions. (Security through obscurity failing — Chapter 10.)
- **Privilege escalation** — horizontal (access another user's data at the same level) or vertical (gain higher privileges). The web analog of Chapter 5/6's OS privesc.
- **Forced browsing, parameter tampering, mass assignment** — manipulating requests to reach or modify things you shouldn't (e.g., adding `"isAdmin": true` to a profile-update request the developer didn't expect).

**The root cause is almost always the same:** authorization checks that are missing, done on the client side (trivially bypassed), or done once instead of on *every* access (violating complete mediation — Chapter 10). The fix: **enforce authorization server-side, on every request, based on the authenticated identity — never trust the client to tell you who it is or what it may do.**

---

## Common mistakes

- **Confusing authentication with authorization.** They fail and are attacked differently; keep them crisp.
- **Treating all MFA as equal.** SMS/push are phishable/fatigable; FIDO2/passkeys are phishing-resistant. Know the hierarchy.
- **Client-side authorization checks.** Hiding a button is not access control. Enforce on the server, every time.
- **Trusting client-supplied identity/role.** Never let the client assert who it is or what it's allowed to do (JWT `isAdmin` claims the server doesn't validate, hidden form fields, etc.).
- **Forgetting deprovisioning.** Stale accounts and leftover admin rights are a top real-world weakness.
- **Ignoring session security after login.** MFA is defeated by stealing the post-auth session.

---

## Labs

> **Lab 12.1 — Set up strong auth for yourself.** Adopt a password manager, generate unique passwords for your important accounts, and enable the strongest available MFA (prefer a hardware key/passkey where offered). This is both practice and basic professional hygiene — you can't credibly advise others while reusing passwords.

> **Lab 12.2 — Exploit IDOR (lab).** In DVWA or Juice Shop, find an IDOR: locate a request with an object ID, change it, and access another user's resource. Document the request, the fix (server-side ownership check), and how you'd detect the attack in logs. Portfolio-worthy.

> **Lab 12.3 — Session cookie inspection.** Log into a lab app, open browser dev tools, and inspect the session cookie. Check for `HttpOnly`, `Secure`, `SameSite`. Note which are missing and what attack each missing flag enables. Then read about stealing a cookie via XSS (Chapter 16).

> **Lab 12.4 — Map an OAuth/OIDC flow.** Using a "Sign in with Google/GitHub" on any app (or a diagram), trace the OIDC flow: redirect to IdP → authenticate → consent → token back to app. Identify where redirect-URI validation and scope limits matter. Write it up.

> **Lab 12.5 — Password spraying vs brute force (lab).** In your lab, simulate the difference: many passwords vs one account (brute force) and one common password vs many accounts (spraying). Observe which trips a lockout and which doesn't, and why spraying evades it. ⚠️ Your lab accounts only.

---

## References and further reading

- **OWASP Top 10 — A01: Broken Access Control** ([owasp.org/Top10](https://owasp.org/www-project-top-ten/)). Read this entry closely; it's the #1 risk and this chapter's core.
- **OWASP Authentication, Access Control, Session Management, and JWT Cheat Sheets** — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org). Practical do/don't guidance.
- **NIST SP 800-63B — Digital Identity Guidelines.** The authoritative modern guidance on authentication (why passphrases beat forced complexity, why not to force rotation, MFA guidance). Genuinely readable and influential.
- **Auth0 / Okta developer docs** — clear explanations of OAuth 2.0, OIDC, and SAML with diagrams.
- **"OAuth 2.0 Simplified" — Aaron Parecki (free online).** The clearest OAuth explainer.
- **PortSwigger Web Security Academy** — free labs on access control, authentication, JWT, and OAuth. The best hands-on practice; pairs with Chapter 16.
- **FIDO Alliance / passkeys.dev** — for why WebAuthn/passkeys are phishing-resistant.

---

## Self-check

1. Precisely distinguish authentication and authorization, and give an attack that targets each.
2. Why is a password + a security question *not* MFA, and why is a FIDO2 key stronger than an SMS code?
3. An attacker steals a user's session cookie after they've logged in with MFA. Why does MFA not save them, and which cookie flags reduce this risk?
4. What is IDOR, why is it so common, and what is the correct fix?
5. Why does SSO concentrate risk, and what's the cloud analog of a Kerberos Golden Ticket in a SAML world?

<details>
<summary>Answers</summary>

1. **Authentication** proves *who you are* (login); **authorization** determines *what you may do* once identified. An authentication attack (e.g., credential stuffing/phishing) makes the attacker *become* a user; an authorization attack (e.g., IDOR/broken access control) lets an authenticated user *do more than allowed*.
2. Both a password and a security question are "something you know" — the *same factor category* — so together they're single-factor. MFA requires factors from *different* categories. A **FIDO2 key** is stronger than **SMS** because it's phishing-resistant (the cryptographic response is bound to the real domain and never transmitted/relayable) and immune to SIM-swap/interception, whereas SMS codes can be phished in real time or intercepted.
3. Because the session token *is* proof of an already-completed authentication — possessing it impersonates the user without needing the password or MFA again. **`HttpOnly`** (blocks JavaScript/XSS theft), **`Secure`** (HTTPS-only, prevents sniffing), and **`SameSite`** (limits cross-site sending) reduce the risk, along with short lifetimes and session regeneration.
4. Insecure Direct Object Reference: the app fetches a resource using a client-supplied ID without verifying the requester owns it, so changing the ID exposes others' data. It's common because developers implement the "happy path" and forget per-object authorization. Fix: enforce **server-side ownership/authorization checks on every access**, based on the authenticated identity.
5. Because one authentication to the IdP grants access to *many* applications — compromising the IdP or a user's IdP session yields all of them at once. The analog is **Golden SAML**: after compromising the IdP's token-signing key, an attacker forges valid SAML assertions to impersonate anyone to any federated service — persistent, wide-reaching access, like a Golden Ticket in AD.

</details>

---

## What's next

You now understand who attackers are, how systems work, the principles of security, and the machinery of crypto and identity. The final piece of the security core is a shared *language* for describing what adversaries actually do — so red and blue teams can talk to each other and to threat intel. [Chapter 13](13-adversary-models-and-mitre-attack.md) teaches MITRE ATT&CK and the adversary models that structure the entire rest of this booklet.
