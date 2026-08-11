# Chapter 4 · Computing Fundamentals for Security

> **Why this chapter matters:** You have a CS degree, so you know these topics *academically*. Security requires knowing them *operationally* — not "a process has an address space" but "here is exactly how an attacker turns a bug in that address space into code execution." This chapter re-grounds the fundamentals through a security lens. Skim what you know; slow down where the security angle is new.

> **By the end of this chapter you will:** understand how a program becomes a running process in memory, why memory-corruption bugs happen and how they become exploits, how numbers and encodings hide attacks, and how the CPU privilege model underlies all system security.

---

## 4.1 From source to running process

Security lives in the gap between what a programmer *intended* and what the machine *actually does*. To see that gap you have to see the whole path from source code to execution.

1. **Source code** → **compiler** → **machine code** (for C/C++/Go/Rust) or **bytecode** (for Java/C#) or stays as **source interpreted at runtime** (Python/JS). The language's memory model determines its vulnerability classes: C/C++ expose raw memory (buffer overflows, use-after-free); memory-safe languages (Rust, Go, Java, Python) largely eliminate those but have their own issues (injection, deserialization, logic flaws).
2. The OS **loader** takes an executable file (ELF on Linux, PE on Windows, Mach-O on macOS), maps it into memory, resolves dependencies (shared libraries), and creates a **process**.
3. A process gets its own **virtual address space** — an illusion, maintained by the CPU + OS, that it has a large, private, contiguous block of memory. This isolation is a *security boundary*: process A cannot read process B's memory. Breaking that isolation is a whole category of attack.

> **Security insight:** every executable file format, every loader step, every library resolution is an attack surface. DLL hijacking abuses library resolution order. Malicious ELF/PE files abuse the loader. Understanding "how does this even run" is the prerequisite to understanding "how is this abused."

---

## 4.2 Memory layout of a process

This is the single most important diagram in low-level security. Memorize it.

```
High addresses
┌─────────────────────────┐
│         Stack           │  ← local variables, function call frames, return addresses
│           ↓ grows down   │     (LIFO; grows toward lower addresses)
│                          │
│      (unused gap)        │
│                          │
│           ↑ grows up     │
│         Heap             │  ← dynamically allocated memory (malloc/new)
├─────────────────────────┤
│  BSS  (uninit. globals)  │
│  Data (init. globals)    │
│  Text (code, read-only)  │  ← the program's machine instructions
└─────────────────────────┘
Low addresses
```

- **Text/code segment** — the actual instructions. Normally read-only and non-writable (so attackers can't just overwrite the program).
- **Data / BSS** — global and static variables.
- **Heap** — memory you request at runtime (`malloc`, `new`). Grows upward. Manual management here causes use-after-free, double-free, and heap overflows.
- **Stack** — grows *downward*. Holds each function call's **stack frame**: its local variables, saved registers, arguments, and — critically — the **return address** telling the CPU where to resume when the function finishes.

### Why the stack is the classic attack target
When you call a function, the CPU pushes the return address onto the stack, then the function's local variables sit *right next to it*. If a local buffer (say `char name[64]`) can be written past its end, the overflow continues into adjacent memory — including the saved return address. Overwrite the return address, and when the function returns, the CPU jumps wherever you told it. That's a **stack buffer overflow**, the archetypal memory-corruption exploit.

```c
// The canonical vulnerable function
void greet(char *input) {
    char name[64];
    strcpy(name, input);   // strcpy copies until it hits a null byte — no bounds check!
    printf("Hello, %s\n", name);
}
// If `input` is 100 bytes, strcpy writes 100 bytes into a 64-byte buffer.
// The extra 36 bytes overwrite the saved registers and return address.
```

You don't need to *write* exploits like this to work in most of security, but you must **understand** them, because: (a) they still exist in C/C++ codebases and firmware everywhere; (b) understanding them is what makes concepts like ASLR, DEP, and stack canaries make sense; and (c) they're on every offensive certification and many interviews.

### Modern mitigations (and why exploits got harder)
Attackers overwrite return addresses; defenders responded with layered mitigations. Know these by name:

- **Non-executable stack / DEP / NX bit** — marks the stack/heap non-executable, so attacker-injected code there won't run. Attackers responded with **Return-Oriented Programming (ROP)** — chaining together existing snippets of legit code ("gadgets") instead of injecting new code.
- **ASLR (Address Space Layout Randomization)** — randomizes where the stack, heap, and libraries load, so the attacker can't predict addresses. Defeated by info leaks that reveal an address.
- **Stack canaries / stack cookies** — a secret value placed just before the return address; if an overflow changes it, the program aborts before returning. Defeated by leaking or brute-forcing the canary.
- **CFI (Control-Flow Integrity)** — validates that jumps/returns go only to legitimate targets.

The lesson isn't the specific bypasses — it's the **pattern of security as an arms race**: every defense provokes a new offense, forever. You saw this framing in Chapter 2; here it is at the machine level.

---

## 4.3 Numbers, encodings, and how they hide attacks

Attacks constantly exploit the fact that the same bytes mean different things in different contexts, and that number representations have edges.

**Number bases you must read fluently:**
- **Binary** (base 2) — how the machine actually stores everything.
- **Hexadecimal** (base 16) — the human-readable shorthand for bytes. `0x41` = 65 = `'A'`. You'll read hex constantly (memory dumps, hashes, packet captures, shellcode). Get fluent: `0x00`–`0xFF` is one byte, 0–255.
- **Decimal** — for humans.

**Integer issues that become vulnerabilities:**
- **Integer overflow** — a value exceeds its type's max and wraps around. `255 + 1` in an 8-bit unsigned int becomes `0`. If that value is a buffer size or a loop count or a bank balance, wrapping it is an exploit. (A length check `if (len < MAX)` can be bypassed if `len` overflows to a small number.)
- **Signed/unsigned confusion** — `-1` as a signed int is `0xFFFFFFFF` as unsigned, i.e., a huge number. A check that assumes non-negative can be tricked.

**Encodings — the shape-shifters of security:**
- **ASCII / Unicode (UTF-8)** — how text maps to bytes. Unicode normalization and homoglyphs (characters that look identical but differ in bytes) power phishing and filter bypasses.
- **Base64** — encodes binary as text (used in email, JWTs, and *constantly* by malware to hide payloads). Not encryption — trivially reversible. When you see a long `SGVsbG8=`-style string, decode it. Attackers base64-encode PowerShell commands to evade simple detection; you'll decode these in SOC work daily.
- **URL encoding** (`%20` = space, `%27` = `'`) — used to smuggle payloads past web filters.
- **Hex encoding, HTML entities** — more of the same shape-shifting.

> **Security insight:** a huge fraction of attacks are fundamentally **encoding and context confusion** — data that's safe in one context becomes dangerous when reinterpreted in another (a filename becomes a command, a comment becomes code, text becomes an instruction). SQL injection, XSS, command injection, and even prompt injection are all variations of this one idea. Learn to see it. **CyberChef** (an in-browser tool, "the Cyber Swiss Army Knife") is how you'll decode and transform between all these representations — install it mentally as your go-to.

---

## 4.4 Processes, privileges, and the CPU privilege model

Why can't any program just read all of memory or reformat the disk? Because the CPU enforces **privilege levels** (rings), and the OS uses them to build isolation.

- **Ring 0 (kernel mode)** — full hardware access. The OS kernel and drivers run here. A compromise here ("kernel-level" / "rootkit") owns the machine totally.
- **Ring 3 (user mode)** — restricted. Your applications run here. They can't touch hardware or other processes' memory directly; they must ask the kernel via **system calls** (`syscalls`).
- The transition from ring 3 to ring 0 (a syscall) is a **security boundary**. Much of exploitation is about crossing boundaries you shouldn't: user→kernel (privilege escalation via a kernel bug), one user→another user (horizontal), unprivileged user→admin/root (vertical).

**Privilege escalation** — turning limited access into more access — is one of the most important concepts in the whole field. When an attacker gets a foothold, they almost always land as a low-privileged user and must escalate to do real damage. You'll spend serious time on this in [Chapter 17](../part-3-offensive-security/17-host-and-network-exploitation.md). The seeds are here: **least privilege** (Chapter 10) is the defensive counterpart — if that foothold process had no privileges worth stealing, escalation gets much harder.

**Key process concepts through a security lens:**
- **PID / parent-child relationships** — attacks show up as anomalous process trees (Word spawning PowerShell spawning `cmd` — a red flag you'll learn to spot in Chapter 21).
- **Process privileges / tokens (Windows) / uid-gid (Linux)** — *who* a process runs as determines what it can do. Stealing a more-privileged token or process is a core attack.
- **Environment variables and command-line arguments** — often carry secrets (a mistake) and are readable by other processes on the box.

---

## 4.5 Filesystems and data at rest

Data doesn't vanish when you "delete" it, and filesystems store far more than file contents. This underlies all of digital forensics ([Chapter 23](../part-4-defensive-security/23-digital-forensics.md)).

- **"Deleting" a file** usually just removes its directory entry and marks its blocks free — the data stays on disk until overwritten. This is why forensic tools can recover "deleted" files and why secure deletion (overwriting) exists.
- **Metadata** — filesystems record timestamps (created/modified/accessed — "MAC times"), permissions, ownership, and more. Forensics reconstructs timelines from this; attackers "timestomp" to hide.
- **Permissions** — Linux `rwx` for user/group/other, plus special bits (SUID/SGID — a program running with its *owner's* privileges, a classic privilege-escalation vector); Windows ACLs. You'll go deep on these in Chapters 5 and 6.
- **Slack space, unallocated space, journaling, shadow copies** — all hold recoverable data an investigator (or attacker) can mine.

> **Security insight:** "encryption at rest" exists precisely because physical/logical access to storage reveals everything the filesystem hasn't truly erased. And Volume Shadow Copies (Windows) are both a backup feature and a place attackers steal credential databases from.

---

## 4.6 How this chapter connects to everything else

| This concept | Powers this later topic |
|---|---|
| Memory layout & overflows | Exploit development, malware analysis (Ch 24), why memory-safe languages matter |
| Encodings & context confusion | Injection attacks, XSS, WAF bypass (Ch 16), prompt injection (Ch 29) |
| Privilege rings & syscalls | Privilege escalation (Ch 17), EDR/kernel telemetry (Ch 20–21) |
| Process trees & tokens | Detection engineering (Ch 21), IR (Ch 22) |
| Filesystem metadata & deletion | Digital forensics (Ch 23) |

You are not expected to master exploit development from this chapter. You *are* expected to be able to explain, out loud, how a buffer overflow overwrites a return address, why encodings enable injection, and what a privilege boundary is. That mental model is the soil everything else grows in.

---

## Common mistakes

- **Assuming your CS degree already covered this operationally.** It covered the theory. The security framing — "here's how the bug becomes an exploit" — is usually new.
- **Trying to master exploit development now.** That's a Part 3+ specialization. Here you build the *model*, not the skill.
- **Glossing over encodings as "trivial."** Encoding/context confusion is one of the deepest ideas in security. Respect it.
- **Ignoring the low level because "I'll work in the cloud / with memory-safe languages."** The vulnerabilities move up the stack but the *thinking* — boundaries, trust, reinterpretation — is identical everywhere.

---

## Labs

> **Lab 4.1 — See a process's memory map.** On a Linux VM, run a program (e.g. `sleep 1000 &`), find its PID, and `cat /proc/<pid>/maps`. Identify the stack, heap, and the loaded libraries. Write down what you see and match it to §4.2's diagram.

> **Lab 4.2 — Watch a buffer overflow conceptually.** Compile the vulnerable `greet()` program from §4.2 (`gcc -fno-stack-protector -z execstack vuln.c -o vuln` on a lab VM — mitigations off *for learning only*). Feed it increasingly long input until it crashes (segfault). You've just corrupted the return address. You don't need to weaponize it — observe that too much input crashes the program by overwriting control data. Write up *why* it crashed. ⚠️ Lab VM only.

> **Lab 4.3 — Decode like an analyst.** Open CyberChef (gchq.github.io/CyberChef, or run it locally). Decode these: base64 `cG93ZXJzaGVsbCAtZW5j` ; URL-encoded `%3Cscript%3E` ; hex `0x6d616c` . For each, note what an attacker might be hiding. Then encode a command *yourself* in base64 and decode it back.

> **Lab 4.4 — Integer overflow by hand.** In any language, write a loop that adds 1 to an 8-bit unsigned value starting at 250 and print each result. Watch it wrap `255 → 0`. Then write two sentences on how a length or price field wrapping could be a vulnerability.

> **Lab 4.5 — Privilege rings in practice.** On Linux, run `strace ls` and watch the system calls scroll by — every one is a user→kernel boundary crossing. Pick three syscalls, look them up (`man 2 <name>`), and note what boundary each crosses.

---

## References and further reading

- **Randal Bryant & David O'Hallaron — *Computer Systems: A Programmer's Perspective* (CSAPP).** The best single book for the operational understanding of memory, processes, and machine-level execution. If one section is fuzzy, this book fixes it. Chapters 3 and 9 especially.
- **"Smashing the Stack for Fun and Profit" — Aleph One (Phrack, 1996).** The legendary original paper on stack buffer overflows. Dated in specifics, timeless in explanation. Free online. Read it once.
- **Jon Erickson — *Hacking: The Art of Exploitation* (2nd ed.).** Teaches C, assembly, and exploitation from the ground up with a bundled Linux environment. The best hands-on bridge from "CS grad" to "understands exploits." Work through Chapters 0x200–0x300.
- **CyberChef** — [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef). Bookmark it. You'll use it weekly forever.
- **"What Every Programmer Should Know About Memory" — Ulrich Drepper.** Deep, optional, superb.
- **OverTheWire — Narnia / Behemoth wargames** — free, browser-accessible binary exploitation practice when you want to go deeper.

---

## Self-check

1. In a stack buffer overflow, what specific piece of data does the attacker most want to overwrite, and what happens when they do?
2. Explain why ASLR and DEP/NX made exploitation harder, and name the offensive technique that responded to DEP.
3. Base64 is not encryption. Why do attackers use it constantly anyway?
4. What is a privilege boundary, and give one example of an attack that crosses one.
5. Why can a "deleted" file often be recovered, and what does that fact justify (name a control)?

<details>
<summary>Answers</summary>

1. The **saved return address** in the current stack frame. Overwriting it means that when the function returns, the CPU jumps to an attacker-chosen address instead of the legitimate caller — giving the attacker control of execution flow.
2. **DEP/NX** marks memory like the stack non-executable, so injected shellcode there won't run; **ASLR** randomizes memory locations so the attacker can't predict where to jump or find their payload. DEP was answered by **Return-Oriented Programming (ROP)** — chaining existing executable code fragments instead of injecting new code.
3. To **hide/obfuscate payloads** from simple signature-based detection and to safely transport binary data through text-only channels (email, HTTP headers, JWTs). It's trivially reversible, which is exactly why it's for evasion, not secrecy.
4. A boundary the CPU/OS enforces between privilege levels (e.g., user mode ring 3 → kernel mode ring 0, or one user account → another). Example: a **privilege-escalation** exploit that abuses a kernel bug to go from a normal user to root/SYSTEM crosses the user→kernel boundary.
5. Because deletion typically only removes the directory entry and marks blocks free without overwriting the actual data, which persists until reused. This justifies **secure deletion (overwriting/wiping)** and **encryption at rest** — and is the basis of file recovery in digital forensics.

</details>

---

## What's next

You understand what happens under your code. Now you learn to *operate* — starting with the operating system every security professional lives in. [Chapter 5](05-linux-for-security.md) makes you fluent in Linux: not "I can use it" but "I can investigate, script, and defend it in my sleep."
