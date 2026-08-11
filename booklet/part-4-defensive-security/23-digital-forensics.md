# Chapter 23 · Digital Forensics

> **Why this chapter matters:** When an incident happens, someone has to answer "what *exactly* did the attacker do, when, and what did they take?" — in a way that holds up to scrutiny, including in court. Digital forensics is that rigorous, defensible reconstruction. It powers the identification phase of IR (Chapter 22), stands alone as the DFIR specialization, and builds an obsessive attention to evidence that makes you a better analyst everywhere. It also connects straight back to Chapter 4: filesystems, memory, and deleted-data recovery are the raw material.

> **By the end of this chapter you will:** understand forensic principles (order of volatility, chain of custody), the major evidence sources and artifacts, disk and memory forensics workflows and tools, and how to build a defensible timeline.

---

## 23.1 The forensic mindset and principles

Forensics differs from ordinary troubleshooting by its **rigor and defensibility**. The core principles:

- **Preserve, don't alter.** Every action you take on evidence may change it. You work on *copies* (forensic images), verify integrity with hashes, and document everything. Locard's exchange principle ("every contact leaves a trace") applies to *you* too — minimize your footprint on the evidence.
- **Chain of custody.** A documented, unbroken record of who handled the evidence, when, and how — from collection to analysis to storage. If the chain breaks, the evidence may be inadmissible and your conclusions untrustworthy. This is what makes findings hold up legally.
- **Integrity via hashing.** Hash evidence at acquisition (Chapter 11) and re-verify to prove it wasn't altered. A matching hash proves the copy equals the original.
- **Repeatability & documentation.** Another examiner following your documented steps should reach the same conclusions. Contemporaneous notes are non-negotiable.
- **Order of volatility** — collect the most *ephemeral* evidence first, because it disappears fastest:

```
Most volatile → least volatile
1. CPU registers, cache
2. RAM (running processes, network connections, decrypted data, injected code)
3. Network state (connections, ARP/routing tables)
4. Running processes / temp files
5. Disk (files, deleted data, slack space)
6. Logs (remote/archived)
7. Physical/offline backups, printouts
```
This is why "pull the plug" (Chapter 22) is often *wrong* — it destroys RAM, which may hold the only copy of decryption keys, injected malware, or the attacker's active session. Live acquisition of memory before shutdown is frequently critical.

---

## 23.2 Disk forensics

Recovering and interpreting data from storage — the traditional heart of forensics, built on Chapter 4's filesystem knowledge.

**Acquisition:** create a **forensic image** — a bit-for-bit copy of the drive — using a **write blocker** (hardware/software that prevents any modification to the original). Tools: `dd`/`dc3dd`, FTK Imager, Guymager. Hash before and after to prove integrity.

**What you recover and analyze:**
- **Files (active, deleted, and hidden).** Recall Chapter 4: "deleted" files often persist until overwritten — forensic tools recover them from unallocated space (**file carving** reconstructs files from raw data by signatures).
- **Filesystem metadata** — MAC times (Modified/Accessed/Created — the "MACB" timestamps), which build timelines. Attackers "timestomp" to hide, but inconsistencies across metadata sources expose it.
- **Windows artifacts (a rich menu):** the **Registry** hives (evidence of execution, USB devices, persistence — Chapter 6), **Prefetch** (what programs ran and when), **Shimcache/Amcache** (execution evidence), **Event Logs**, **LNK files & Jump Lists** (file access), **browser history**, **$MFT** (master file table), **Volume Shadow Copies** (past states), **`$Recycle.Bin`**, **SRUM** (resource usage). Each answers a specific question ("what ran?", "what was plugged in?", "what did they open?").
- **Linux artifacts:** logs (Chapter 5), `.bash_history`, cron, systemd, SSH `known_hosts`/`authorized_keys`, `/tmp`, file timestamps.

**The output is a timeline** — a chronological reconstruction of events across all artifacts. **Super-timelining** tools (`plaso`/`log2timeline`) aggregate every timestamped artifact into one master timeline, the centerpiece of most investigations.

