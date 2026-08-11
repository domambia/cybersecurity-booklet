# Chapter 14 · Reconnaissance and OSINT

> **Why this chapter matters:** Every attack — and every authorized penetration test — begins with reconnaissance. Before touching a target, professionals map it: what does it expose, who works there, what technology does it run, where are the soft spots? Recon determines everything that follows; a good tester spends serious time here because the vulnerabilities you find are only as good as the attack surface you discovered. This chapter starts the hands-on attack chain — legally, within scope.

> **By the end of this chapter you will:** perform passive and active reconnaissance, use OSINT to profile an organization, enumerate an attack surface (domains, subdomains, hosts, technologies, people), and understand how defenders reduce and monitor their own exposure.

> ⚠️ **Scope reminder (Chapter 3):** Passive recon of public information is generally low-risk, but *active* recon (scanning, probing) touches the target's systems and requires authorization. Practice active techniques only against your lab, permission-granting platforms, or systems you're authorized to test. When in doubt, don't.

---

## 14.1 The two modes of recon

- **Passive reconnaissance** — gathering information *without directly interacting* with the target's systems, using public/third-party sources. The target can't see you doing it. Low-risk, and where you start.
- **Active reconnaissance** — *directly interacting* with the target (pinging, scanning, connecting to services) to elicit information. More revealing, but it touches their systems and generates logs — so it needs authorization and it's detectable.

The professional flow: exhaust passive recon first (build a picture invisibly), then move to active recon within scope to confirm and expand. Chapter 15 goes deep on active scanning; this chapter emphasizes passive/OSINT and the overall attack-surface mapping mindset.

---

## 14.2 OSINT: open-source intelligence

**OSINT** is intelligence gathered from publicly available sources. It's astonishing how much of an organization's attack surface and human profile is exposed for free. Categories and sources:

**Organizational footprint:**
- **Domains and subdomains** — the company's web presence. Subdomains often expose forgotten dev/staging/admin systems (`dev.`, `vpn.`, `test.`, `mail.`). Tools/sources: `crt.sh` (Certificate Transparency logs reveal subdomains via issued certs — a goldmine), `subfinder`, `amass`, DNS enumeration.
- **IP ranges and ASNs** — what network blocks the org owns. WHOIS, `bgp.he.net`, regional registries.
- **DNS records** — `dig`, `nslookup`, `dnsdumpster`; MX (mail), TXT (SPF/DKIM — reveals mail infrastructure and security posture), NS records.
- **Technology stack** — what they run. `Wappalyzer`, `BuiltWith`, HTTP headers, `whatweb`. Knowing the CMS/framework/versions points you at known vulnerabilities.

**Exposed assets and data:**
- **Shodan / Censys** — search engines for *internet-connected devices and services*. Find an org's exposed servers, open ports, service banners, even exposed databases and industrial systems — all passively, because these engines already scanned the internet. Learn Shodan; it's a defining OSINT tool. (`org:"Target Corp"`, `port:3389`, product filters.)
- **Google dorking** — advanced search operators to find exposed files, login pages, and data: `site:target.com filetype:pdf`, `site:target.com inurl:admin`, `intitle:"index of"`. The **Google Hacking Database (GHDB)** catalogs powerful dorks.
- **Code and secrets leakage** — GitHub/GitLab searches for the org's name, hardcoded credentials, API keys, internal hostnames. Developers leak secrets constantly (Chapter 28). Tools: `trufflehog`, GitHub search.
- **Breach data / paste sites** — Have I Been Pwned (which emails/domains appeared in breaches), leaked credential dumps (relevant for credential stuffing — Chapter 12). Use ethically and within scope.

**People (for social engineering and password attacks):**
- **LinkedIn** — employee names, roles, tech stack (from job postings — "must know Splunk and Okta" tells you their tooling), org structure.
- **Email format discovery** — deduce `firstname.lastname@company.com` from a few known addresses → build a target list for phishing/spraying (`hunter.io`, patterns).
- **Social media** — personal details for pretexting, out-of-office info, physical security clues.
- **Job postings, press releases, conference talks** — reveal technologies, projects, and vendors.

