# 09 · Incident response scenarios

[← Cloud and supply chain](../08-cloud-and-supply-chain/README.md) · [Index](../README.md) · [Next: Threat modelling →](../10-threat-modelling-and-design/README.md)

Jane Street runs "several scenario-based interviews, e.g. malware on a corporate system, spread over one afternoon" plus the hiring manager ([Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,34.htm)). The online assessment ends with a briefing for the CEO. So this block is two skills: running the first hour well, and explaining it to someone who doesn't care about PowerShell. Tags: [JS] Jane Street, [SOC] SOC and detection question banks, [GN] gracenolan, [TW] tadwhitaker, [DM] Miessler.

---

## The shape of a good scenario answer

Every scenario gets the same skeleton, said in this order, in under three minutes:

1. **Confirm** it's real: what the alert says, what the log shows, what the user says. Say what would make it a false positive.
2. **Scope** before you act: one host or many, one account or many, since when. One grep across the fleet is cheap; isolating the wrong thing isn't.
3. **Contain** proportionally: isolate the host (keep it on), disable the account, revoke the token, block the domain. Name the business impact of each and who you'd tell.
4. **Preserve**: memory, disk, logs, before anyone "fixes" anything.
5. **Eradicate and recover**: reimage rather than clean, rotate what was exposed, restore from known good.
6. **Learn**: root cause versus symptom, the detection that was missing, the control that would have stopped it.
7. **Communicate** throughout: a line to the team now, a briefing to leadership at the first checkpoint, a write-up after.

Say the trade-off every time. "Isolating now stops spread but may tip off the attacker and lose the trader's afternoon. I'd isolate because the host is already confirmed malicious; if it were only suspicious I'd watch for ten minutes first."

---

## Scenarios that have been asked

<details>
<summary><b>Malware on a corporate system.</b> · [JS Cyber Security Analyst, scenario round]</summary>

Confirm: what fired (EDR, AV, a user), the hash and its reputation, the process tree. Scope: search the hash, the parent process pattern and any C2 domain across every endpoint; check whether the user's account did anything odd (new logons, mailbox rules). Contain: network-isolate through EDR, keep it powered on, suspend the user's sessions if credentials may be gone. Preserve: memory capture, EDR timeline export, the file itself. Eradicate: reimage; rotate the user's password and any secrets on the box; check persistence on any other host that ran the same thing. Learn: how it arrived (phish, download, USB), why the control missed it, what detection to add. Brief: "One trading-floor laptop ran malware from a phishing attachment at 12:47. We isolated it at 13:05, the trader is on a spare machine, no other hosts show the same activity, and we're confirming nothing left the building."
</details>

<details>
<summary><b>Multiple failed logins followed by a success from an unusual location.</b> · [SOC, the most common scenario]</summary>

Validate: the account, success or failure, source IP and ASN, device, time, and whether the user travels. Compare with the user's normal pattern. Look at what happened right after: MFA registration changes, mailbox forwarding rules, new OAuth app consents, password reset, unusual data access. If it's explained (VPN, travel), document and close. If not: revoke sessions and tokens, reset the password, remove any added MFA devices, check the source IP against other accounts (spray), and tell the user. Detection improvement: alert on success-after-failures and on impossible travel, not on failures alone.
</details>

<details>
<summary><b>A user clicked a link in a phishing email. What now?</b> · [SOC] [TW: analyse a suspicious link]</summary>

Get the email and the URL. Was it credential phishing or a download? Credential: assume the password is gone, reset it, revoke sessions, check for MFA fatigue or a relayed session, look at sign-ins since the click. Download: treat as the malware scenario. Then scope: who else received it (search the mail store by sender, subject, URL, hash), who else clicked (proxy logs), and pull the message from every mailbox. Block the domain and the sender, submit the sample, and tell the reporter thank you, because the next report depends on it. Analysis walk-through if they ask: headers (From versus Reply-To versus Return-Path, SPF, DKIM, DMARC, hops), link target and domain age, attachment type and hash, detonate in a sandbox, verdict, then write the signature.
</details>

<details>
<summary><b>Unusual outbound traffic from a workstation, or one host beaconing to an unknown domain.</b> · [SOC]</summary>

