# Chapter 7 · Networking Deep Dive

> **Why this chapter matters:** Networking is the single highest-leverage foundation in security. Every attack traverses a network; every defense watches one. Beginners skip it because it's less glamorous than "hacking," then hit a ceiling they can't explain. You have the CS-degree theory — this chapter makes it *operational*: you'll be able to explain every packet on the wire and name the attack that lives at each layer. Give this chapter more time than any other in Part 1.

> **By the end of this chapter you will:** map protocols to layers and the attacks to each layer, subnet by hand, explain the TCP handshake and TLS from a capture, understand the security weaknesses of DNS/DHCP/ARP/HTTP, and read traffic in Wireshark.

---

## 7.1 The two models: OSI and TCP/IP

The **OSI model** (7 layers) is the teaching/reference model; the **TCP/IP model** (4 layers) is what the internet actually uses. Learn to map any protocol, tool, and attack to a layer — it's how professionals reason ("this is a Layer 2 attack," "that's a Layer 7 issue").

| OSI Layer | Name | Examples | Attacks that live here |
|---|---|---|---|
| 7 | Application | HTTP, DNS, SMTP, SSH | Web attacks, injection, phishing, most of what you'll do |
| 6 | Presentation | TLS/SSL, encoding | TLS downgrade, cert issues |
| 5 | Session | Sessions | Session hijacking |
| 4 | Transport | **TCP, UDP** | Port scanning, SYN floods, session attacks |
| 3 | Network | **IP, ICMP, routing** | IP spoofing, routing attacks, ICMP tunneling |
| 2 | Data Link | **Ethernet, ARP, MAC, VLAN** | ARP spoofing, MAC flooding, VLAN hopping |
| 1 | Physical | Cables, radio | Physical tampering, wiretapping |

TCP/IP collapses these into Link, Internet, Transport, Application — but the OSI vocabulary is what people speak. Know both.

> **The mental habit:** for any attack you learn, ask "what layer?" It organizes everything. ARP spoofing (L2) works differently from a SYN flood (L4) from SQL injection (L7), and the defenses differ accordingly.

---

## 7.2 IP addressing and subnetting (do this by hand)

**IPv4** — 32 bits, written as four octets: `192.168.1.10`. Split into a **network** portion and a **host** portion by the **subnet mask**.

**CIDR notation** — `192.168.1.0/24` means the first 24 bits are the network, leaving 8 bits (256 addresses, 254 usable hosts) for hosts. `/16` = 65,536 addresses. `/8` = ~16 million. You must be able to compute these without a calculator — it comes up in scoping engagements, reading firewall rules, and scanning.

**Worked example:** `192.168.10.0/26`
- `/26` = 26 network bits, 6 host bits → 2⁶ = 64 addresses per subnet.
- Subnets: `.0`, `.64`, `.128`, `.192`.
- For the `.0` subnet: network = `.0`, first host = `.1`, last host = `.62`, broadcast = `.63`.

Practice until instant. **Private ranges** (RFC 1918, non-routable on the internet): `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` — you'll see these constantly in internal networks and labs.

**IPv6** — 128 bits, hex, e.g. `2001:db8::1`. Increasingly present; know it exists, that it's often *enabled and unmonitored* (a real blind spot attackers use), and the basics of its addressing.

**NAT (Network Address Translation)** — lets many private hosts share one public IP. Explains why your lab VMs have `192.168.x.x` addresses and why "which host actually did this?" is a real investigative question behind NAT.

---

## 7.3 TCP, UDP, and ports

**Ports** identify services on a host. A socket is `IP:port`. Know the well-known ones cold (make Anki cards):

