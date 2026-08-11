# Appendix F · Lab Build Files

> **Companion to [Chapter 9](../part-1-foundations/09-building-your-home-lab.md).** Chapter 9 is the *why and architecture*; this is the *concrete build* — the machines, network settings, and step-by-step configs. ⚠️ The golden rule (Chapter 9): **everything isolated from your home network and the internet by default; host-only networking; snapshot before every experiment.** Verify isolation before you attack anything.

> Download URLs and versions change — search the official source rather than trusting a stale link. Build in *stages* (Chapter 9), not all at once.

---

## F.1 Hardware & hypervisor recap

- **Minimum:** 16 GB RAM, 4-core CPU with VT-x/AMD-V enabled in BIOS/UEFI, 512 GB SSD. **Comfortable:** 32 GB, 8-core, 1 TB NVMe.
- **Hypervisor:** VirtualBox (free, start here) or VMware Workstation (free personal). Proxmox VE on a dedicated box once serious. Apple Silicon: UTM or VMware Fusion (note ARM limits which x86 target VMs run — use ARM Kali + web/cloud labs).
- **No capable machine?** Do Parts 2–4 on TryHackMe/HTB/PortSwigger (browser-based) and a minimal 2-VM local lab. Don't let hardware block you.

---

## F.2 The network design (do this first)

Create an **isolated host-only (or internal) network** in your hypervisor and put all lab VMs on it.

- **VirtualBox:** File → Host Network Manager → Create a host-only network (e.g., `192.168.56.0/24`, DHCP off or a small range). Set each VM's adapter to **Host-Only Adapter** on that network. For *temporary* internet (updates), switch a VM to **NAT**, update, then switch back.
- **VMware:** Virtual Network Editor → add a **Host-only** network (e.g., `VMnet1`). Attach VMs to it.

> ⚠️ **Never use Bridged** for vulnerable/malware VMs — it puts them on your real LAN. **Verify isolation:** from Kali, confirm you can reach a target IP and **cannot** reach the internet (`ping 8.8.8.8` should fail) or your home router.

Suggested addressing (static, host-only `192.168.56.0/24`):

| Host | IP | Role |
|---|---|---|
| Kali (attacker) | .10 | Offensive toolkit |
| Metasploitable 2 | .20 | Vulnerable Linux target |
| Web targets (DVWA/Juice Shop) | .21 | Web range (Docker host) |
| Windows Server (DC) | .30 | Domain Controller (`lab.local`) |
| Windows 10/11 client | .31 | Domain-joined workstation |
| Blue-team (Security Onion/Wazuh) | .40 | SIEM/monitoring |

---

## F.3 Stage 1 — Attacker + first target (build now)

**Kali Linux (attacker):**
1. Download the official **Kali VirtualBox/VMware image** (kali.org/get-kali → prebuilt VM). Import it.
2. RAM 2–4 GB, adapter → Host-Only.
3. Update once (temporary NAT): `sudo apt update && sudo apt full-upgrade -y`, then back to host-only.
4. **Snapshot → `kali-clean`.**

