# Chapter 20 · Logging, Telemetry, and SIEM

> **Why this chapter matters:** Defense begins with *visibility*. You cannot detect, investigate, or respond to what you can't see. This chapter is about where evidence comes from — the logs and telemetry that record everything happening across an organization — and the SIEM that centralizes and lets you query it. This is the foundation of every blue-team role, and the SOC analyst job (the highest-volume entry point into security) *is* living in these tools. Everything you learned to *do* in Part 3 leaves traces here; now you learn to find them.

> **By the end of this chapter you will:** know the key data sources and what each reveals, understand log management and SIEM architecture, write queries in a SIEM query language, and understand the practical challenges (volume, noise, cost) that shape real detection.

---

## 20.1 The visibility principle

Every action on a system can leave a trace: a login, a process starting, a network connection, a file change, an API call. **Telemetry** is the collective term for this data. Defense is the discipline of collecting the *right* telemetry, centralizing it, and turning it into detections and investigations.

The attacker's whole post-exploitation playbook (Part 3) generates telemetry: the reverse shell is an outbound connection and an odd process; Kerberoasting is Event 4769; lateral movement is anomalous logons; persistence is a new service or scheduled task. **If it's logged and you're looking, it's catchable.** If it's not logged, you're blind — which is why *what you collect* is the first and most consequential defensive decision.

> **Recall Chapter 5/6:** attackers try to delete or tamper with local logs. This is *the* reason telemetry is shipped off-host to a central platform in near-real-time — so evidence survives the compromise and can be correlated across many systems. A log that only exists on the breached box is a log you can't trust.

---

## 20.2 The key data sources (and what each reveals)

| Source | Reveals | Example detections |
|---|---|---|
| **Windows Security Event Log** | Logons, privilege use, account/group changes, Kerberos | 4625 brute force, 4769 Kerberoasting, 4720 new account (Ch 6) |
| **Sysmon** (Windows) | Rich process creation w/ command lines & hashes, network conns, image loads, registry | Malicious process trees, LOLBins, C2 (Ch 6, 21) |
| **PowerShell logs** | Script block content, transcripts | Obfuscated/malicious PowerShell (Ch 8) |
| **EDR/XDR telemetry** | Endpoint behavior, process lineage, memory, response actions | The core of modern endpoint detection |
| **Linux logs / auditd** | Auth (auth.log), sudo, execve, file access | SSH brute force, privesc (Ch 5) |
| **Firewall / network flow (NetFlow)** | Connections, volumes, denied traffic | C2 beaconing, exfiltration, scanning |
| **DNS logs** | Every name lookup | DNS tunneling, C2 domains, DGA (Ch 7) |
| **Proxy / web logs** | Outbound web, user agents | Malware downloads, data exfil, C2 |
| **Web server / WAF logs** | Inbound web requests | SQLi/XSS attempts, recon (Ch 16) |
| **Cloud audit logs** (CloudTrail / Azure Activity / GCP Audit) | Every API/control-plane action | IAM abuse, resource changes, cloud attacks (Ch 26) |
| **Authentication/IdP logs** (Okta, Entra ID) | SSO logins, MFA events | Impossible travel, MFA fatigue, spraying (Ch 12) |
| **Application logs** | App-specific events | Business-logic abuse, auth failures |

**The principle of the right telemetry:** endpoint (Sysmon/EDR) + identity (auth/IdP) + network (firewall/DNS/proxy) + cloud audit logs together give you coverage across where attacks actually happen. Missing any one creates a blind spot attackers exploit (e.g., no DNS logging = no DNS-tunneling detection).

---

## 20.3 What is a SIEM?

A **SIEM (Security Information and Event Management)** platform is the blue team's central nervous system. It:
1. **Ingests** logs/telemetry from everywhere (agents, forwarders, APIs).
2. **Parses & normalizes** them into a common schema (so a "user" field means the same thing whether it came from Windows, Okta, or AWS).
3. **Stores & indexes** for fast search across huge volumes.
4. **Correlates** events across sources and **generates alerts** from detection rules.
5. **Supports investigation** (search, pivot, timeline) and dashboards/reporting.

