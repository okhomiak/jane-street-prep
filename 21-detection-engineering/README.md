# 21 · Detection engineering, threat hunting and email security

[← Gap analysis](../20-gap-analysis/README.md) · [Index](../README.md)

This block exists because the postings describe the job and the repo did not cover it as a discipline. The Senior Cybersecurity Operations Engineer posting names the stack: EDR and SIEM for triage, Elasticsearch and ElastAlert, network security monitoring tools, security orchestration and "self-healing" systems ([posting](https://www.janestreet.com/join-jane-street/position/8687509002/)). The Detection and Response posting wants "projects to improve our detection capabilities, automation and efficiencies" and threat hunts ([posting](https://www.janestreet.com/join-jane-street/position/7642974002/)). The SOC internship posting says investigators trace chains of activity using mostly in-house or open-source log sources and tools ([posting](https://www.janestreet.com/join-jane-street/position/8632723002/)); read that as: you will not be handed a vendor console, you will build and query. So expect questions about *building* detection, not just reading alerts. Tags: [JS] Jane Street primary, [SOC] SOC question banks, [est] established practice.

---

## Detection as an engineering discipline

<details>
<summary><b>How do you go from "we should detect X" to a rule in production?</b> · [est] [the lifecycle interviewers want to hear]</summary>

Treat it like shipping software:

1. **Hypothesis.** Name the behaviour (an ATT&CK technique, or an attack path from a threat model) and the evidence it leaves. "Kerberoasting leaves 4769 events with RC4 encryption in a mostly AES domain."
2. **Data.** Check you actually have the log, at the right verbosity, with the fields you need, from all the hosts that matter. Most detection failures are missing data, not bad logic. Confirm the field is populated and the clock is right.
3. **Logic.** Write the rule as narrowly as the behaviour allows and as broadly as the attacker's variations require. Prefer behaviour over indicators (see the pyramid of pain below).
4. **Test.** Replay known-good and known-bad data. Atomic tests (Atomic Red Team) or your own simulation generate the true positive; a week of production traffic shows the false positives.
5. **Tune.** Add safe exclusions (specific signed binaries, known admin hosts), not blanket ones. Record why each exclusion exists.
6. **Deploy with a runbook.** Every alert says what it means, what to check first, and what to do. A rule without a runbook is noise with a name.
7. **Measure and retire.** Track fires, true-positive rate, and time to triage per rule. Rules that never fire or never land are candidates to fix or delete.

Say the one line that shows judgement: a detection's cost is paid in analyst minutes on every false positive, so precision is a budget, not a nice-to-have.
</details>

<details>
<summary><b>Write a detection rule. What does Sigma look like, and how does it get to Elastic or ElastAlert?</b> · [JS stack] [est]</summary>

Sigma is the portable, vendor-neutral format: YAML describing a log source and a condition, converted into the target's query language. A rule for the classic phishing chain, Office spawning a script host:

```yaml
title: Office application spawning a script interpreter
status: experimental
logsource:
  category: process_creation
  product: windows
detection:
  selection_parent:
    ParentImage|endswith:
      - '\WINWORD.EXE'
      - '\EXCEL.EXE'
      - '\OUTLOOK.EXE'
  selection_child:
    Image|endswith:
      - '\powershell.exe'
      - '\wscript.exe'
      - '\cscript.exe'
      - '\mshta.exe'
  condition: selection_parent and selection_child
falsepositives:
  - Legitimate macros in finance workflows (allow-list by signed macro or host)
level: high
tags:
  - attack.execution
  - attack.t1059
```

Converters (pySigma, sigma-cli) turn that into a KQL/Lucene/EQL query for Elasticsearch, a Splunk search, or a Microsoft Sentinel rule. **ElastAlert** then runs queries against Elasticsearch on a schedule and alerts. Its rule types are the vocabulary to know: `any` (match at all), `frequency` (N matches in a window, the brute-force detector), `spike` (volume up or down relative to a baseline), `flatline` (a source went quiet, which is how you notice a dead log forwarder or a disabled agent), `blacklist`/`whitelist`, `new_term` (a value never seen before, like a new admin account or a new country), `cardinality` (too many distinct values, like one account hitting many hosts), and `change` (a field changed for an entity). If you can map a detection idea to one of those, you can say exactly how you would build it on their stack.
</details>

<details>
<summary><b>Indicators versus behaviours: the pyramid of pain, and why it decides what you build.</b> · [est] [SOC classic]</summary>

David Bianco's pyramid ranks indicators by how much it costs the attacker when you detect on them. Bottom, trivial to change: hash values, then IP addresses, then domain names. Middle: network and host artefacts (a user-agent string, a registry key, a mutex). Top, expensive to change: tools, and above them tactics, techniques and procedures. Detecting on a hash catches one sample once; detecting on "Office spawning a script host" catches the technique however the payload changes. So the build order is behaviours first, indicators for fast wins and threat-intel feeds, and you measure coverage against ATT&CK techniques rather than against a count of rules. A heat map of techniques you can see, techniques you can detect, and techniques you have no data for is the artefact that shows a mature programme.
</details>

<details>
<summary><b>Tuning and the economics of false positives.</b> · [SOC tier 3] [est]</summary>

The base-rate problem is the thing to say out loud: if a detector is 99% accurate and the true event is one in ten thousand, almost every alert is a false positive (the maths is in [back-of-envelope](../04-security-fundamentals/back-of-envelope.md)). That is why a "99% accurate" classifier is not a control on its own. Tuning moves: tighten the condition on the attacker's invariant (the thing they cannot avoid) rather than on the noise; add context from asset inventory so the same event on a domain controller and a kiosk score differently; aggregate (alert on the user or host, not every event); suppress known-good with specific, documented exclusions; and route low-confidence signals to a hunt queue instead of the pager. Measure per rule: fires per day, true-positive rate, median triage time. Delete or fix anything with a true-positive rate near zero that costs real minutes.
</details>

<details>
<summary><b>How do you measure a detection programme or a SOC?</b> · [SOC] [est]</summary>

Outcome metrics first: mean time to detect, mean time to contain, and how many incidents were found by your own detections versus by a user, a vendor or a third party (the embarrassing one). Then quality: alert precision, percentage of alerts with a runbook, ATT&CK coverage for the techniques in your threat model, log coverage (which hosts and sources are actually reporting, and the `flatline` rule that tells you when one stops). Then health: analyst time per alert, backlog age, and how much of triage is automated. Avoid vanity counts (number of rules, number of alerts), and say so.
</details>

---

## Threat hunting

<details>
<summary><b>What is threat hunting, and how do you run a hunt?</b> · [JS D&R posting: "threat hunts"] [est]</summary>

Hunting is proactively searching for activity your detections missed, driven by a hypothesis, not by an alert. A hunt in five steps: pick a hypothesis from threat intel or a gap in your coverage map ("an attacker who phished a developer would use their SSH key to reach build hosts"); identify the data that would show it (auth logs on build hosts, SSH `Accepted publickey` lines, source IPs); query and stack (frequency analysis: rare source IPs, rare user-to-host pairs, first-seen combinations); investigate the outliers to a verdict; and, whatever you found, **turn the hunt into a detection** so you never hunt the same thing twice. The last step is what separates a hunt from a fishing trip, and it is the answer to "what did the hunt produce" even when it found nothing.

Useful hunting techniques to name: stacking (count distinct values and look at the rare tail), least-frequency analysis across the fleet (the one process name that appears on one host), first-seen (new binaries, new admin accounts, new external destinations), and baselining a user or host and diffing against it.
</details>

<details>
<summary><b>How do you consume threat intelligence without drowning in it?</b> · [est]</summary>

Separate strategic (who targets firms like us and how, which shapes the threat model) from tactical (TTPs, which become detections and hunts) from operational (indicators, which go into automated feeds with an expiry). For a trading firm the strategic answer is specific: financially motivated and state-linked groups that target financial infrastructure, with supply chain and developer-targeted lures as the current initial-access pattern (the 2026 Binance chain started with a tampered installer and a fake job interview, [block 12](../12-security-news/README.md)). Indicators have a shelf life; TTPs do not. Judge a feed by how often its indicators actually match your traffic, and retire the ones that never do.
</details>

---

## Network security monitoring

<details>
<summary><b>Zeek, Suricata, NetFlow: what each gives you.</b> · [JS posting: "network security monitoring tools"] [est]</summary>

**Zeek** turns packets into structured logs: `conn.log` (every connection with duration, bytes, state), `dns.log`, `http.log`, `ssl.log` (SNI, certificate, JA3/JA4 fingerprint), `files.log`, `notice.log`, all joined by a connection `uid`. It is the hunting dataset: you ask questions of it. **Suricata** is signature and anomaly detection on the wire (IDS, or IPS inline), emitting EVE JSON alerts; it is the rule engine. **NetFlow/IPFIX** is cheap metadata from routers (who talked to whom, how much, when), no payload, good for scale and history. In a latency-sensitive trading environment you deploy all of these from a tap or mirror port, never inline on the order path ([block 17](../17-trading-domain/README.md)).
</details>

<details>
<summary><b>How do you detect a beacon, and how do you detect DNS tunnelling?</b> · [SOC: "one host beaconing"] [est]</summary>

**Beaconing** is periodic check-in to a command-and-control server. The signal is regularity: group connections by source and destination, compute the gaps between them, and look for a tight distribution (low variance, or a fixed interval plus small jitter), consistent small payload sizes, long-lived relationships to a destination nobody else talks to, and a young or low-reputation domain. Attackers add jitter, so look at the distribution, not an exact period. Also useful: JA3/JA4 fingerprints that match known tooling, and TLS to a bare IP or a self-signed certificate.

**DNS tunnelling** hides data in queries. Signals: unusually long or high-entropy subdomains, a very high query volume to one domain, odd record types (TXT, NULL) in bulk, many unique subdomains under one parent, and a client talking to a resolver it should not. The control that makes detection easier is forcing all DNS through your own resolvers and blocking direct outbound 53 and DoH to unknown providers.
</details>

---

## Email security and phishing analysis

The ops posting names phishing, business email compromise and impersonation as threats the team handles; the departments page says employee education is a top priority and security is the last line of defence. Analysing a phishing email is one of the most repeated SOC interview tasks, and SPF/DKIM/DMARC is a rapid-fire staple.

<details>
<summary><b>Explain SPF, DKIM and DMARC, and what each actually stops.</b> · [SOC staple] [est]</summary>

- **SPF** is a DNS TXT record listing the servers allowed to send mail for a domain. The receiver checks the *envelope* sender (MAIL FROM / Return-Path) against it. Weaknesses: it checks the envelope, not the From header the user sees, and it breaks on forwarding because the forwarder's IP is not in the list.
- **DKIM** is a cryptographic signature added by the sending server over selected headers and the body; the public key lives in DNS at `selector._domainkey.domain`. It proves the message was not altered since signing and ties it to the signing domain. It survives most forwarding. Weakness: it says who *signed*, not that the From header is honest.
- **DMARC** ties them together: it requires **alignment**, meaning the domain in the visible From header must match the domain that passed SPF or the `d=` in a passing DKIM signature, and it publishes a policy (`none`, `quarantine`, `reject`) telling receivers what to do on failure. It also returns aggregate (`rua`) and forensic (`ruf`) reports so you can see who is sending as you.

The sentence that shows understanding: SPF and DKIM on their own do not stop someone spoofing your display domain; DMARC at `reject` with alignment is what does, and most domains sit at `p=none` for months because they are afraid of breaking legitimate mail.
</details>

<details>
<summary><b>Business email compromise: why it beats technical controls, and what you do about it.</b> · [JS posting names BEC] [est]</summary>

BEC usually has no malware and no malicious link, so gateways pass it. The patterns: a lookalike domain (`janestreet-llc.com`), a display-name spoof with a free-mail address, a Reply-To swap, a hijacked real thread from a compromised partner mailbox, and the ask is money movement or a change of payment details, often with urgency and an executive's name. Controls are layered and the decisive one is not technical: DMARC at reject on your own domain, external-sender tagging, lookalike-domain monitoring and registration, impersonation rules in the gateway for executive names, and above all an **out-of-band verification rule for any payment or banking-detail change** that no email can override. For a trading firm, say that payment and settlement instructions are the crown-jewel version of this, and the process control matters more than the filter.
</details>

<details>
<summary><b>Here is a suspicious email. Analyse it.</b> · [SOC: one of the most repeated tasks]</summary>

Work the headers top-down in the order mail travelled: `Received:` hops (bottom is oldest; find the first external hop and its IP), `Return-Path` versus `From` versus `Reply-To` (mismatches are the tell), the `Authentication-Results` header for SPF/DKIM/DMARC verdicts, `Message-ID` domain, and the date versus the hops. Then the content: the real link target behind the display text, the domain's age and registrar, URL shorteners and redirects, attachments by hash and type (macro-enabled Office, ISO/IMG, HTML smuggling, a PDF with a link), and the lure's intent (credentials, payment, malware). Then scope: who else received it (search the gateway), did anyone click or enter credentials (proxy logs, identity sign-ins), and act: block the sender and domain, purge from mailboxes, reset credentials if entered, and feed the indicators and the technique back into detections. Narrate the hypothesis as you go; the exercise is scored on method.
</details>

---

## Reported questions, with sources

| Question | Where | Source |
| --- | --- | --- |
| Threat hunts; projects to improve detection and automation | [JS] Detection and Response posting | [janestreet.com](https://www.janestreet.com/join-jane-street/position/7642974002/) |
| Elasticsearch, ElastAlert, EDR, SIEM, network security monitoring, self-healing systems | [JS] Senior Cyber Ops Engineer posting | [janestreet.com](https://www.janestreet.com/join-jane-street/position/8687509002/) |
| Trace chains of activity with mostly in-house or open-source log tooling | [JS] SOC posting | [janestreet.com](https://www.janestreet.com/join-jane-street/position/8632723002/) |
| Phishing, BEC, ransomware, supply chain, impersonation, insider as named threats | [JS] Senior Cyber Ops Engineer posting | same |
| Windows event IDs; how to tune a noisy rule; measure a SOC; design a detection for credential dumping | SOC banks | [epicdetect](https://epicdetect.io/blogs/soc-analyst-interview-questions), [rootguard](https://rootguard.gitbook.io/cyberops/resources-hub/blogs/posts-field-notes/soc-prep-interview-questions) |
| One host beaconing to an unknown domain; a user clicked a phishing link | SOC scenario banks | [SIEM XPERT](https://www.siemxpert.com/blog/top-25-soc-analyst-scenario-based-interview-questions-in-2026/) |
| Design a monitoring system to detect anomalies in network traffic; design a SIEM | design rounds | [block 10](../10-threat-modelling-and-design/README.md) |

Where this meets your own work: detection-as-code is the same discipline as your scanning pipeline and remediation bot, rules in version control, tested before deploy, measured after. Say that, and say which `ElastAlert` rule type you would reach for when they describe a behaviour.

[← Gap analysis](../20-gap-analysis/README.md) · [Index](../README.md)
