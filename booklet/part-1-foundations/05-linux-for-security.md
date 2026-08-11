# Chapter 5 · Linux for Security Practitioners

> **Why this chapter matters:** Linux runs most of the internet's servers, most security tooling, and your attack platform (Kali). Whether you go red or blue, you'll live in a Linux shell. "I can use Linux" is not enough — you need *fluency*: the ability to investigate, script, and defend a system without stopping to Google basics. This chapter builds that fluency and frames every command through what it means for an attacker and a defender.

> **By the end of this chapter you will:** navigate and manipulate Linux confidently from the shell, understand the permission model and its abuse (SUID, sudo), read the logs and process state that reveal an intrusion, and know where attackers hide and where defenders look.

---

## 5.1 Why Linux, and how to get fluent

Fluency comes from *living* in the shell, not reading about it. Set up a Linux VM (Ubuntu for a normal system, Kali for the attacker toolkit — both in [Chapter 9](09-building-your-home-lab.md)) and do everything in it: take notes, browse files, break and fix things. The commands below are your working vocabulary; the goal is to run them without thinking.

**The mental model:** in Linux, *everything is a file* — regular files, directories, devices (`/dev/sda`), running-process info (`/proc/<pid>`), even network sockets are represented in the filesystem. This uniformity is why the shell is so powerful: the same tools (`cat`, `grep`, `ls`) work on all of it.

---

## 5.2 The filesystem hierarchy (and where security lives)

```
/          root of everything
├── bin, /usr/bin   → executable programs (commands)
├── sbin            → system/admin binaries
├── etc             → system configuration  ← attackers read this; defenders audit it
│   ├── passwd      → user accounts (world-readable!)
│   ├── shadow      → password hashes (root-only) ← crack target
│   ├── sudoers     → who can run sudo ← privesc target
│   └── crontab, cron.d → scheduled jobs ← persistence hides here
├── home            → user home directories (.ssh keys, .bash_history live here)
├── root            → root user's home
├── var
│   ├── log         → LOGS ← where blue team investigates
│   ├── www         → web content (often)
│   └── tmp, /tmp   → world-writable ← attackers stage tools here
├── proc            → live kernel/process info (virtual)
├── dev             → devices
└── opt, /srv       → optional/served software
```