> **The defender's flip (do this for your own org):** everything above is what an attacker sees of *you*. Blue teams and "attack surface management" programs continuously run this same OSINT against themselves to find and shut down forgotten exposure — leaked keys, stale subdomains, exposed services on Shodan. Recon is a two-way skill.

---

## 14.3 Attack surface mapping

The output of recon is a **map of the attack surface** — everything an attacker could target:
- External IPs, domains, subdomains, and the services on them.
- Web applications and APIs.
- Technologies and versions (→ known CVEs).
- People and email addresses (→ phishing/spraying).
- Third parties and supply chain (vendors, cloud providers, dependencies).
- Leaked data and secrets.

Keep this in a structured document (it feeds directly into scanning in Chapter 15 and into your pentest report in Chapter 19). A big attack surface with old, unpatched, forgotten assets is where real breaches start — the "shadow IT" server nobody remembered.

---

## 14.4 The reconnaissance workflow (worked)

For an authorized engagement against `target.example`:

1. **WHOIS & DNS** — who owns it, name servers, mail servers, IP ranges.
2. **Subdomain enumeration** — `crt.sh`, `subfinder`, `amass` → discover `app.`, `dev.`, `vpn.`, `api.` etc. Each is a new door.
3. **Certificate Transparency** — `crt.sh` reveals subdomains (even internal-sounding ones) from every TLS cert ever issued for the domain.
4. **Technology fingerprinting** — `whatweb`/Wappalyzer on each live host → CMS, frameworks, server software, versions.
5. **Shodan/Censys** — what's already known to be exposed, open ports, banners, without you scanning.
6. **Google dorking + GitHub** — exposed files, admin panels, leaked secrets.
7. **People/email** — LinkedIn + email pattern → target list (for social-engineering scope, if authorized).
8. **Consolidate** into the attack-surface map → prioritize targets for active scanning (Chapter 15).

