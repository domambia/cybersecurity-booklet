# Chapter 11 · Applied Cryptography

> **Why this chapter matters:** Cryptography provides two legs of the CIA triad — confidentiality and integrity — plus authentication. But it's where beginners either freeze (scared by math) or, worse, act confident and cause disasters (rolling their own crypto, misusing libraries). You don't need to be a cryptographer. You need to *use* crypto correctly and *recognize misuse in code and configs* — which is exactly what real jobs require. This chapter is applied, not academic: the math is minimized; the judgment is maximized.

> **By the end of this chapter you will:** know when to use symmetric vs asymmetric crypto, hash vs encrypt, how PKI/TLS establish trust, how passwords should be stored, what JWTs/OAuth are, and — most valuably — how to spot the common crypto failures that appear in real code reviews and pentests.

---

## 11.1 The one rule above all others

> ⚠️ **Never invent your own cryptographic algorithm or protocol, and never implement crypto primitives yourself unless you are a specialist doing it deliberately.** Use well-vetted, widely-reviewed libraries (`libsodium`, your language's `cryptography` package, the platform's native crypto) at a high level. "Rolling your own crypto" is the single most reliable way to build something that looks secure and is trivially broken. Even experts get primitives wrong; that's why we rely on *public, peer-reviewed* algorithms and battle-tested implementations (recall *open design* from Chapter 10).

Your job is choosing the right tool and using it correctly — not building the tool.

---

## 11.2 What crypto gives you (and what it doesn't)

Cryptography provides:
- **Confidentiality** — only authorized parties can read the data (encryption).
- **Integrity** — unauthorized changes are detectable (hashing, MACs, signatures).
- **Authentication** — you can verify who you're talking to (signatures, certificates).
- **Non-repudiation** — a party can't deny an action (digital signatures).

