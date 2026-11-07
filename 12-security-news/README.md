# 12 · Security news

[← Judgement and hygiene](../11-judgement-and-hygiene/README.md) · [Index](../README.md) · [Next: Behavioural →](../13-behavioural/README.md)

The posting wants "an avid follower of cybersecurity news" who can "tell the difference between the latest cool hack by security researchers and what actually matters in our day-to-day" ([Greenhouse](https://job-boards.greenhouse.io/janestreet/jobs/8057310002)). So this round isn't a quiz; it's a ranking exercise. Bring a few stories, lead with the one that matters most for a trading firm, and name one that's a big headline but mostly noise for them. Keep the dates current; replace these the week of your interview.

How to talk about one story in 60 seconds: what happened in one sentence, the one lesson, then what you'd check at Jane Street tomorrow.

---

## Five stories from autumn 2026, ranked for a trading firm

*Top two stories re-verified against current reporting on 7 November 2026. Refresh again the week of your interview.*

### 1. Binance, ~$6.3bn stolen through an engineer's laptop (29 Oct 2026): matters most

On 29 October 2026, starting at 20:05 UTC, an attacker drained about $6.36bn from Binance hot wallets across ten blockchains, most of it in roughly 20 minutes; Binance's entry point was a compromised DevOps engineer's laptop that led to a service container able to submit withdrawals, and no wallet private key was found to have leaked ([BleepingComputer](https://www.bleepingcomputer.com/news/security/binance-links-63-billion-crypto-theft-to-compromised-devops-laptop/)). Coverage of the 3 November report describes the chain: a tampered installer in June, a fake-job "assignment" repo in September, a pivot through an internal AI tool, then a stolen container deploy token driving the signing service ([AirdropAlert](https://airdropalert.com/blogs/binance-hack-post-mortem/)). TRM Labs and Chainalysis attribute it to North Korea's TraderTraitor; about $2.7bn was frozen.

**One lesson:** no cryptography broke. The attacker got the ability to ask the signer nicely, and the signer checked that the signature was valid, not that the decision was. **At Jane Street tomorrow:** which laptops hold a token that can deploy to anything near the order or payment path, and does anything that signs or sends check the decision independently, not just the credential. Also: engineer-targeted recruiting lures are a real initial-access channel.

### 2. LiteLLM 1.107.0 poisoned release (21 Oct 2026): high

On 21 October 2026, LiteLLM's release pipeline was compromised: attackers used stolen GitHub tokens, one belonging to an org owner, to repoint the release workflow at a lookalike repository and publish a malicious v1.107.0. Each wheel dropped a `.pth` file, which Python runs at every interpreter start, so the credential stealer ran whenever Python started in that environment, even if LiteLLM was never imported. It was live on PyPI for under two hours and on Docker tags longer; the private enterprise release was not affected. It was LiteLLM's second supply-chain compromise of 2026.

**One lesson:** a Python-heavy shop that doesn't pin and mirror can be compromised in the two hours a bad release is live. **At Jane Street tomorrow:** hash-pinned dependencies, an internal mirror with a cooldown before new releases are allowed, no outbound internet from research and build hosts, and short-lived credentials so a stolen one dies fast.

### 3. eMudhra weak CA keys (17 Oct 2026): low, but one real action

eMudhra revoked eight intermediate CA certificates after a researcher found their private keys were weak enough to recover almost instantly; one published key had the value 4. Mozilla and Google blocked them on 17 October, Apple on 22 October, and no malicious use was found. Software with its own trust store, including macOS's built-in curl, could keep trusting them until updated ([BleepingComputer](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates)).

**One lesson:** browsers self-healed; anything with its own certificate store didn't. **At Jane Street tomorrow:** inventory what validates certificates outside the OS trust store (Go services, pinned clients, appliances) and confirm they got the update.

### 4. ConnectWise Cloud Backup OAuth abuse (16 Oct 2026): medium

ConnectWise disclosed on 16 October that an attacker abused application identities that already had consent to read customer Microsoft 365 tenants; sign-ins hit almost 1,000 tenants and over 300 succeeded, and in one the app granted itself Global Administrator ([DEV Community](https://dev.to/ashtech52/cyber-attacks-october-2026-the-biggest-incidents-exploited-zero-days-and-what-to-patch-this-week-2ibh)).

**One lesson:** a third-party app with tenant-wide consent is a standing key that never shows up as a login you watch. **At Jane Street tomorrow:** audit every OAuth app with broad Graph permissions, and alert on an app escalating its own privileges.

### 5. PHP2Shell, two critical PHP RCEs (21 Oct 2026): mostly noise here

PHP shipped emergency fixes on 21 October for two remote code execution flaws (CVE-2026-109618 and -109619, CVSS 9.2), with exploitation attempts seen four days before the fix and over 5 million PHP hosts reachable online ([DEV Community](https://dev.to/ashtech52/cyber-attacks-october-2026-the-biggest-incidents-exploited-zero-days-and-what-to-patch-this-week-2ibh)).

**One lesson:** huge blast radius across the web, but it's a PHP story. **At Jane Street tomorrow:** this isn't a PHP shop, so the action is narrow: check vendor appliances and any small internal tool that quietly runs PHP, and move on. This is the one I'd name as "big headline, low relevance to us", which is exactly the judgement the posting asks for.

---

## Themes worth being able to discuss

- **Identity is the perimeter.** Binance, ConnectWise, and most 2026 cloud breaches are credential and token misuse, not memory-corruption exploits. The defences are MFA that can't be phished, short-lived credentials, and least privilege.
- **The software supply chain is a primary target.** npm and PyPI have had a steady run of worm-like and maintainer-takeover attacks through 2025-26 (Shai-Hulud and its variants, Axios, the tj-actions GitHub Action compromise). SBOMs, pinning and provenance are the answers.
- **AI tooling is now in the attack path.** The Binance chain reportedly pivoted through an internal AI tool, and LiteLLM is AI infrastructure; agent and MCP security is a live research area, which is your territory (the sandboxed multi-agent DAST verifier and Threat-modeling.io).
- **Edge devices keep getting popped.** Ivanti, Palo Alto, Citrix, Fortinet, BeyondTrust all had actively-exploited floods into CISA's KEV catalogue in 2026. The lesson is to patch the edge fast and not expose management interfaces.

---

## Where to actually read (name two or three, not ten)

Original and high-signal: vendor and project advisories (the source, not the aggregator), CISA KEV for what's actively exploited, Google Threat Intelligence and Mandiant, Project Zero, company security blogs (Cloudflare, Trail of Bits). Daily digest: BleepingComputer, The Record, Risky Business, tl;dr sec. Community: a curated Mastodon or X list, r/netsec. Say which two you read every week and why; the posting rewards the person who filters, not the one who firehoses.

> Keep this file fresh. The week before your interview, replace any story older than a month with the current top items from CISA KEV and one good digest, and re-rank them for a trading firm.

[← Judgement and hygiene](../11-judgement-and-hygiene/README.md) · [Index](../README.md) · [Next: Behavioural →](../13-behavioural/README.md)