---

## 23.3 Memory forensics

RAM holds evidence that exists *nowhere else*: running processes, network connections, injected/fileless malware, decrypted data, credentials, command history. Modern attacks are increasingly **fileless** (living in memory to evade disk-based detection), making memory forensics essential.

**Acquisition:** capture a **memory image** from the live system *before* shutdown (order of volatility!) with tools like WinPmem, DumpIt, or LiME (Linux).

**Analysis with Volatility** (the standard framework — learn it): from a memory image you can list processes (`pslist`, and `psscan` to find *hidden* ones), see network connections (`netscan`), find injected code (`malfind`), extract command history, dump process memory, recover registry from RAM, and detect rootkit techniques (process/DLL hiding). Memory analysis frequently reveals what disk analysis can't — the malware that never touched disk, the network beacon, the credentials in an attacker's process.

---

## 23.4 Specializations and scope

Forensics spans several sub-fields you may encounter:
- **Host/computer forensics** (disk + memory, above).
- **Network forensics** — reconstructing activity from packet captures and flow data (Chapter 7's Wireshark, at investigation scale) — what was exfiltrated, what C2 looked like.
- **Mobile forensics** — phones (huge in criminal cases; specialized tools like Cellebrite).
- **Cloud forensics** — investigating in cloud environments where you don't control the hardware (Chapter 26) — reliant on cloud logs, snapshots, and APIs; a growing challenge.
- **Malware forensics** — analyzing the malicious code found (Chapter 24).

**DFIR** (Digital Forensics & Incident Response) combines this chapter and Chapter 22 into one of the most respected blue-team career tracks.

---

## 23.5 Anti-forensics (what attackers do to hinder you)

Attackers actively fight forensics: log deletion/tampering (Chapter 5), timestomping, wiping free space, encryption, fileless/in-memory operation, using legitimate tools ("living off the land" — LOLBins), and steganography. Understanding anti-forensics helps you (a) find the traces that *survive* (attackers rarely clean perfectly — a deleted log may persist in a shadow copy or a central SIEM; timestomping leaves metadata inconsistencies) and (b) appreciate why *centralized, tamper-resistant logging* (Chapter 20) is so valuable — it puts evidence beyond the attacker's reach.

> **The through-line:** forensics is applied Chapter 4 (how data is really stored, why deletion isn't deletion) plus the discipline to make your findings *defensible*. That discipline — hash it, image it, document it, don't alter it — is a professional habit that elevates all your work.

---

## Common mistakes

- **Working on the original evidence.** Always image and work on copies; use write blockers; hash to prove integrity.
- **"Pull the plug" reflexively.** Destroys memory, often the most valuable evidence. Consider live memory acquisition first.
- **Poor documentation / broken chain of custody.** Renders findings untrustworthy or inadmissible. Document contemporaneously.
- **Relying on one artifact.** Corroborate across sources; single artifacts can be tampered or misleading (attackers timestomp). Timelines from many sources reveal the truth.
- **Ignoring memory / assuming disk is enough.** Fileless attacks live in RAM; disk-only forensics misses them.
- **Contaminating evidence with your own activity.** Minimize and document your footprint (Locard applies to you).

---

## Labs

> **Lab 23.1 — Image and verify.** Create a forensic image of a small USB drive or a VM disk with FTK Imager (free) or `dd`. Hash the source and the image; confirm they match. Practice the preserve-and-verify discipline. Document a mock chain of custody.

> **Lab 23.2 — Recover deleted files.** Delete files on a test drive/image, then recover them with Autopsy (free) or `photorec`/`foremost` (file carving). Prove Chapter 4's lesson: deletion ≠ erasure. Write up how and why recovery worked.

> **Lab 23.3 — Windows artifact hunt.** On a Windows VM, perform some activity (run programs, plug in a USB, browse), then investigate with Autopsy or the artifact tools: find execution evidence (Prefetch/Amcache), USB history (Registry), and browser history. Reconstruct what "the user" did. This is core DFIR work.

> **Lab 23.4 — Memory forensics with Volatility.** Capture (or download a sample) memory image and analyze it with Volatility: list processes, find network connections, look for injected code with `malfind`. Public "memory forensics challenge" images exist for practice. Write up what memory revealed that disk wouldn't.

> **Lab 23.5 — Build a super-timeline.** Use `plaso`/`log2timeline` on a disk image (or a provided dataset) to build a timeline, then investigate a scenario by following it. Timelining is the investigative centerpiece.

> **Lab 23.6 — CTF-style forensics.** Do forensics challenges on **CyberDefenders**, **DFIR CTFs**, or picoCTF's forensics category. Realistic, guided, and portfolio-worthy writeups.

---

## References and further reading

- **Cory Altheide & Harlan Carvey — *Digital Forensics with Open Source Tools*** and **Harlan Carvey — *Windows Forensic Analysis*.** The practical standards; Carvey's Windows work is essential for the artifact detail.
- **"The Art of Memory Forensics" — Ligh, Case, Levy, Walters.** The definitive memory forensics book (by Volatility's creators). Dense and excellent.
- **Autopsy / The Sleuth Kit** — [sleuthkit.org](https://www.sleuthkit.org) — free, powerful disk forensics platform. Your primary lab tool.
- **Volatility Framework** — [volatilityfoundation.org](https://www.volatilityfoundation.org) — the memory forensics standard.
- **SANS DFIR posters & cheat sheets** (free) — the "Windows Forensic Analysis" and "Hunt Evil" posters map artifacts to questions. Print them.
- **CyberDefenders, DFIR.training, Ali Hadi's forensic challenges** — free, realistic datasets and challenges.
- **plaso/log2timeline** — [github.com/log2timeline/plaso](https://github.com/log2timeline/plaso) — super-timelining.
- **SWGDE / NIST forensic guidance** — for the rigor, standards, and legal-admissibility side.

---

## Self-check

1. What is the order of volatility and what practical IR decision does it inform?
2. What is chain of custody and why does it matter even in a non-legal internal investigation?
3. From Chapter 4: why can deleted files be recovered, and what technique reconstructs them from raw data?
4. Why is memory forensics essential against modern attacks, and what tool is the standard?
5. Name three Windows artifacts and the investigative question each answers.

<details>
<summary>Answers</summary>

1. It ranks evidence from most to least ephemeral (CPU/cache → RAM → network state → disk → archived logs → backups), meaning you collect the most volatile evidence first. It informs the IR decision *not* to reflexively power off a system — because shutting down destroys RAM, which may hold the only copy of decryption keys, fileless malware, or the attacker's active session; live memory acquisition should often come first.
2. It's the documented, unbroken record of who handled evidence, when, and how, from collection through storage. It matters because it proves the evidence wasn't altered or contaminated — establishing trust in your conclusions (and admissibility if it ever becomes legal). Even internally, it keeps findings credible and repeatable.
3. Because deletion typically only removes the filesystem's directory entry and marks the blocks free without overwriting the data, which persists until reused. **File carving** reconstructs files directly from raw unallocated space by recognizing file signatures/headers, independent of filesystem metadata.
4. Because modern attacks are increasingly **fileless** — living entirely in RAM to evade disk-based detection — so critical evidence (running/hidden processes, network beacons, injected code, credentials, decrypted data) exists nowhere but memory. The standard analysis tool is **Volatility**.
5. Examples: **Prefetch/Amcache/Shimcache** — "what programs executed and when?"; **Registry (USBSTOR)** — "what USB devices were connected?"; **LNK files/Jump Lists/browser history** — "what files/sites did the user access?"; **$MFT** — "what files existed and their timestamps?".

</details>

---

## What's next

Investigations often turn up a suspicious file. [Chapter 24](24-malware-analysis-and-reverse-engineering.md) teaches you to answer "what does this thing do?" — malware analysis and reverse engineering, from safe static triage to dynamic behavioral analysis, building directly on Chapter 4's low-level foundations.