The major platforms you'll encounter: **Splunk** (dominant, powerful, SPL query language), **Microsoft Sentinel** (cloud-native, KQL), **Elastic Security / ELK** (open source, popular in home labs), **QRadar**, **Chronicle**, **Wazuh** (open source, great for labs). **Learn one query language deeply** — the *concepts* transfer, and depth in one (SPL, KQL, or Elastic's) beats shallow familiarity with all.

**SIEM vs adjacent tools (know the vocabulary):**
- **SOAR** (Security Orchestration, Automation & Response) — automates response/enrichment on top of the SIEM (Chapter 8's automation applied to the SOC).
- **EDR/XDR** — endpoint-focused detection & response; feeds the SIEM and acts on endpoints.
- **UEBA** (User & Entity Behavior Analytics) — baselines normal behavior to flag anomalies. Often a SIEM feature.

---

## 20.4 Querying: turning logs into answers

The daily skill of a SOC analyst and detection engineer is *querying*. The languages differ syntactically but share concepts: **filter → transform → aggregate → sort**.

**Splunk SPL** — find failed logins by source:
```
index=windows EventCode=4625
| stats count by src_ip, Account_Name
| sort -count
```

**Microsoft Sentinel KQL** — same idea:
```kql
SecurityEvent
| where EventID == 4625
| summarize count() by IpAddress, Account
| sort by count_ desc
```

**Elastic (KQL/Lucene / ES|QL)** — filter for the same events, aggregate in a visualization.

Notice these are the *same pipeline* you learned in Bash (`grep | awk | sort | uniq -c | sort -rn`, Chapter 5) — filter, extract, count, rank — just at enterprise scale over normalized data. Your Chapter 5/8 skills transfer directly. The analyst's art is knowing *what question to ask* (what does this attack look like in the data?) — which is exactly why you learned the attacks in Part 3.

---

## 20.5 The hard realities (what makes this a real skill)

Detection isn't "collect everything and get alerted." The constraints shape everything:

- **Volume** — a mid-size org generates enormous log volume. You can't keep or search all of it forever.
- **Cost** — most SIEMs charge by data ingested/stored. This forces hard choices about *what* to collect and *how long* to keep it — a real architectural and budget decision, not a technicality.
- **Noise / false positives** — naive rules drown analysts in alerts, causing **alert fatigue** (the human failure mode that lets real alerts get missed — famously implicated in major breaches). Tuning is central (Chapter 21).
- **Coverage gaps** — you only detect what you log. Mapping your telemetry to ATT&CK (Chapter 13) reveals blind spots.
- **Parsing/normalization** — messy, inconsistent log formats must be wrangled into a usable schema; a huge part of real SIEM work.
- **Time** — accurate, synchronized timestamps (and time zones) are essential for correlation and timelines. Clock skew ruins investigations.

Understanding these is what separates someone who *ran a SIEM query* from someone who can *build and operate detection* in the real world — and it's exactly what interviews for SOC/detection roles probe.

---

## Common mistakes

- **Assuming "collect everything" is the goal.** Volume and cost force prioritization; collect the *right* telemetry mapped to real threats.
- **Only knowing one data source.** Attacks cross endpoint, identity, network, and cloud; single-source visibility misses the chain.
- **Learning a SIEM's clicks but not its query language.** The query language is the skill; master one deeply.
- **Ignoring false positives.** Untuned detection creates alert fatigue that hides real attacks — arguably worse than no detection.
- **Trusting only host-local logs.** Attackers tamper with them; centralize off-host.
- **Forgetting time synchronization.** Bad timestamps make correlation and timelines impossible.

---

## Labs

> **Lab 20.1 — Stand up a SIEM.** In your lab (Chapter 9 blue-team stage), deploy the free Elastic stack (ELK) or **Wazuh**, or use a free Splunk instance / Microsoft Sentinel free tier. Ship Windows Security + Sysmon logs and Linux auth logs into it. Confirm events are arriving. This is the environment for the rest of Part 4.

> **Lab 20.2 — Learn one query language.** Pick SPL, KQL, or Elastic's and complete its official tutorial (Splunk's free "Search Tutorial", Microsoft's KQL tutorial, or Elastic's). Then reproduce your Chapter 5 log one-liners (top failed-login sources, etc.) as SIEM queries. Feel the concepts transfer.

