# Chapter 26 · Cloud Security

> **Why this chapter matters:** Almost everything runs in the cloud now, and cloud security is one of the two most in-demand skills in 2026 (the other is AI security — Chapter 29). The cloud isn't "someone else's computer with the same old rules" — it changes *who* is responsible for what, makes **identity** the new perimeter, and turns misconfiguration into the dominant breach cause. Your CS background and the fundamentals from Parts 1–4 transfer, but you must relearn where the boundaries are. This chapter makes you cloud-literate and points you at the specialization.

> **By the end of this chapter you will:** understand the shared responsibility model, why cloud IAM is the #1 attack surface, the common cloud misconfigurations and attacks, cloud logging/detection, and how to secure and assess a cloud environment.

---

## 26.1 What actually changes in the cloud

The security *principles* (Parts 1–2) are unchanged — least privilege, defense in depth, CIA. What changes is the *terrain*:

- **You don't own the hardware or the bottom of the stack.** The provider does. This splits responsibility (below) and changes what you can inspect (no pulling a disk for forensics — Chapter 23's cloud-forensics challenge).
- **Everything is an API and software-defined.** Networks, storage, servers, permissions — all created/changed via API calls and config. This means misconfiguration is a *config* problem (fast to cause, fast to fix, and fully logged), and the **control plane** (the management API) becomes a prime target.
- **Identity is the perimeter.** There's no network edge to hide behind; a leaked credential or over-permissioned role *is* the breach. IAM replaces the firewall as the thing that matters most.
- **Elastic and ephemeral.** Resources spin up/down constantly; you can't rely on static inventories or long-lived host agents the same way.
- **Speed.** Developers provision infrastructure themselves, fast — which means security must be *automated and built-in* (Chapter 28), not a manual gate.

---

## 26.2 The shared responsibility model (get this exact)

The single most important cloud-security concept. Security responsibility is *split* between provider and customer, and the split *depends on the service model*:

```
                    IaaS        PaaS        SaaS
Data                YOU         YOU         YOU     ← always yours
Identity/access     YOU         YOU         YOU     ← always yours
Application         YOU         YOU        provider
OS/runtime          YOU       provider     provider
Virtualization      provider   provider    provider
Physical/hardware   provider   provider    provider
```

- **The provider secures the cloud** (hardware, virtualization, physical datacenters — "security *of* the cloud").
- **You secure what you put in it** (your data, your IAM configuration, your app, and — depending on model — the OS — "security *in* the cloud").
- **Two things are *always* yours no matter the model: your data and your identity/access management.** Most cloud breaches happen squarely in the customer's half — misconfigured storage, over-permissioned identities, leaked keys — *not* a provider failure. The provider's infrastructure is generally very secure; the customer's configuration usually isn't.

Misunderstanding this model is itself a root cause of breaches ("we thought AWS handled that"). Know exactly where the line falls for IaaS/PaaS/SaaS.

---

## 26.3 Cloud IAM: the #1 attack surface

If you learn one thing deeply about cloud security, make it **IAM (Identity and Access Management)**. In the cloud, IAM *is* the security boundary, and IAM misconfiguration is the dominant path to compromise.

Core concepts (names vary by provider — AWS/Azure/GCP — but the ideas are universal):
- **Identities** — users, groups, and — crucially — **roles** and **service accounts** (machine identities that workloads assume). Machine identities vastly outnumber humans in cloud and are often the weak link.
- **Policies/permissions** — what an identity can do, attached to identities or resources. AWS IAM policies, Azure RBAC roles, GCP IAM bindings.
- **Roles and assumption** — temporary credentials via role assumption (AWS STS, etc.) — powerful and, if misconfigured, a privilege-escalation path.
- **Federation** — connecting your corporate IdP (Chapter 12) to the cloud; misconfigurations here (e.g., trust policies) enable attacks.

**IAM attacks and the core problem — privilege escalation via misconfigured permissions:**
- **Over-permissioned identities** — the #1 issue. An identity with `*:*` (admin) or far more than it needs. When compromised (leaked key, SSRF stealing its token — Chapter 16), the blast radius is enormous. **Least privilege** (Chapter 10) is the entire game here, and it's chronically violated because broad permissions are *easier*.
- **Privilege escalation paths** — a chain where an identity with permission to, say, create roles or modify policies or pass a role to a service can escalate to admin. (Analogous to BloodHound paths in AD — Chapter 18; tools like **PMapper**, **Cloudsplaining**, and **ScoutSuite** find these cloud IAM paths.)
- **Leaked credentials** — access keys hardcoded in code/Git (Chapter 28), in public S3 buckets, or exposed via SSRF. A leaked long-lived key is a direct login. This is why key rotation, short-lived credentials, and secret scanning matter enormously.
- **The instance metadata service (IMDS)** — a special internal endpoint that hands cloud credentials to a workload. **SSRF (Chapter 16) that reaches IMDS steals those credentials** — one of the most important cloud attack chains. (IMDSv2 and blocking IMDS access mitigate it.)