| Port | Service | Security note |
|---|---|---|
| 20/21 | FTP | Cleartext creds |
| 22 | SSH | Encrypted admin access; brute-force target |
| 23 | Telnet | Cleartext — should never exist |
| 25 | SMTP | Email; spoofing/relay |
| 53 | DNS | Recon, tunneling, poisoning |
| 80 | HTTP | Cleartext web |
| 88 | Kerberos | AD authentication |
| 135/139/445 | RPC/NetBIOS/**SMB** | Windows file sharing; huge attack surface (EternalBlue, PtH) |
| 389/636 | LDAP/LDAPS | AD queries |
| 443 | HTTPS | Encrypted web |
| 3389 | RDP | Remote desktop; ransomware's favorite door |

**TCP** — connection-oriented, reliable, ordered. Begins with the **three-way handshake**:

```
Client → Server:  SYN            "let's talk, my sequence # is X"
Server → Client:  SYN-ACK        "ok, ack X+1, my sequence # is Y"
Client → Server:  ACK            "ack Y+1 — connected"
```

Understanding this explains port scanning: a full connection means the port is **open**; a **RST** reply means **closed**; no reply often means **filtered** (a firewall dropped it). Nmap's scan types (SYN scan, connect scan) are just variations on manipulating this handshake — which is why Chapter 4's low-level mindset pays off here.

**UDP** — connectionless, no handshake, fast, unreliable. Used by DNS, DHCP, VoIP, and many games. Harder to scan (no handshake to observe) and abused for **amplification DDoS** (small request, huge response, spoofed source).

---

## 7.4 The protocols that break — and why

Security is largely about the protocols that were designed in a trusting era and never got safe by default.

**ARP (Address Resolution Protocol, L2)** — maps IP addresses to MAC addresses on a local network. It has *no authentication* — a host simply believes any ARP reply. **ARP spoofing/poisoning** lets an attacker on the same LAN claim to be the gateway, redirecting traffic through themselves (**man-in-the-middle**). Foundational LAN attack; foundational reason to encrypt everything.

**DNS (Domain Name System, L7/UDP 53)** — translates names to IPs. Weaknesses everywhere: **DNS spoofing/cache poisoning** (feed false answers), **DNS tunneling** (smuggle data/C2 through DNS queries to evade firewalls — a classic exfiltration channel you'll hunt for in Part 4), and DNS as a rich *reconnaissance* source (subdomains reveal infrastructure). DNSSEC adds integrity but adoption is partial.

**DHCP (UDP 67/68)** — auto-assigns IP configuration. A **rogue DHCP server** can hand out a malicious gateway/DNS, enabling MITM.

**HTTP/HTTPS (L7)** — the web's protocol. HTTP is cleartext (readable/modifiable in transit); **HTTPS = HTTP over TLS**. You must know HTTP deeply for web hacking (Chapter 16): methods (GET/POST/PUT/DELETE), status codes (200/301/403/404/500), headers, cookies, and how sessions work.

**SMB (445)** — Windows file sharing. Historically catastrophic (the EternalBlue exploit → WannaCry/NotPetya). A primary lateral-movement and exploitation surface in enterprise networks.

**TLS (Transport Layer Security)** — the encryption under HTTPS, SSH-adjacent services, VPNs, and more. You must understand:
- The **handshake**: client hello → server hello + **certificate** → key exchange → encrypted session. It provides **confidentiality** (encryption), **integrity** (tamper detection), and **authentication** (the server proves identity via a certificate signed by a trusted CA).
- **Certificates and the chain of trust**: your browser trusts a set of Certificate Authorities (CAs); a valid cert chains up to one. This is **PKI** (Chapter 11).
- **Common failures**: expired/self-signed certs, weak cipher suites, downgrade attacks, and the reason certificate validation must never be disabled "to make it work" (a frequent, dangerous developer shortcut).

---

## 7.5 Network security controls (the defender's toolkit)

- **Firewalls** — filter traffic by rules. **Stateless** (per-packet) vs **stateful** (tracks connections) vs **next-gen/L7** (application-aware). Default-deny inbound is the baseline.
- **IDS/IPS (Intrusion Detection/Prevention Systems)** — watch traffic for known-bad patterns (signatures) or anomalies. IDS alerts; IPS blocks. **Snort** and **Suricata** are the open-source standards you'll meet in Part 4.
- **Proxies and WAFs (Web Application Firewalls)** — sit in front of servers to filter/inspect. WAFs specifically try to block web attacks (and are routinely bypassed — an arms race).
- **VPNs** — encrypted tunnels for remote access or site-to-site links.
- **Network segmentation** — dividing the network into zones (e.g., a DMZ for public servers, separate VLANs for finance) so a breach in one zone can't freely reach others. This is *defense in depth* at the network layer and a core mitigation for lateral movement.
- **Zero Trust** — the modern model: "never trust, always verify." Instead of a trusted internal network behind a hard perimeter, every request is authenticated and authorized regardless of origin. It exists precisely because flat internal networks let one foothold reach everything (recall the AD endgame in Chapter 6).

---

## 7.6 Reading traffic: Wireshark and tcpdump

You cannot claim to understand networking until you've *watched* it. Packet analysis is a core skill for forensics, detection, and debugging.

- **Wireshark** — GUI packet analyzer. Capture live or open a `.pcap` file. Use **display filters** (`http`, `tcp.port == 443`, `ip.addr == 10.0.0.5`, `dns`) to slice the traffic. "Follow TCP Stream" reconstructs a whole conversation.
- **tcpdump** — the command-line equivalent, essential on servers with no GUI: `tcpdump -i eth0 -w capture.pcap port 80`.

Watching a single web request in Wireshark — DNS lookup, TCP handshake, TLS handshake, encrypted application data — and being able to explain every packet is a defining "I get it now" moment. Make it happen this chapter.

---

## Common mistakes

- **Skipping subnetting because "tools do it."** You'll misjudge scope, misread firewall rules, and look junior in interviews. Do it by hand until it's instant.
- **Learning protocols abstractly.** Capture them in Wireshark. Seeing the actual ARP reply or TLS certificate exchange makes it real.
- **Not asking "what layer?"** for each attack. That question organizes the entire domain.
- **Assuming HTTPS means "secure and done."** TLS protects data in transit; it does nothing about the app's own vulnerabilities, and it's routinely undermined by disabled cert validation.
- **Ignoring IPv6.** It's often on and unmonitored — a real blind spot.

---

## Labs

> **Lab 7.1 — Subnet by hand.** Without a calculator, work out for `10.0.5.0/27`: number of hosts, all subnet ranges, and the network/first-host/last-host/broadcast for the third subnet. Then check with an online subnet calculator. Repeat with three more CIDRs until it's effortless.

> **Lab 7.2 — Watch a TCP handshake.** Start a Wireshark capture, then `curl http://example.com` (or browse a plain-HTTP site). Find the SYN, SYN-ACK, ACK. Then do it against an HTTPS site and identify the TLS handshake and the server's certificate. Write up every packet type you see, in order.

> **Lab 7.3 — See DNS resolution.** Capture traffic while running `dig example.com`. Find the DNS query and response. Note it's cleartext UDP. Write two sentences on how an on-path attacker could abuse this.

> **Lab 7.4 — ARP on your LAN.** Run `arp -a` to see your machine's IP↔MAC table. Read (do not run against others) how ARP spoofing works to become MITM, and write up *why* the protocol permits it and what defense (dynamic ARP inspection, encryption) counters it. ⚠️ Do not ARP-spoof any network you don't own.

> **Lab 7.5 — TryHackMe networking.** Complete TryHackMe's "Network Fundamentals" / "Intro to Networking" path and their Wireshark rooms. Guided, hands-on reinforcement of this entire chapter.

---

## References and further reading

- **Kurose & Ross — *Computer Networking: A Top-Down Approach*.** The standard, readable networking textbook; you may already have it. The top-down (start at HTTP) approach fits security thinking perfectly.
- **Charles Kozierok — *The TCP/IP Guide* (free online).** Exhaustive protocol reference. Use it to answer "how does X actually work?"
- **Chris Sanders — *Practical Packet Analysis* (with Wireshark).** The best hands-on book for actually reading traffic. Work through it with the sample captures.
- **Wireshark** — [wireshark.org](https://www.wireshark.org); their sample-capture wiki has real traffic to dissect.
- **Professor Messer's Network+ videos (free on YouTube)** — excellent, free, exam-aligned networking fundamentals if you want structured video.
- **TryHackMe network fundamentals path** — your primary hands-on lab.
- **PortSwigger / MDN on HTTP** — for the deep HTTP knowledge you'll need in Chapter 16.

---

## Self-check

1. For `172.16.8.0/22`, how many usable hosts, and what is the address range?
2. Walk through the TCP three-way handshake and explain how a port scanner uses it to tell "open" from "closed" from "filtered."
3. Why is ARP spoofing possible, and what does it let an attacker do?
4. What three security properties does TLS provide, and how does a certificate provide the "authentication" one?
5. What is network segmentation and which attacker behavior from Chapter 6 does it specifically hinder?

<details>
<summary>Answers</summary>

1. `/22` = 10 host bits → 1024 addresses, **1022 usable hosts**. Range: `172.16.8.0` (network) through `172.16.11.255` (broadcast); usable `172.16.8.1`–`172.16.11.254`.
2. Client sends SYN, server replies SYN-ACK, client sends ACK → connected. A scanner infers: SYN-ACK back = **open**; RST back = **closed**; no response (dropped) = **filtered** (firewall). SYN scans send the SYN and judge by the reply without completing the handshake.
3. ARP has no authentication — hosts accept any ARP reply mapping an IP to a MAC. An attacker on the same LAN can claim to be the gateway (or another host), so traffic is routed through them, enabling a **man-in-the-middle** to read/modify traffic.
4. **Confidentiality** (encryption), **integrity** (tamper detection), **authentication** (identity). Authentication comes from the server's **certificate**, signed by a trusted Certificate Authority; the client verifies the signature chains up to a CA it trusts and matches the domain, proving the server is who it claims.
5. Dividing the network into isolated zones so traffic between them is controlled/monitored. It specifically hinders **lateral movement** — an attacker who compromises one host/zone can't freely pivot across the whole network toward high-value targets like the Domain Controller.

</details>

---

## What's next

You can now read the network. Time to stop only *running* other people's tools and start *building your own*. [Chapter 8](08-scripting-and-automation.md) turns your CS degree into your biggest security advantage: the ability to automate, parse, and tool-build that most security folks lack.