> **Lab 20.3 — Detect your own attacks.** Run a simple attack from Kali against a monitored host (a brute force, or the Chapter 18 Kerberoast). Then *find it in the SIEM* by querying for the expected Event IDs. Write up the query and what the attack looked like in the data. This attack-and-find loop is the essence of blue-team work.

> **Lab 20.4 — Map telemetry to ATT&CK.** For five ATT&CK techniques from Chapter 13, identify which log source(s) would reveal them and write the detection query concept. Note any technique your lab *can't* currently see — that's a coverage gap.

> **Lab 20.5 — Use a real dataset.** Download a public security dataset (e.g., Splunk's "Boss of the SOC" (BOTS) data, or Security Datasets / Mordor). Investigate a scenario end to end. This is the closest thing to real SOC experience you can get at home.

---

## References and further reading

- **Splunk Free / "Boss of the SOC" (BOTS)** — free SPL practice with realistic attack data. Excellent, hands-on.
- **Microsoft Sentinel + KQL** — the free KQL tutorial (Kusto) and Sentinel's GitHub detection repo. KQL is a great, in-demand language to learn.
- **Elastic Security / Wazuh docs** — for the open-source lab stack.
- **Security Datasets / Mordor (OTRF)** and **Splunk Attack Range** — pre-generated attack telemetry to practice detection.
- **Sysmon + SwiftOnSecurity config** (Chapter 6) — the enrichment that makes Windows detection possible.
- **"The DFIR Report"** ([thedfirreport.com](https://thedfirreport.com)) — shows exactly what real intrusions look like in logs, mapped to ATT&CK. Read weekly.
- **MITRE ATT&CK Data Sources** — which telemetry reveals which techniques; drives your collection strategy.
- **Gartner/vendor SIEM overviews** — for the market landscape (know the major platforms by name).

---

## Self-check

1. Why is telemetry shipped off-host to a central SIEM instead of kept locally?
2. Name four categories of data source and one attack each would reveal.
3. What does a SIEM do, in five steps, and how does SOAR relate to it?
4. Why isn't "collect and alert on everything" a viable detection strategy? Name two constraints.
5. How do your Chapter 5 Bash pipeline skills transfer to SIEM querying?

<details>
<summary>Answers</summary>

1. Because attackers who compromise a host can delete or tamper with its local logs to hide, and because detection/investigation require correlating events across many systems. Centralizing off-host in near-real-time preserves trustworthy evidence beyond the attacker's reach and enables cross-source correlation.
2. Examples: **Endpoint** (Sysmon/EDR) → malicious process trees / LSASS dumping; **Identity** (auth/IdP logs) → password spraying / impossible travel; **Network** (firewall/DNS/proxy) → C2 beaconing / DNS tunneling / exfiltration; **Cloud** (CloudTrail/Azure/GCP audit) → IAM abuse / unauthorized resource changes.
3. A SIEM (1) ingests logs from many sources, (2) parses/normalizes them to a common schema, (3) stores and indexes for fast search, (4) correlates events and generates alerts from detection rules, and (5) supports investigation and dashboards. **SOAR** sits on top, automating enrichment and response actions for those alerts.
4. Because **volume** (orgs generate more data than can be stored/searched indefinitely) and **cost** (SIEMs charge by ingest/storage) force prioritization, and because untuned rules create **false positives / alert fatigue** that bury real detections. You must collect the *right* telemetry mapped to real threats and tune rules.
5. SIEM queries follow the same pipeline as `grep | awk | sort | uniq -c | sort -rn` — **filter** events (e.g., EventID==4625), **extract/transform** fields, **aggregate** (summarize/stats count by), and **sort** by frequency — just over normalized data at enterprise scale. The concepts (filter → transform → aggregate → rank) are identical.

</details>

---

## What's next

You can collect and query telemetry. Now you learn to turn it into *detections that actually work in production* — accurate, tuned, and durable. [Chapter 21](21-detection-engineering.md) is detection engineering: writing the rules (Sigma, YARA, analytics) that catch attackers without drowning analysts, and the coding/automation mindset where a CS graduate shines.
