# Chapter 27 · Containers and Kubernetes Security

> **Why this chapter matters:** Modern applications are packaged in **containers** and orchestrated by **Kubernetes**. If you work in cloud, DevSecOps, or AppSec, you *will* meet them, and they bring a whole new attack surface — image vulnerabilities, container escapes, and Kubernetes's famously complex, insecure-by-default configuration. This chapter builds on your Linux foundations (Chapter 5) and cloud knowledge (Chapter 26) to make you container-literate — a genuinely differentiating skill, because many security people still find this area intimidating and skip it.

> **By the end of this chapter you will:** understand what containers actually are (and aren't), the container and Kubernetes attack surface, common misconfigurations and escapes, and how to secure the build-ship-run lifecycle.

---

## 27.1 What a container actually is

A **container** packages an application with its dependencies into a single, portable unit that runs consistently anywhere. Crucially — and this is the security foundation — **a container is not a VM.** It's a set of Linux processes *isolated by kernel features*, sharing the host's kernel:
- **Namespaces** — isolate what a process can *see* (its own process tree, network, mounts, users).
- **Cgroups** — limit what a process can *use* (CPU, memory).
- **Capabilities / seccomp / AppArmor/SELinux** — restrict what a process can *do* (which syscalls, which privileges).

Because containers **share the host kernel** (unlike VMs, which have their own), the isolation is weaker: a kernel vulnerability or a misconfiguration can let a process **escape** the container onto the host — the defining container-security risk. This is why "a container is a security boundary" is only *conditionally* true, and why running untrusted workloads needs extra isolation. Your Chapter 4 (kernel/privilege) and Chapter 5 (Linux) knowledge is exactly what makes this comprehensible.

**Docker** is the common container runtime/tooling; **images** are the templates (built from a **Dockerfile**), **registries** store them, **containers** are running instances.

---

## 27.2 The container attack surface

Think across the lifecycle — **build, ship, run**:

**Image (build) risks:**
- **Vulnerable base images / dependencies** — an image built on an outdated base or with vulnerable libraries (Chapter 28's supply chain). Most images have known CVEs. **Image scanning** (Trivy, Grype, Clair) is essential.
- **Secrets baked into images** — API keys, passwords in image layers (and layers persist in history even if "removed" — like Git, Chapter 28). A top real-world leak.
- **Malicious or typosquatted images** from public registries — supply-chain risk (Chapter 28).
- **Bloated images** — more software = more attack surface. Minimal/distroless images are a hardening win.

**Registry (ship) risks:** unauthenticated/public registries, unsigned images (no integrity — sign images and verify signatures), tampered images.

**Runtime (run) risks — where escapes happen:**
- **Privileged containers** — `--privileged` gives the container near-total host access; a trivial escape. Never run privileged unless truly required.
- **Running as root** — containers running as UID 0 (root) — if escaped, root on the host. Run as non-root.
- **Dangerous mounts** — mounting the **Docker socket** (`/var/run/docker.sock`) or the host filesystem into a container = instant host control. A classic escape.
- **Excessive capabilities** — added Linux capabilities (e.g., `CAP_SYS_ADMIN`) that enable escapes. Drop all, add back only what's needed.
- **Kernel exploits** — because the kernel is shared, a kernel privesc (Chapter 17) escapes the container.

---

## 27.3 Kubernetes: orchestration and its (large) attack surface

**Kubernetes (K8s)** orchestrates containers at scale — scheduling, scaling, networking, and healing thousands of containers across many hosts. It's powerful and *notoriously complex and insecure by default*, making it a rich target. Core components (know the vocabulary):
- **Control plane:** **API server** (the central control point — the crown jewel; if compromised, you own the cluster), **etcd** (the datastore — holds all config *and secrets*; must be encrypted and locked down), scheduler, controller manager.
- **Nodes** run **pods** (groups of containers) via the **kubelet**.
- **Objects:** pods, deployments, services, namespaces, **secrets**, **service accounts** (pod identities), **RBAC** roles.

**The Kubernetes attack surface:**
- **Exposed/unauthenticated API server or kubelet** — direct cluster control. (Shodan — Chapter 14 — finds exposed clusters.)
- **Over-permissive RBAC** — the K8s version of Chapter 26's IAM problem; a pod's service account with cluster-admin rights is catastrophic if the pod is compromised (via, say, a web vuln in the app — Chapter 16). Least privilege again.
- **Weak pod security** — privileged pods, host namespace sharing (`hostPID`, `hostNetwork`), host path mounts → node compromise → cluster compromise.
- **Secrets management** — K8s Secrets are only base64-encoded (not encrypted!) by default and in etcd; anyone with access reads them. External secret managers and encryption-at-rest are needed.
- **Network policy** — flat pod networking by default (any pod can talk to any pod) → easy lateral movement (Chapter 7's segmentation, missing). Network policies must be defined.
- **Lateral movement path** — the enterprise-AD analogy (Chapter 18): compromise a pod → abuse its service account/RBAC → reach the API server → cluster takeover. **BloodHound-style tools exist for K8s** (e.g., mapping RBAC attack paths).

---

## 27.4 Securing the container and Kubernetes lifecycle

Defense, applied across build-ship-run:

**Build:** scan images for vulnerabilities and secrets in CI (Chapter 28); use minimal/distroless base images; pin and verify dependencies; don't bake in secrets; sign images.

**Ship:** private registries with authentication; image signing + verification (Sigstore/cosign); admission control that rejects unsigned or non-compliant images.

**Run (hardening):**
- **Never privileged; run as non-root; drop all capabilities** (add back minimally); read-only root filesystem where possible.
- **Pod Security Standards / admission controllers** (e.g., OPA/Gatekeeper, Kyverno) to *enforce* policy — policy-as-code that rejects insecure pods (Chapter 28's shift-left, applied to K8s).
- **Least-privilege RBAC** for service accounts; no default cluster-admin.
- **Network policies** to segment pod traffic (default-deny).
- **Encrypt etcd / use external secret managers**; restrict API server exposure; enable audit logging.
- **Runtime security** — tools like **Falco** detect anomalous container behavior at runtime (a container spawning a shell, unexpected network connections — behavioral detection, Chapter 21, applied to containers).

**Benchmarks & tools:** the **CIS Kubernetes/Docker Benchmarks** are the checklists; **kube-bench** (checks cluster against CIS), **kube-hunter** (finds K8s vulnerabilities), **Trivy** (image + config + cluster scanning), **Falco** (runtime detection). Learning one or two of these makes you immediately useful.

> **The through-line:** container/K8s security is your Linux, cloud, least-privilege, segmentation, supply-chain, and detection knowledge — re-applied to a new abstraction. Namespaces/cgroups are Chapter 4/5; RBAC is Chapter 26's IAM; image scanning is Chapter 28's supply chain; Falco is Chapter 21's behavioral detection; network policy is Chapter 7's segmentation. Nothing here is truly new — it's your fundamentals in container clothing. That's the good news: you're more ready for this than it looks.

---

## Common mistakes

- **Treating containers as VMs / strong security boundaries.** They share the host kernel; isolation is weaker and escapes are the defining risk.
- **Running privileged / as root / mounting the Docker socket.** Each is a near-trivial host takeover. Avoid.
- **Ignoring image scanning and baked-in secrets.** Most images carry known CVEs and sometimes secrets in persistent layers.
- **Over-permissive Kubernetes RBAC.** A pod with cluster-admin is one web bug away from cluster takeover. Least privilege for service accounts.
- **Assuming K8s Secrets are encrypted.** They're base64 by default; treat them as exposed without extra measures.
- **Flat pod networking.** No network policy means free lateral movement. Default-deny and segment.
- **Skipping this topic as "too complex."** It's your existing fundamentals reapplied; that's a differentiator, not a wall.

---

## Labs

> **Lab 27.1 — Container basics and isolation.** Install Docker in your lab. Run a container, then explore its isolation: from inside, look at its process tree, network, and mounts; from the host, find the container's processes (proving they're just host processes in namespaces). Write up how namespaces/cgroups create the illusion of a separate machine.

> **Lab 27.2 — Scan an image.** Build a Dockerfile using an old base image, then scan it with **Trivy** or **Grype**. Review the CVEs and secrets found. Rebuild with a minimal/updated base and rescan. Document the reduction — a real DevSecOps task.

> **Lab 27.3 — Container escape (lab).** In an isolated lab, run a deliberately misconfigured container (`--privileged` or with the Docker socket mounted) and perform a container escape to the host. Then fix the misconfiguration. Understand *why* it worked. ⚠️ Lab only; snapshot first.

> **Lab 27.4 — Vulnerable Kubernetes.** Stand up a local cluster (minikube/kind) and work through **kube-hunter** and a deliberately vulnerable K8s environment (e.g., **Kubernetes Goat**). Practice the attack paths: exposed dashboard, over-permissive RBAC, pod-to-node escape. Write up each.

> **Lab 27.5 — Harden a cluster.** Run **kube-bench** (CIS benchmark) against your cluster, then remediate findings. Add a **network policy** (default-deny) and an admission policy (Kyverno/OPA) that rejects privileged pods. Deploy **Falco** and trigger a runtime alert (spawn a shell in a pod). This is container blue-team work and a strong portfolio project.

---

## References and further reading

- **Liz Rice — *Container Security*.** The definitive, approachable book on how container isolation works and how it fails. **The** book for this chapter.
- **Kubernetes documentation — Security section** and the **NSA/CISA Kubernetes Hardening Guide** (free, excellent, practical).
- **CIS Docker & Kubernetes Benchmarks** — the hardening checklists (automated by kube-bench).
- **Kubernetes Goat** ([madhuakula.com/kubernetes-goat](https://madhuakula.com/kubernetes-goat/)) — deliberately vulnerable K8s for hands-on learning.
- **Trivy (Aqua), Grype, kube-bench, kube-hunter, Falco, Kyverno/OPA Gatekeeper** — the core free tools; learn Trivy and Falco first.
- **HackTricks — Cloud/Container/Kubernetes Pentesting** — practical offensive reference.
- **Sigstore/cosign** — for image signing and supply-chain integrity (Chapter 28).
- **"Hacking Kubernetes" — Andrew Martin & Michael Hausenblas** — offense-and-defense deep dive.

---

## Self-check

1. Why is a container a weaker security boundary than a VM, and what is the defining container-security risk?
2. Name three container runtime misconfigurations that lead to host compromise.
3. Why is the Kubernetes API server the crown jewel, and how might a compromised application pod lead to cluster takeover?
4. Why is it dangerous to assume Kubernetes Secrets are secure by default?
5. Explain how container/K8s security is really your earlier fundamentals reapplied — give three concrete mappings.

<details>
<summary>Answers</summary>

1. Because containers **share the host kernel** (isolated only by namespaces/cgroups/capabilities), whereas VMs have their own kernel and hypervisor isolation. So a kernel vulnerability or a misconfiguration can let a containerized process **escape to the host** — container escape is the defining risk, and containers are only conditionally a security boundary.
2. Any three: running **`--privileged`**, running **as root** (UID 0), mounting the **Docker socket** (`/var/run/docker.sock`) or host filesystem into the container, and granting **excessive Linux capabilities** (e.g., `CAP_SYS_ADMIN`). Each can grant effective host control from inside the container.
3. The API server is the central control point for the whole cluster — compromising it means controlling all workloads, nodes, and secrets. A compromised app pod (e.g., via a web vuln) can abuse its mounted **service account token** and over-permissive **RBAC** to talk to the API server with excessive rights, escalating from one pod to cluster-admin — the K8s analog of AD lateral movement to Domain Admin.
4. Because K8s Secrets are only **base64-encoded, not encrypted**, by default and are stored in etcd; anyone with API/etcd access (or over-broad RBAC) can read them in plaintext. Without encryption-at-rest for etcd, tight RBAC, and/or an external secrets manager, they're effectively exposed.
5. Examples: namespaces/cgroups/capabilities = **Chapter 4/5 Linux kernel & privilege** concepts; Kubernetes RBAC and service accounts = **Chapter 26 cloud IAM / least privilege**; image scanning for vulnerable dependencies = **Chapter 28 software supply chain**; Falco runtime detection = **Chapter 21 behavioral detection**; network policies = **Chapter 7 segmentation**. The abstraction is new; the security principles are ones you already know.

</details>

---

## What's next

Containers and cloud are provisioned and deployed through automated pipelines — which are themselves a security-critical (and attacked) system. [Chapter 28](28-devsecops-and-software-supply-chain.md) covers DevSecOps and software supply-chain security: building security *into* how software is made and shipped. It's the purest expression of the security-engineering path and where a CS graduate's skills are most directly rewarded.