Identify the process making the connection (EDR, `ss -p`), the destination's reputation and age, the pattern (regular interval and tiny payloads say C2; large bursts say exfiltration), and how many hosts talk to it: one is targeted, many is either a campaign or a legitimate SaaS. Contain by blocking the destination at the proxy and DNS and isolating the host if the process is malicious. Then back in time: what ran on that host just before the first connection. Log analysis commands are in [block 02](../02-log-investigation/README.md).
</details>

<details>
<summary><b>Possible ransomware. Walk me through leading the response.</b> · [SOC]</summary>

Minutes matter: isolate affected hosts and, if it's spreading, segments; disable the accounts it's using; stop backup jobs so they don't overwrite good copies; preserve one encrypted sample and the ransom note. Identify the strain and the entry point (usually a phishing payload, an exposed RDP or VPN, or a stolen credential days earlier). Scope by searching for the dropper and the lateral movement pattern. Recover from offline backups after confirming the backdoor is gone, which means rotating every credential including `krbtgt`. Communicate: leadership within the hour, legal on regulatory clocks, no contact with the attacker without legal. Lead point: assign roles (incident lead, comms, technical lanes, scribe) before the first command.
</details>

<details>
<summary><b>An AWS access key was pushed to a public repo 20 minutes ago.</b> · [common scenario]</summary>

Revoke first, investigate second: bots find public keys in minutes. Then CloudTrail for every call made with that key since the push (new users, new keys, EC2 in odd regions for mining, S3 listing). Rotate anything the key could reach, remove it from git history and force-push, enable push protection and a pre-commit scanner, and ask why a long-lived key existed at all; the fix is roles and short-lived credentials.
</details>

<details>
<summary><b>Impossible travel, a PowerShell alert, a new scheduled task, a suspicious hash on one endpoint, lateral movement.</b> · [SOC]</summary>

Impossible travel: VPN and mobile carriers cause most of these; check the device and app, then treat an unexplained one as a stolen session. PowerShell: decode the command, check the parent, check outbound connections (see [block 07](../07-windows-and-endpoints/README.md)). Scheduled task: who created it, what it runs, from where; a task pointing at AppData is persistence. One endpoint with a bad hash: isolate, then search the fleet; one host is the start, not the end. Lateral movement: 4624 type 3 with NTLM, one account on many hosts in minutes, PsExec-style services (7045); contain the account, not just the host.
</details>

<details>
<summary><b>A suspected insider is exfiltrating data.</b> · [SOC] [JS ACME scenario theme]</summary>

Different rules: preserve evidence before anything else, involve HR and legal before you touch the person's access, don't tip them off, and document chain of custody. Confirm with logs (what was read, when, where it went, whether it matches the role), scope what else they accessed, then act with HR in the room: disable access, collect devices. Say the trap: treating it as a technical incident alone, or acting on a hunch without evidence.
</details>

<details>
<summary><b>We've been told by an ISP that our machines are compromised. Investigate by asking me questions.</b> · [DM role-play]</summary>

Ask in order: what did the ISP see (IPs, ports, times, what kind of traffic), which of our systems hold those IPs (NAT means it could be anything), what those systems do and who uses them, what logs and tooling exist, what changed recently, whether there's any known exposure. Then narrow: the host, the process, the account, the entry point. Miessler scores the quality and order of your questions, not a solution.
</details>

<details>
<summary><b>How would you build a timeline? Symptom versus root cause?</b> · [SOC] [GN]</summary>

Collect timestamps from every source into one normalised table (UTC, source, host, user, event), sort, and mark the earliest attacker touch, initial access, persistence, lateral movement, impact. Gaps in the timeline are findings too. Symptom: the miner on the box. Root cause: the exposed service with a default password that let it in. The write-up fixes the cause.
</details>

<details>
<summary><b>When do you involve legal, users, managers, directors? When do you stop the attack rather than watch?</b> · [GN]</summary>

Legal early when personal data, regulatory clocks (72 hours for GDPR), law enforcement, insurance or an insider are in play. Management at the first checkpoint with impact in business terms and a next update time. Users when their action is needed or their data is affected. Watching the attacker gives you scope and intent; it costs exposure every minute, so watch only while the thing at risk is contained elsewhere and never when money or trading systems are reachable. The decision is a named person's, usually the incident lead with the business owner.
</details>

<details>
<summary><b>What would you do if you couldn't tell whether an alert was malicious? If you made a mistake during an incident?</b> · [SOC]</summary>