Commit the security-relevant ones: `/etc/passwd` and `/etc/shadow` (accounts and hashes), `/etc/sudoers` and `/etc/cron*` (privilege and persistence), `/var/log` (evidence), `/tmp` and `/dev/shm` (where attackers drop tools because they're world-writable), and `~/.ssh`, `~/.bash_history` (credentials and command history).

---

## 5.3 Shell fluency: the commands you must own

Group these by purpose and *use them until they're reflexes*.

**Navigation & inspection:** `pwd`, `ls -la` (the `-a` shows hidden `.` files where secrets hide), `cd`, `tree`, `file` (identifies a file's real type regardless of extension — attackers rename things), `stat` (all timestamps and metadata).

**Reading files:** `cat`, `less`, `head`, `tail -f` (follow a log live), `strings` (extract readable text from a binary — first move in malware triage).

**Searching — the highest-leverage skills:**
- `grep -r "password" /etc/` — recursively search file *contents*. Add `-i` (case-insensitive), `-n` (line numbers), `-E` (regex).
- `find / -name "*.conf" 2>/dev/null` — search by name/attributes. `find` is also a privesc workhorse: `find / -perm -4000 2>/dev/null` lists all SUID binaries (a privilege-escalation hunting technique you'll use in Chapter 17).
- `awk` and `sed` — field extraction and stream editing. `awk -F: '{print $1}' /etc/passwd` prints every username. These turn raw logs into answers.
- Pipes and redirection: `cat access.log | grep 404 | awk '{print $1}' | sort | uniq -c | sort -rn` — "count the IPs generating the most 404s," a real threat-hunting one-liner. Understand each stage.

**Processes & system state:** `ps aux` (all processes), `top`/`htop` (live), `ss -tulpn` (listening ports and owning processes — replaces old `netstat`), `lsof` (open files/sockets), `kill`, `systemctl status <svc>`, `journalctl -u <svc>` (systemd logs).

**Users & permissions:** `id`, `whoami`, `who`/`w` (who's logged in), `sudo -l` (what can I run as root? — first thing you check after landing on a box), `su`.

**Networking:** `ip a` (interfaces/addresses), `ip route`, `ping`, `curl`/`wget` (fetch URLs — also how attackers download payloads), `dig`/`nslookup` (DNS), `ssh`, `scp`.

> ✅ **Milestone:** you can chain `grep`/`awk`/`sort`/`uniq` into a one-liner that answers a real question about a log file, without looking up the syntax.

---

## 5.4 The permission model — and how it's abused

Linux permissions are the foundation of Linux security and a rich source of both defense and attack.

**Reading permissions:** `ls -l` shows `-rwxr-xr--`:
- First char: type (`-` file, `d` directory, `l` symlink).
- Then three triplets: **owner**, **group**, **others**, each `rwx` (read/write/execute).
- `rwxr-xr--` = owner can read/write/execute, group can read/execute, others can only read.

**Numeric (octal):** r=4, w=2, x=1. So `rwx`=7, `r-x`=5, `r--`=4. `chmod 755 file` = `rwxr-xr-x`. `chmod 644` = `rw-r--r--`. Learn to convert instantly.

**Ownership:** `chown user:group file`. Every file has an owner and a group.

**The special bits — where privilege escalation lives:**
- **SUID (Set User ID), octal 4000** — a program with SUID runs with the privileges of its *owner*, not the user who launched it. `passwd` is SUID-root because changing your password requires writing to root-owned `/etc/shadow`. **The abuse:** if a SUID-root program can be tricked into running arbitrary commands (or if a dangerous binary like `find`, `vim`, or `bash` is wrongly SUID-root), any user can become root. This is one of the most common Linux privilege-escalation paths. The reference site **GTFOBins** catalogs exactly which binaries can be abused this way.
- **SGID (2000)** — same idea with group.
- **Sticky bit (1000)** — on a directory like `/tmp`, lets users delete only their *own* files even though the directory is world-writable.

**sudo** — controlled privilege delegation via `/etc/sudoers`. `sudo -l` shows what the current user may run as root. Misconfigured sudoers (e.g., a user allowed to run `vim` or `less` as root — both of which can spawn a shell) is another classic privesc. Again: GTFOBins.

> **Both-sides framing:** the *attacker* enumerates SUID binaries and sudo rights to escalate; the *defender* audits and minimizes them (least privilege — Chapter 10). The same knowledge, used from two chairs. This is the purple-team mindset in miniature.

---

## 5.5 Users, authentication, and where credentials live

- **`/etc/passwd`** — one line per user: `username:x:UID:GID:comment:home:shell`. World-readable. The `x` means the password hash is elsewhere (in shadow). UID 0 = root (any account with UID 0 is root-equivalent — a backdoor trick is adding one).
- **`/etc/shadow`** — the password hashes, readable only by root. Format: `username:$id$salt$hash:...`. The `$id$` says the algorithm (`$6$` = SHA-512, `$y$` = yescrypt/Argon-family). **If an attacker can read shadow, they can crack passwords offline** (Chapter 17). Defenders ensure strong hashing and that shadow stays root-only.
- **SSH keys** in `~/.ssh/` — `id_rsa` (private — a stolen one is a login), `authorized_keys` (public keys allowed to log in — attackers *add* their key here for persistence). `known_hosts` records servers you've connected to (useful forensic trail of lateral movement).
- **`~/.bash_history`** — a record of typed commands. Gold for both forensics (what did the attacker do?) and attackers (what secrets did the admin type?).
- **PAM (Pluggable Authentication Modules)** — the framework that actually decides authentication. Advanced, but know it exists; malicious PAM modules are a stealthy persistence technique.

---

## 5.6 Logs: where the blue team investigates

If you go defensive, you'll live here. Learn what each log holds.

| Log (Debian/Ubuntu paths) | Contains | Why it matters |
|---|---|---|
| `/var/log/auth.log` (or `secure` on RHEL) | Authentication: logins, sudo, SSH | Brute-force attempts, successful/failed logins, privilege use |
| `/var/log/syslog` / `messages` | General system messages | Broad system activity |
| `journalctl` (systemd) | Unified structured logs | `journalctl -u ssh --since "1 hour ago"` |
| `/var/log/`app logs (nginx, apache, etc.) | Web/app activity | Web attacks, access patterns |
| `~/.bash_history`, `last`, `lastb` | Command history, login/failed-login records | Attacker activity, intrusion timeline |
| `auditd` (`/var/log/audit/`) | Kernel-level audit events | Deep forensic detail; must be configured |

**A real defensive one-liner:** find IPs with the most failed SSH logins (a brute-force signature):
```bash
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head
```
You'll build exactly this instinct — turn a log into an answer — in Part 4. Here you're just learning where the evidence lives.

> ⚠️ **Attackers delete or tamper with logs** to hide. This is why production environments ship logs *off the host* to a central SIEM the attacker can't reach (Chapter 20). A log only on the compromised box is a log you can't trust.

---

## 5.7 Persistence: where attackers hide to survive a reboot

Once an attacker is on a Linux box, they want to stay. Knowing their hiding spots is essential for both hunting them (blue) and establishing footholds (red). Common Linux persistence locations:

- **Cron jobs** — `/etc/crontab`, `/etc/cron.d/`, `/etc/cron.{hourly,daily}/`, per-user `crontab -l`. A job that re-runs a payload every minute.
- **Systemd services and timers** — a malicious `.service` unit that restarts the implant.
- **Shell startup files** — `~/.bashrc`, `~/.profile`, `/etc/profile.d/` — code that runs on every login.
- **SSH `authorized_keys`** — the attacker's key added for silent re-entry.
- **New/UID-0 accounts**, modified PAM, kernel modules/rootkits (advanced).

Your future threat-hunting checklist starts here: enumerate cron, systemd units, startup files, authorized_keys, and unexpected UID-0 accounts. Attackers know these spots; so must you.

---

## 5.8 Packages, services, and hardening basics

- **Package managers** — `apt` (Debian/Ubuntu), `dnf`/`yum` (RHEL/Fedora), `pacman` (Arch). Keeping packages patched is one of the highest-impact defenses in existence — most breaches exploit *known, patched* vulnerabilities.
- **Services** — every listening service is attack surface. `ss -tulpn` shows what's listening. Hardening principle: **turn off what you don't need**.
- **Firewalls** — `ufw` (simple), `iptables`/`nftables` (powerful). Default-deny inbound is the baseline.
- **SELinux / AppArmor** — mandatory access control that confines processes even if they're compromised. Conceptually important (defense in depth at the OS level); you'll meet them again in hardening work.

Basic hardening checklist (you'll formalize this in Chapter 30's CIS Controls): patch regularly, remove unused services and packages, enforce least privilege, use key-based SSH and disable root SSH login, enable a host firewall, centralize logging, and audit SUID binaries and sudo rights.

---

## Common mistakes

- **Reading commands instead of running them.** Fluency is muscle memory. Type everything.
- **Memorizing flags instead of understanding tools.** Know *what* `find` does and you can look up flags; memorizing flags without the concept is fragile.
- **Ignoring the log side because "I want to be red team."** Attackers who don't understand logging get caught. Defenders who don't understand attacks miss it. Learn both.
- **Being scared to break the VM.** That's what snapshots are for. Break it, fix it, learn.
- **Skipping `sudo -l`, SUID enumeration, and `.bash_history`** — these three checks are among the most productive things you can do on any Linux box, offensively or defensively.

---

## Labs

> **Lab 5.1 — OverTheWire: Bandit.** Go to overthewire.org/wargames/bandit and complete levels 0–20 (or as far as you can). This is the single best free way to build shell fluency — each level teaches a real command by forcing you to use it. Do not skip. Write up the technique each level taught. *(This is authorized practice — it's the platform's purpose.)*

> **Lab 5.2 — Investigate a system.** On your Ubuntu VM: list all users from `/etc/passwd`; check what you can run with `sudo -l`; find all SUID binaries with `find / -perm -4000 2>/dev/null`; look at the last 20 authentication events. Write up what a defender would want to know from each.

> **Lab 5.3 — Log analysis one-liner.** Generate some failed SSH logins (try to SSH into your VM with a wrong password a few times), then write a `grep | awk | sort | uniq -c | sort -rn` pipeline that counts failed attempts per source. Explain each stage of your pipeline in your notes.

> **Lab 5.4 — Hunt for persistence.** On your VM, manually plant a "persistence" mechanism (add a harmless cron job that writes to a file, and add an SSH key to `authorized_keys`). Then, pretending you're an incident responder who's never seen this box, find both. Document your hunting methodology — this becomes a reusable checklist.

> **Lab 5.5 — GTFOBins privesc.** Read gtfobins.github.io. Pick one binary (e.g. `find` or `vim`). On your VM, make it SUID-root deliberately (`sudo chmod u+s /usr/bin/find`), then use the GTFOBins technique to get a root shell. Then *remove* the SUID bit and write up why this was exploitable and how a defender would catch/prevent it. ⚠️ Lab VM only — this deliberately weakens the box.

---

## References and further reading

- **William Shotts — *The Linux Command Line* (free at linuxcommand.org).** The best free, beginner-to-competent Linux shell book. Work through it alongside Bandit.
- **OverTheWire Bandit** — [overthewire.org/wargames/bandit](https://overthewire.org/wargames/bandit). Your primary lab for this chapter.
- **GTFOBins** — [gtfobins.github.io](https://gtfobins.github.io). The definitive reference for Unix binary privilege-escalation. Bookmark permanently.
- **Evan Sultanik / general — *Linux Basics for Hackers* by OccupyTheWeb.** Linux specifically for security work, using Kali. Great second book.
- **`man` pages** — `man <command>` is the authoritative reference and it's already on your machine. Get comfortable reading them; `man 5 passwd` documents file formats, `man 2 <syscall>` documents syscalls.
- **Michael Kerrisk — *The Linux Programming Interface*.** The deep reference for when you need to truly understand a syscall or subsystem. Career-long resource.

---

## Self-check

1. What does the SUID bit do, and why is a SUID-root `find` binary dangerous?
2. Where are Linux password hashes stored, who can read that file, and why does that permission matter for an attacker?
3. Write (from memory) a pipeline that finds the top 5 IPs generating failed SSH logins in `auth.log`.
4. Name four places a Linux attacker commonly establishes persistence.
5. Why do production systems ship logs to a central server instead of keeping them only locally?

<details>
<summary>Answers</summary>

1. SUID makes a program run with the privileges of its **owner** rather than the invoking user. A SUID-root `find` is dangerous because `find` can execute arbitrary commands (`find . -exec /bin/sh \;`), so any user can spawn a **root shell** through it — full privilege escalation.
2. In **`/etc/shadow`**, readable only by **root**. It matters because if an attacker can read shadow (e.g., after escalating or via a file-read bug), they can crack the hashes **offline** at leisure, potentially recovering plaintext passwords and reusing them elsewhere.
3. `grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -5` (field index may vary by log format — the concept is: extract the IP, count, sort descending).
4. Any four: cron jobs (`/etc/cron*`, user crontabs), systemd services/timers, shell startup files (`~/.bashrc`, `/etc/profile.d/`), SSH `authorized_keys`, new or UID-0 accounts, malicious PAM modules / kernel rootkits.
5. Because an attacker who compromises a host can **delete or tamper with local logs** to hide. Shipping logs to a central SIEM the attacker can't reach preserves trustworthy evidence and enables correlation across many hosts.

</details>

---

## What's next

You're fluent in the system that runs the internet's servers and your own tools. But most *enterprise* attacks happen on Windows, in Active Directory. [Chapter 6](06-windows-and-active-directory.md) takes you into the environment where the majority of real corporate breaches unfold — and which most beginners wrongly skip.