---

## 26.4 Common cloud misconfigurations (where breaches actually come from)

Beyond IAM, the recurring culprits — memorize these, they're most of real cloud breaches and most of what a cloud assessment checks:
- **Publicly exposed storage** — world-readable object storage (S3 buckets, Azure Blobs, GCS) leaking data. A perennial headline breach.
- **Overly permissive network rules** — security groups/firewall rules open to `0.0.0.0/0` on sensitive ports (databases, RDP, SSH). Exposes services directly (recall Shodan — Chapter 14).
- **Exposed/unauthenticated services** — databases and admin panels reachable from the internet with weak/no auth.
- **Unencrypted data** — storage or databases without encryption at rest/in transit (Chapter 11).
- **Disabled or unmonitored logging** — no CloudTrail/audit logging, so attacks are invisible (Chapter 20).
- **Secrets in code/config/environment** — keys and tokens where they shouldn't be (Chapter 28).
- **Public snapshots/AMIs, misconfigured serverless functions, over-broad service accounts.**

**CSPM (Cloud Security Posture Management)** tools continuously scan for exactly these misconfigurations against benchmarks (CIS, provider best practices). Knowing the misconfigurations *and* that CSPM automates finding them is core cloud-defender literacy.

---

## 26.5 Cloud logging, detection, and defense

Defense (Part 4) applied to cloud:
- **Audit logs are everything** — **AWS CloudTrail**, **Azure Activity/Monitor logs**, **GCP Audit Logs** record every control-plane API call: who did what, when, from where. This is your primary detection source (Chapter 20). Enable it, centralize it, alert on it (e.g., root account usage, IAM policy changes, disabled logging, unusual regions).
- **Cloud-native security services** — **GuardDuty** / **Security Hub** (AWS), **Microsoft Defender for Cloud** / **Sentinel** (Azure), **Security Command Center** (GCP) provide managed detection, posture management, and alerting. Know the major ones by name.
- **Cloud detections** — impossible travel, credential use from anomalous locations/IPs, IAM privilege escalation, mass data access/exfiltration, resource creation for cryptomining (a common post-compromise action), disabling of security controls.
- **Defensive architecture** — least-privilege IAM, short-lived credentials, encryption everywhere, network segmentation (VPCs, private subnets, no public exposure by default), secrets managers (not env vars), MFA on all human accounts (especially root/global admin), and **infrastructure-as-code with security scanning** (Chapter 28) so the secure config is the default and drift is caught.

---

## 26.6 Getting hands-on and specializing