Crypto does **not** provide: availability (encrypted data can still be deleted or DoS'd), protection against a compromised endpoint (if the attacker owns your machine, they read data before you encrypt it), or security against implementation bugs and misuse (the usual failure mode). Crypto is a tool, not a magic shield — most real-world "crypto failures" are actually *usage* failures.

---

## 11.3 Symmetric encryption — the workhorse

**Symmetric** = one shared secret key encrypts and decrypts. Fast; used for bulk data.

- **Standard algorithm: AES** (Advanced Encryption Standard), typically AES-128 or AES-256. Trusted, ubiquitous, hardware-accelerated.
- **The catch is the *mode*.** A block cipher like AES needs a mode of operation to handle data longer than one block, and mode choice is where people fail:
  - **ECB (Electronic Codebook) — never use.** It encrypts identical plaintext blocks to identical ciphertext blocks, leaking patterns. The famous "ECB penguin" image (an encrypted picture where the penguin is still clearly visible) is the canonical demonstration. Seeing ECB in code is an instant finding.
  - **CBC** — chains blocks; needs a random **IV (initialization vector)**. Historically common but vulnerable to padding-oracle attacks if misused, and provides no integrity by itself.
  - **GCM (Galois/Counter Mode) — prefer this.** It's **authenticated encryption (AEAD)**: it provides confidentiality *and* integrity together, so tampering is detected. Modern default.

**Key insight:** encryption alone (without authentication) doesn't stop tampering — an attacker can flip bits in ciphertext to alter the plaintext in predictable ways. That's why **AEAD modes like GCM (or ChaCha20-Poly1305)** are the right default: they bundle integrity in. If you see raw CBC or ECB without a separate MAC, be suspicious.

**The hard problem of symmetric crypto: key distribution.** How do two parties who've never met share a secret key without an eavesdropper catching it? That's what asymmetric crypto solves.

---

## 11.4 Asymmetric (public-key) encryption — solving key distribution

**Asymmetric** = a mathematically linked **key pair**: a **public key** (shareable with everyone) and a **private key** (kept secret). What one key does, only the other undoes.

Two core uses:
1. **Encryption for confidentiality:** anyone encrypts *to* you with your **public** key; only your **private** key decrypts. (Now someone can send you a secret without a pre-shared key.)
2. **Digital signatures for authentication + integrity + non-repudiation:** you sign with your **private** key; anyone verifies with your **public** key. A valid signature proves *you* (holder of the private key) produced it and the data wasn't altered.

- **Algorithms: RSA** (older, based on factoring large numbers) and **ECC (Elliptic Curve)** (newer, smaller keys for equal strength — increasingly preferred).
- Asymmetric crypto is *slow*, so in practice it's used to **exchange a symmetric key** (or sign a hash), then symmetric crypto does the bulk work. That hybrid is exactly what TLS does.

**Diffie–Hellman key exchange** — a clever method that lets two parties derive a shared secret over a public channel without ever transmitting it, safe from a passive eavesdropper. It (in its elliptic-curve form, ECDHE) underpins modern TLS and provides **forward secrecy** (a stolen long-term key can't decrypt past sessions). Know the concept; you don't need the math.

---

## 11.5 Hashing — fingerprints, not encryption

A **cryptographic hash** takes any input and produces a fixed-size "fingerprint" (digest). Properties: **deterministic** (same input → same output), **one-way** (can't reverse the digest to the input), **collision-resistant** (hard to find two inputs with the same digest), and **avalanche** (a tiny input change utterly changes the output).

- **Use hashes for integrity** (verify a download matches its published hash), **fingerprinting** (dedupe, indicators of compromise — malware is often identified by its hash), and as a building block in signatures and MACs.
- **Standard algorithms: SHA-256 / SHA-3.** **MD5 and SHA-1 are broken** (collisions found) — seeing them used for security is a finding, though MD5 still appears as a *non-security* checksum.
- **Hashing ≠ encryption.** Hashing is one-way and keyless; encryption is reversible with a key. Confusing them is a classic beginner error.

**HMAC** — a hash combined with a secret key to provide **integrity + authentication** of a message (proves it came from someone holding the key and wasn't altered). Used everywhere (API request signing, cookies, JWTs).

---

## 11.6 Password storage — the highest-stakes everyday crypto

This is where crypto knowledge most directly saves (or sinks) a company, and it's a top interview question.

**Never store passwords in plaintext. Never "encrypt" them (reversible = a single key compromise reveals all). Never hash them with a fast hash like MD5/SHA-256 alone.**

Why not a fast hash? Because attackers who steal the hash database run *billions* of guesses per second on GPUs, and precomputed **rainbow tables** reverse common fast hashes instantly. You must make guessing *slow and unique*:

- **Salt** — a unique random value per password, stored alongside the hash. Defeats rainbow tables and ensures two users with the same password get different hashes.
- **Slow, purpose-built password hashing functions** — designed to be deliberately expensive (CPU and/or memory):
  - **Argon2** (current best-practice, memory-hard — resists GPU/ASIC cracking) — prefer this.
  - **bcrypt** (long-trusted, has a tunable work factor).
  - **scrypt** and **PBKDF2** (acceptable, PBKDF2 where compliance requires it).
- The work factor is tuned so a login takes a fraction of a second (fine for one user) but mass-cracking becomes infeasible.

**So: password storage = salt + Argon2/bcrypt with an appropriate work factor.** If you see fast unsalted hashes for passwords, that's a critical finding. You'll attack weak password hashing in Chapter 17 (cracking) — this is the defense.

---

## 11.7 PKI and how the web establishes trust

**PKI (Public Key Infrastructure)** is the system of **Certificate Authorities (CAs)**, **certificates**, and trust that lets you talk securely to a server you've never met — the backbone of HTTPS.

The chain of trust:
1. A server proves its identity with a **certificate** — a document binding its public key to its domain name, **digitally signed by a CA**.
2. Your browser/OS ships with a list of **trusted root CAs**.
3. When you connect, your browser verifies the server's certificate chains up to a trusted root, is valid (not expired/revoked), and matches the domain. If so → trust established, TLS session begins (Chapter 7's handshake).

**Common PKI/TLS failures you'll encounter (and flag):**
- **Expired or self-signed certificates** (browser warnings people click through — dangerous habit).
- **Disabled certificate validation** in application code ("verify=False" to "make it work") — silently removes the authentication guarantee and enables man-in-the-middle. A frequent, serious code-review finding.
- **Weak cipher suites / old TLS versions** (SSLv3, TLS 1.0/1.1 — deprecated).
- **Trusting user-supplied certificates or a compromised CA.**

**Let's Encrypt** made free, automated certificates ubiquitous — know it exists; it's why HTTPS is now near-universal.

---

## 11.8 Tokens: JWT, OAuth, OIDC, SAML (applied crypto in web auth)

You'll meet these constantly (they overlap with Chapter 12). Crypto-relevant essentials:

- **JWT (JSON Web Token)** — a signed (sometimes encrypted) token carrying claims (who you are, what you can do), used for stateless authentication. Structure: `header.payload.signature`, base64-encoded (so the payload is *readable* — never put secrets in it unencrypted; it's signed, not hidden). **Classic JWT flaws:** accepting the `alg: none` algorithm (no signature!), weak signing secrets (crackable HMAC keys), and confusing signature *verification* with mere *decoding*. These are common findings — you'll test for them in Chapter 16.
- **OAuth 2.0** — a *delegation* framework ("let this app access my Google data without giving it my password"). Not authentication by itself.
- **OpenID Connect (OIDC)** — authentication built *on top of* OAuth 2.0 (the "Sign in with Google" you actually experience).
- **SAML** — XML-based SSO, common in enterprises. XML signature-wrapping attacks are its classic crypto pitfall.

You don't implement these from scratch (rule #1 again) — you use libraries and configure them correctly, and you learn to spot the misconfigurations.

---

## 11.9 What's on the horizon: post-quantum

Large quantum computers would break RSA and ECC (via Shor's algorithm), threatening today's asymmetric crypto. The response is **post-quantum cryptography (PQC)** — new algorithms (NIST has standardized the first ones) being rolled out now, plus "harvest now, decrypt later" concern (adversaries storing encrypted data today to decrypt once quantum arrives). You don't need depth here yet — just know the term, that migration is underway, and that symmetric crypto/hashes are far less affected (larger keys suffice).

---

## Common mistakes (these ARE the findings you'll report)

- **Rolling your own crypto.** The cardinal sin. Use vetted libraries.
- **ECB mode, or encryption without authentication.** Prefer AEAD (GCM/ChaCha20-Poly1305).
- **Fast/unsalted hashes for passwords.** Use salted Argon2/bcrypt.
- **Confusing hashing with encryption, or encoding (base64) with encryption.** Base64 is *not* security (Chapter 4).
- **Disabling certificate validation.** Silently destroys TLS's authentication guarantee.
- **Putting secrets in a JWT payload** (it's readable) or accepting `alg: none`.
- **Reusing IVs/nonces.** Catastrophic in many modes — nonces must be unique.
- **Hardcoding keys/secrets in source code** (they end up in Git history — Chapter 28).

---

## Labs

> **Lab 11.1 — See the ECB penguin.** Using CyberChef or a small script, encrypt a simple bitmap image with AES-ECB and view the result — the image structure survives. Then do it with CBC/GCM and see it become noise. Write up *why* ECB leaks patterns.

> **Lab 11.2 — Hash and salt.** In Python's `hashlib`/`bcrypt` or the `cryptography` library: (1) hash "password123" with SHA-256 and look it up in an online hash database — instant reversal, proving fast hashes fail. (2) Hash it with bcrypt and a salt; hash it again — note the different output each time. Explain the difference in your notes.

> **Lab 11.3 — Crack weak password hashes.** Take a small list of SHA-256 hashes of weak passwords and crack them with `hashcat` or `john` using a wordlist (rockyou.txt). Time it. Then bcrypt the same passwords and observe how much slower cracking becomes. This connects directly to Chapter 17. ⚠️ Only crack hashes you generated yourself.

> **Lab 11.4 — Inspect a real certificate.** In your browser, view the certificate of a site you use. Trace its chain to the root CA. Note the expiry, the cipher suite, and the TLS version (use SSL Labs' free test for the full picture). Write up what each element guarantees.

> **Lab 11.5 — Decode and tamper a JWT.** Get a sample JWT (jwt.io). Decode its three parts — note the payload is *readable*. Read about the `alg: none` and weak-secret attacks. Try cracking a weak-HMAC JWT's secret with a tool. Explain why "I decoded it" is not "I verified it."

---

## References and further reading

- **Jean-Philippe Aumasson — *Serious Cryptography* (2nd ed.).** The best modern applied-crypto book for practitioners — rigorous but readable, focused on using crypto correctly. Your primary reference for this chapter.
- **Niels Ferguson, Bruce Schneier, Tadayoshi Kohno — *Cryptography Engineering*.** Excellent on the engineering pitfalls (why things fail in practice).
- **Cryptopals Challenges** — [cryptopals.com](https://cryptopals.com). Free, hands-on crypto exercises that teach by *breaking* real misuses (ECB, CBC padding oracles, weak MACs). The single best way to internalize crypto failures. Do at least Set 1–2.
- **OWASP Cryptographic Storage Cheat Sheet** and **Password Storage Cheat Sheet** — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org). Practical, current, exactly what to do.
- **SSL Labs (ssllabs.com/ssltest)** — free TLS configuration analyzer; learn by testing real sites.
- **jwt.io** — decode/experiment with JWTs.
- **NIST PQC** — [csrc.nist.gov/projects/post-quantum-cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography) — for the post-quantum picture when you're ready.

---

## Self-check

1. When do you use symmetric vs asymmetric encryption, and how does TLS use both together?
2. Why is hashing passwords with SHA-256 alone insufficient, and what should you use instead?
3. Explain the difference between encryption, hashing, and encoding (base64), with one use of each.
4. A developer sets `verify=False` on HTTPS requests "to fix a certificate error." What guarantee did they just destroy, and what attack does it enable?
5. Someone says "the JWT is safe because it's encoded." What's wrong with that statement?

<details>
<summary>Answers</summary>

1. **Symmetric** (AES) for fast bulk data encryption when both sides share a key; **asymmetric** (RSA/ECC) to solve key distribution and for signatures/authentication when parties haven't pre-shared a secret. TLS uses asymmetric crypto (certificates + key exchange like ECDHE) to authenticate the server and establish a shared **symmetric** session key, then uses fast symmetric encryption (e.g., AES-GCM) for the actual data — a hybrid.
2. SHA-256 is *fast*, so attackers can brute-force stolen hashes at billions/sec and use rainbow tables. Passwords need a **unique salt** plus a **slow, memory-hard** function — **Argon2** (preferred), **bcrypt**, or **scrypt/PBKDF2** — tuned so each guess is expensive.
3. **Encryption** is reversible with a key (confidentiality) — e.g., AES-GCM protecting data at rest. **Hashing** is one-way and keyless (integrity/fingerprinting) — e.g., verifying a file download. **Encoding** (base64) is a reversible, keyless representation change with **no security** — e.g., safely transporting binary in text. Confusing them is a common error.
4. It destroys **authentication** — the client no longer verifies the server's certificate, so it can't tell the real server from an impostor. This enables a **man-in-the-middle** attack: an attacker can present any certificate, decrypt/modify traffic, and the client accepts it.
5. Base64 encoding is not secrecy — anyone can decode a JWT and read its payload. A JWT's protection comes from its **signature** (integrity/authenticity), which must be *verified*, not from encoding. Secrets shouldn't be placed in an unencrypted JWT payload, and "decoding" a token is not the same as "verifying" it.

</details>

---

## What's next

Crypto lets systems prove *who* someone is and protect data. Next, the broader machinery of identity: how systems authenticate users, authorize actions, and manage access at scale — and how that machinery is attacked. [Chapter 12](12-identity-authentication-authorization.md) covers the domain (IAM) that is simultaneously the #1 web vulnerability class and the #1 cloud attack surface.