Say what you'd look at next and what you'd ask a second pair of eyes, set a time box, and escalate with "unsure, here's why" rather than closing it quietly. A mistake: say it immediately to the lead, state the impact, fix it, and put it in the write-up. Jane Street's hiring principles reward exactly this.
</details>

<details>
<summary><b>How would you tune a noisy rule? Measure a SOC? Design a detection for credential dumping?</b> · [SOC tier 3]</summary>

Tune: look at the last hundred fires, group by what made them false, add the exclusions that are safe (specific process paths, signed binaries), raise the threshold, and keep a metric of true positives per rule. Measure: time to detect, time to contain, alert precision, coverage against ATT&CK for the techniques that matter to you, and how many incidents came from outside the SOC. Credential dumping: Sysmon 10 process access to LSASS with specific access masks from non-system processes, `comsvcs.dll MiniDump` command lines, `procdump` on LSASS, Mimikatz strings in 4688, plus Credential Guard so the dump is empty.
</details>

<details>
<summary><b>An internal AI agent took an action it should not have (ran a command, touched production, moved data).</b> · [JS-flavoured: they run an agent fleet]</summary>

Treat the agent like a compromised-or-confused service account, and lean on the fact that their RPC makes AI-originated calls identifiable ([block 16](../16-ai-and-llm-security/README.md)). Contain: suspend that agent's identity and revoke its tokens and sessions, the same first move as any credential compromise. Scope: pull the agent's action log (every tool call under its "AI user" identity), find the trigger, and check specifically for **indirect injection**, was there untrusted content in its context (a document, a repo file, a tool description) that carried the instruction. Preserve the trace, including the prompt and retrieved context, because that is the forensic record here. Eradicate and recover: roll back whatever it changed (prompts, policies, data, tool schemas are all versioned), and if it reached production treat the blast radius like any unauthorised change. Then the prevention point they will want: tighten the allow-list and move the control to the safe-path-is-the-fast-path side so no one disables it, plus a regression test that replays the injection. This is the one incident type where you can speak to their exact architecture.
</details>

<details>
<summary><b>A market-hours incident: a strategy or system is sending bad or runaway orders.</b> · [trading-firm-specific; see block 17]</summary>

Availability and money are both live, so the order of operations differs from a corporate incident. First, **stop the bleeding with the kill switch**: pull the strategy or the session, which is a prepared control, not an improvisation ([block 17](../17-trading-domain/README.md)). Then establish what is in the market (open orders, fills, positions) and reconcile against what was intended, from the FIX and order logs. Decide containment versus continuity deliberately: halting all trading has its own cost, so you scope to the affected strategy or venue if you safely can. Communicate in parallel: the desk, risk, and compliance, because there may be a regulatory and venue-notification duty. Only once it is contained do you move to root cause, and the Knight Capital lesson frames it: assume a deploy or config change until proven otherwise, and check that every node is running the intended version. Recovery is a controlled, verified redeploy with the kill switch rehearsed. This is the incident that makes a trading firm different from a tech company, so naming the kill-switch-first order shows you understand the domain.
</details>

## The CEO briefing

Jane Street's Dec 2025 assessment asked for one. Practise writing it in five minutes from a technical note.

```text
Subject: Security incident, <date>, status at <time>

What happened:   An attacker got into <system> on <date> through <plain-words how>.
What they got:   <data or access>, and <what we don't yet know>.
What we did:     Contained at <time>. <Reset, isolated, preserved evidence>.
Business impact: <trading, clients, regulators, money>. <None so far / specifics>.
Risk now:        <Low / medium / high> because <one reason>.
Next 48 hours:   1. <action, owner>  2. <action, owner>  3. <action, owner>
Decisions needed: <e.g. notify client X, engage external forensics, inform the FCA>.
Next update:     <time>.
```

Rules: no acronyms, no tool names, numbers where you have them, uncertainty stated plainly, one page.

## Frameworks, in one line each

PICERL (SANS): preparation, identification, containment, eradication, recovery, lessons learned. Google's IMAG: incident commander, operations lead, communications lead, with explicit hand-offs, borrowed from emergency services. MITRE ATT&CK: the shared vocabulary for what the attacker did. Volatility order: network state, memory, disk, backups. Severity: data sensitivity, system criticality, spread, attacker capability, regulatory exposure.

[← Cloud and supply chain](../08-cloud-and-supply-chain/README.md) · [Index](../README.md) · [Next: Threat modelling →](../10-threat-modelling-and-design/README.md)