Cloud security is best learned by *doing* in a real cloud account:
- **Provider free tiers** (AWS/Azure/GCP) — build, break, and secure. ⚠️ **Set billing alerts immediately** — a leaked key or a runaway resource can cost real money (attackers hijack cloud accounts to mine crypto; you don't want the bill).
- **Deliberately vulnerable cloud labs** — **flaws.cloud**, **CloudGoat** (AWS), **AzureGoat**, **SadCloud**, **TerraGoat** — teach cloud attack/defense hands-on, safely.
- **Multi-cloud reality** — most orgs use more than one provider; concepts transfer, but go *deep* on one first (AWS has the most jobs/material; Azure is strong in enterprise).
- **Certifications** (Chapter 33) — provider-specific security certs (AWS Security Specialty, Azure AZ-500) and vendor-neutral CCSP are where the market rewards cloud security skills. This is a top-paid, high-demand lane, often reached after 1–2 years' general experience.

> **The through-line:** cloud security is Parts 1–4's principles re-anchored on **identity and configuration** instead of hosts and network edges. Least privilege, defense in depth, visibility, and detection all still rule — but IAM is the new perimeter, misconfiguration is the new vulnerability, and audit logs are the new evidence. Your fundamentals transfer; the terrain is what you relearn.

---

## Common mistakes

- **Assuming the provider secures your data/config.** The shared responsibility model puts data and IAM *always* on you; most breaches are in the customer's half.
- **Over-permissioning identities for convenience.** `*:*` is the cloud's cardinal sin. Least privilege, always.
- **Hardcoding/leaking credentials.** Keys in Git or public buckets are direct logins. Use secret managers and short-lived credentials; scan for secrets.
- **Ignoring the metadata service / SSRF chain.** SSRF-to-IMDS credential theft is a top cloud attack; understand and mitigate it.
- **Not enabling/centralizing audit logging.** No CloudTrail = invisible attacks. Turn it on first.
- **No billing alerts on lab/free-tier accounts.** A hijacked account mining crypto can cost thousands. Set alerts on day one.
- **Learning cloud security only theoretically.** Build and break in a real account and vulnerable-cloud labs.

---

## Labs

> **Lab 26.1 — Set up safely.** Create a cloud free-tier account (start with AWS). **Immediately set a billing alert/budget.** Enable audit logging (CloudTrail). Enable MFA on the root/admin account. You've just done the three most important real-world first steps.

> **Lab 26.2 — IAM least privilege.** Create an IAM user/role with a deliberately over-broad policy, then tighten it to least privilege for a specific task. Use an analyzer (IAM Access Analyzer / Cloudsplaining) to find excess permissions. Write up the before/after and why over-permissioning is dangerous.

> **Lab 26.3 — Find and fix misconfigurations.** Intentionally create a public storage bucket and an over-open security group, then detect them (with a CSPM tool like **ScoutSuite** or **Prowler**, free) and remediate. Document the findings as if in an assessment report (Chapter 19).

> **Lab 26.4 — Attack a vulnerable cloud lab.** Complete **flaws.cloud** (free, browser-based) and/or **CloudGoat** scenarios. These walk you through real cloud attack chains (leaked keys, IAM privesc, SSRF-to-IMDS). Write up each attack path and its defense.

> **Lab 26.5 — Cloud detection.** In your account, perform a "suspicious" action (create an admin user, disable logging, access from a VPN in another region), then find it in CloudTrail and, if enabled, GuardDuty. Build an alert for it. This is cloud blue-team work.

> **Lab 26.6 — Infrastructure as code.** Write a small Terraform config that provisions a resource securely (encrypted, least-privilege, no public access), then scan it with a free IaC scanner (**Checkov**, **tfsec**). Fix what it flags. Bridges to Chapter 28 and is a strong portfolio piece.

---

## References and further reading

- **AWS/Azure/GCP well-architected & security documentation** — the authoritative sources. AWS Well-Architected Framework (Security Pillar) is excellent and free.
- **flaws.cloud & flaws2.cloud (Scott Piper)** — free, brilliant hands-on AWS security lessons. Start here.
- **CloudGoat (Rhino Security Labs), AzureGoat, SadCloud** — deliberately vulnerable environments for attack/defense practice.
- **ScoutSuite, Prowler, PMapper, Cloudsplaining** — free cloud auditing/IAM-analysis tools. Learn at least one CSPM tool.
- **The "AWS/Azure/GCP Customer Security" and CIS Benchmarks** — the checklists CSPM tools automate.
- **HackTricks Cloud** ([cloud.hacktricks.xyz](https://cloud.hacktricks.xyz)) — practical cloud attack reference.
- **"Hacking the Cloud" ([hackingthe.cloud](https://hackingthe.cloud))** — offensive cloud techniques catalog.
- **rhinosecuritylabs / SummitRoute blogs & the "Cloud Security" reading lists** — deep practitioner content.
- **Certifications:** AWS Certified Security – Specialty, Microsoft SC-900/AZ-500, ISC2 CCSP (Chapter 33).

---

## Self-check

1. State the shared responsibility model and the two things that are *always* the customer's responsibility regardless of service model.
2. Why is identity "the new perimeter" in the cloud, and what is the #1 IAM misconfiguration?
3. Describe the SSRF-to-metadata attack chain and why it's so dangerous in cloud.
4. Name four common cloud misconfigurations that lead to breaches, and the tool category that finds them.
5. Why must you set a billing alert on a lab cloud account before anything else?

<details>
<summary>Answers</summary>

1. The provider secures the underlying cloud (hardware, virtualization, physical) — "security *of* the cloud" — while the customer secures what they put in it — "security *in* the cloud" — with the split shifting by service model (more falls on the customer in IaaS, less in SaaS). **Always the customer's, in every model: their data and their identity/access management (IAM).**
2. Because there's no network edge to hide behind — resources are reached via APIs and authenticated by identity, so a leaked credential or over-permissioned role *is* the breach. The #1 misconfiguration is **over-permissioned identities** (e.g., wildcard `*:*`/admin permissions) that create a huge blast radius when compromised, violating least privilege.
3. A web app vulnerable to **SSRF** (Chapter 16) is made to request the cloud **instance metadata service (IMDS)** — an internal endpoint that returns the workload's cloud credentials. The attacker thereby steals those credentials and uses them (with whatever permissions the role has) to access the cloud environment. It's dangerous because it turns a common web bug into cloud credential theft; mitigations include IMDSv2 and blocking IMDS access.
4. Any four: publicly exposed storage buckets; overly permissive network/security-group rules (`0.0.0.0/0` on sensitive ports); exposed/unauthenticated databases or admin panels; unencrypted data; disabled/unmonitored audit logging; secrets in code/config. The tool category that finds them is **CSPM (Cloud Security Posture Management)** (e.g., ScoutSuite, Prowler).
5. Because a leaked key or misconfiguration can let attackers hijack the account (commonly to mine cryptocurrency) or a runaway resource can accrue large charges — a billing alert/budget caps the financial risk and warns you of compromise early. It's the first real-world safeguard for any cloud account.

</details>

---

## What's next

Cloud workloads increasingly run in containers. [Chapter 27](27-containers-and-kubernetes-security.md) covers securing containers and Kubernetes — the technology that packages and orchestrates modern applications, with its own attack surface, and another high-demand skill that builds directly on this chapter and your Linux foundations.