**Metasploitable 2 (target):**
1. Download **Metasploitable 2** (Rapid7/SourceForge). Import the VM.
2. Adapter → Host-Only. (It's *deliberately* vulnerable — **never** give it internet/bridged.)
3. Default creds `msfadmin/msfadmin`. **Snapshot → `metasploitable-clean`.**

**Verify:** from Kali, `nmap -sn 192.168.56.0/24` finds Metasploitable; `ping 8.8.8.8` fails (isolated). ✅ You have a working offensive lab (Chapter 9 Lab 9.1–9.2).

---

## F.4 Stage 2 — Web targets (for Chapter 16)

Easiest via Docker on a small Linux VM (or on Kali):

```bash
# DVWA
docker run -d -p 80:80 vulnerables/web-dvwa

# OWASP Juice Shop
docker run -d -p 3000:3000 bkimminich/juice-shop
```
Browse from Kali to `http://<host>:80` (DVWA) and `:3000` (Juice Shop). Also grab **VulnHub** images for variety. **Snapshot** the web host.

---

## F.5 Stage 3 — Windows / Active Directory lab (for Chapters 6, 18)

Use **free evaluation ISOs** (Microsoft Evaluation Center: Windows Server + Windows 10/11 Enterprise eval, ~180 days).

**Domain Controller (Windows Server):**
1. Install Windows Server (eval), static IP `.30`, host-only.
2. Install AD DS role → promote to a Domain Controller → create domain `lab.local`.
3. Create OUs, a few **users**, a **group** (e.g., "IT"), and a **service account** with an SPN and a deliberately weak-ish password (for Kerberoasting practice, Chapter 18):
   ```powershell
   # (example) set an SPN on a service account
   setspn -a HTTP/web01.lab.local svc_web
   ```
4. **Snapshot → `dc-clean`.**

**Windows client(s):**
1. Install Windows 10/11 (eval), static IP `.31`, host-only, DNS pointing at the DC (`.30`).
2. **Join it to `lab.local`.**
3. **Snapshot → `win10-clean`.**

> This is the environment you'll attack in Chapter 18 and detect in Part 4. Building it *is* the exercise.

---

## F.6 Stage 4 — Blue-team monitoring (for Part 4)

Pick one:
- **Security Onion** (all-in-one IDS + SIEM + log management) — download the ISO, install to a VM (needs more RAM — 8–16 GB ideally), host-only + a monitoring interface. Follow its setup wizard.
- **Wazuh + Elastic** (lighter) — deploy the Wazuh manager (Docker or VM), install Wazuh agents on the Windows/Linux hosts.
- Either way: **install Sysmon** on all Windows hosts with a community config:
  ```powershell
  # Download Sysmon (Sysinternals) + a config (e.g., SwiftOnSecurity / Olaf)
  sysmon.exe -accepteula -i sysmonconfig.xml
  ```
- Ship Windows Security + Sysmon + PowerShell logs and Linux auth logs into the SIEM (Chapter 20).

**Now you can attack from Kali and watch detections light up on the blue side** — the attack-and-detect loop (Chapters 20–21, portfolio flagship, Chapter 32).

---

## F.7 Cloud lab notes (for Chapters 26–28)

- Create **AWS/Azure/GCP free-tier** accounts. **⚠️ First actions: set a billing alert/budget, enable MFA on root/admin, enable audit logging (CloudTrail/Activity/Audit Logs).**
- Deliberately vulnerable cloud labs (browser or your account): **flaws.cloud / flaws2.cloud** (free), **CloudGoat**, **AzureGoat**, **SadCloud**, **Kubernetes Goat**, **TerraGoat**.
- For containers/K8s: Docker locally; **minikube** or **kind** for a local Kubernetes cluster.

---

## F.8 Documentation template (your portfolio starts here — Chapter 32)

Create a Git repo `home-lab/` with:

```
home-lab/
├── README.md            # overview, isolation model, how to reproduce
├── network-diagram.png  # your topology (adapt Chapter 9's diagram)
├── hosts.md             # each VM: OS, IP, role, creds (lab-only!), purpose
├── build-notes.md       # steps you took, gotchas, decisions
└── validation.md        # how you verified isolation
```

Example `hosts.md` row: `192.168.56.30 — Windows Server 2022 (eval) — Domain Controller for lab.local — target for Ch18 AD attacks, source for Part 4 detections.`

> A documented lab is a portfolio piece (Chapter 32). Write it as you build — the writing *is* revision.

---

## F.9 Safety checklist (run before every offensive/malware lab)

- [ ] Target VMs on **host-only/internal** network (not bridged).
- [ ] Confirmed lab **cannot reach the internet or my home LAN**.
- [ ] **Snapshot taken** right before the experiment.
- [ ] Malware samples handled only in an **isolated, snapshotted** VM with no bridged networking (Chapter 24).
- [ ] I'm only attacking **my own lab / authorized targets** (Chapter 3).

> If any box is unchecked, stop and fix it before proceeding. The lab exists so you can break things safely — keep it that way.

---

## F.10 Automation & "level-up" (optional, later)

Once your manual lab works, study how pros automate lab provisioning:
- **DetectionLab** (Chris Long) — automated Windows AD + monitoring lab (Vagrant/Ansible).
- **Splunk Attack Range**, **Security Datasets / Mordor** — pre-built attack telemetry (Chapter 20).
- **Ludus / GOAD (Game of Active Directory)** — automated vulnerable AD environments for deeper Chapter 18 practice.
- Build your own with **Vagrant + Ansible/Terraform** — which is itself an infrastructure-as-code portfolio piece (Chapters 26, 28).