Notice most of this is *passive* — you've built a rich picture and the target has no idea. That's the power (and the reason defenders must assume they're being profiled).

---

## 14.5 Social engineering (the human attack surface)

Recon on *people* feeds **social engineering** — manipulating humans rather than machines. It's the initial-access method behind an enormous share of real breaches (phishing especially). The main forms:
- **Phishing** (email), **spear phishing** (targeted), **whaling** (executives), **smishing** (SMS), **vishing** (voice), **pretexting** (a fabricated scenario), **baiting** (a malicious USB/download).
- Works by exploiting trust, authority, urgency, fear, and helpfulness — not technical flaws.

> ⚠️ **You will not practice social engineering against real people in this booklet.** It requires explicit authorization (and often is excluded from engagements). Here you learn the *concepts* — how recon enables it, and how to defend (awareness training, phishing-resistant MFA from Chapter 12, email authentication like SPF/DKIM/DMARC, reporting culture). Understanding the attacker's playbook is defensive knowledge; running it on unwitting people is not yours to do without authorization.

---

## Common mistakes

- **Rushing past recon to "the hacking."** Weak recon → missed attack surface → a shallow test. Pros spend real time here.
- **Doing active recon without authorization.** Passive is usually fine; scanning/probing is touching their systems — get permission.
- **Ignoring Certificate Transparency and Shodan.** Two of the highest-value passive sources; beginners overlook them.
- **Not organizing findings.** Recon that isn't captured in a structured map is wasted; you'll need it for scanning and reporting.
- **Forgetting the human surface.** Many breaches start with people, not ports.

---

## Labs

> **Lab 14.1 — OSINT your own footprint.** Run the passive recon workflow against *yourself* or a domain you own: WHOIS, `crt.sh` for subdomains, Shodan for any exposed services, Google dorks, Have I Been Pwned for your email, GitHub for any leaked secrets. Document what you find. You'll likely be surprised — and it's fully authorized because it's you.

> **Lab 14.2 — Certificate Transparency hunt.** Pick a large, well-known company and use `crt.sh` to enumerate its subdomains (this uses public CT logs — passive and legal). Note how many "internal-sounding" subdomains are visible. Write up why CT logs are an attack-surface risk and how defenders manage it. *(Passive OSINT only — do not scan or connect to their systems.)*

> **Lab 14.3 — Shodan exploration.** Create a free Shodan account. Explore (read-only) searches like exposed services by product/port. Understand what an attacker learns for free. Then read Shodan's own guidance on how orgs reduce their exposure.

> **Lab 14.4 — Recon a lab target.** Against your Metasploitable/lab box (authorized), practice *active* recon: DNS, `whatweb`, banner grabbing. Build the attack-surface map you'll scan in Chapter 15.

> **Lab 14.5 — TryHackMe recon rooms.** Complete TryHackMe's OSINT and reconnaissance rooms (search "OSINT", "Passive Reconnaissance", "Google Dorking"). Guided, safe, hands-on.

---

## References and further reading

- **OSINT Framework** — [osintframework.com](https://osintframework.com). A categorized directory of OSINT tools/sources. Your map of the OSINT landscape.
- **Michael Bazzell — *Open Source Intelligence Techniques*.** The definitive OSINT book, regularly updated. The professional reference.
- **Shodan** — [shodan.io](https://www.shodan.io) and its free "Shodan for Penetration Testers" material. Learn its query language.
- **crt.sh** — [crt.sh](https://crt.sh) — Certificate Transparency search. Try it this week.
- **Google Hacking Database (GHDB)** — on exploit-db.com. Catalog of powerful search dorks.
- **TCM Security — *Practical Ethical Hacking* course / OSINT modules** — excellent, affordable hands-on offensive training that starts with recon.
- **OWASP Amass** and **subfinder** docs — for subdomain enumeration.
- **Have I Been Pwned** — [haveibeenpwned.com](https://haveibeenpwned.com) — breach exposure checks.

---

## Self-check

1. Distinguish passive from active reconnaissance, and explain why the authorization requirement differs.
2. What is Certificate Transparency and why is `crt.sh` so valuable to both attackers and defenders?
3. What does Shodan let an attacker learn *without* scanning the target, and how?
4. How does OSINT on employees translate into a technical attack path?
5. Why do experienced testers spend disproportionate time on reconnaissance?

<details>
<summary>Answers</summary>

1. **Passive** recon gathers info from public/third-party sources without touching the target's systems (invisible, low-risk); **active** recon directly interacts with the target (scanning, probing), which touches their systems, generates logs, and is detectable — so it constitutes access that requires authorization, whereas reading public data generally does not.
2. Certificate Transparency is a public, append-only log of every TLS certificate issued. `crt.sh` searches it, revealing an organization's subdomains (including forgotten dev/staging/internal-named hosts) from their certs — a passive goldmine for mapping attack surface, and a reason defenders must inventory and monitor what certs expose.
3. Exposed hosts, open ports, running services and their banners/versions, and sometimes misconfigured databases or devices — because Shodan/Censys continuously scan the whole internet and index the results, so the attacker queries a pre-built database instead of scanning the target themselves (staying passive).
4. Employee names/roles/emails (LinkedIn, email-pattern discovery) build a target list for **phishing/spear phishing** (initial access) and **password spraying/credential stuffing** (Chapter 12); job posts reveal the tech stack to target; personal details enable pretexting. The human surface becomes the entry point.
5. Because the quality of everything downstream depends on the attack surface discovered — you can only exploit what you find. Thorough recon uncovers forgotten, unpatched, or misconfigured assets (where real breaches start) and points scanning/exploitation at the softest targets, making the whole engagement more effective.

</details>

---

## What's next

You've mapped the attack surface. Now you probe it directly to find the specific weaknesses. [Chapter 15](15-scanning-enumeration-vulnerability-assessment.md) covers active scanning, service enumeration, and vulnerability assessment — turning your map into a ranked list of ways in, with nmap and friends. This is where the lab you built in Chapter 9 starts earning its keep.
