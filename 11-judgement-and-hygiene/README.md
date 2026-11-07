# 11 · Judgement, trade-offs and personal hygiene

[← Threat modelling](../10-threat-modelling-and-design/README.md) · [Index](../README.md) · [Next: Security news →](../12-security-news/README.md)

Jane Street's postings say it plainly: they want someone who weighs context and trade-offs, who practises good personal security and can explain their choices, and who knows that "not everyone should lock down their entire life in the name of security" ([Greenhouse](https://job-boards.greenhouse.io/janestreet/jobs/8057310002)). These questions have no single right answer; they're scored on whether you hold a reasoned position under pushback. Answer in three beats: the answer, the trade-off, what you'd do here. Tags: [JS] Jane Street, [DM] Miessler, [TW] tadwhitaker, [GN] gracenolan.

---

## The personal-setup questions

These come up in almost every Jane Street security loop, and the reports show interviewers pushing on them ([Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm)).

<details>
<summary><b>"I don't want 2FA. My passwords are unique and I spot phishing."</b> · [JS, New York]</summary>

Agree with the true part: unique passwords kill credential stuffing, the most common takeover. The gap: a site leaks your password in its own breach, an infostealer grabs it regardless of your judgement, and a good proxy phish fools careful people on a tired day. 2FA covers exactly the cases where the password is already gone. Then measure it: a forum can skip it; email, the password manager, banking and anything that can reset other accounts can't, because they're the keys to everything else. Offer the trade: "Put a passkey on your email and your manager and I won't ask about the rest." You keep your position, you don't moralise, and you let the person choose where it's cheap.
</details>

<details>
<summary><b>"Talk me through your own security setup."</b> · [JS theme]</summary>

Fill this in with your real setup before the interview, then be ready to defend each line and name one trade-off you accepted on purpose.

| Area | What I use | Why | Trade-off I accept |
| --- | --- | --- | --- |
| Password manager | | | |
| MFA on email and the manager | | | |
| Phone (updates, SIM PIN, carrier lock) | | | |
| Laptop (disk encryption, updates, admin rights) | | | |
| Backups (3-2-1, tested restore) | | | |
| Browser and extensions | | | |
| Home network (guest/IoT split, DNS) | | | |

The interviewer isn't checking that you're maximally locked down; they're checking that each choice was a decision. "I use SMS 2FA on my airline account because the threat there is low and I value not getting locked out mid-trip" is a better answer than claiming hardware keys on everything.
</details>

<details>
<summary><b>Is a notebook of passwords ever acceptable?</b> · [JS theme] [DM: "reasonable alternative"]</summary>

Yes, for the right person. For a non-technical parent the real threat is remote attackers, not a burglar reading a notebook, so a notebook of unique written passwords beats one reused password everywhere. Security advice has to fit the threat model of the person using it. For an engineer with a laptop full of secrets, different answer.
</details>

<details>
<summary><b>What makes a password strong, and how do you estimate complexity? How should a password manager store and key the vault?</b> · [JS Hong Kong]</summary>

Both answered in full in [block 04](../04-security-fundamentals/README.md). The short version: length and uniqueness beat character classes, entropy over the generation method not the appearance, and the hash speed matters as much as the password; the vault is ciphertext-only under AES-GCM or XChaCha20-Poly1305, the key derived from the master password with Argon2id through a key hierarchy, zero-knowledge so the server never sees it.
</details>

<details>
<summary><b>TOTP, push, or hardware keys? Rotating passwords on a schedule?</b> · [JS Hong Kong] [GN]</summary>

TOTP and push can be relayed by a phishing proxy, and push gets fatigue-spammed unless number matching is on; FIDO2 keys and passkeys bind to the origin and can't be phished, so they go on admins and money-moving roles first. Scheduled rotation is harmful: it produces `Password1`, `Password2` and help-desk load without stopping same-day use of a stolen password, so rotate on evidence, screen against breach lists, and use MFA (NIST 800-63B agrees).
</details>

---

## The organisational-judgement questions

<details>
<summary><b>What's the goal of information security in an organisation?</b> · [DM]</summary>

To help the organisation succeed while keeping risk at a level the business has chosen to accept. "Lock everything down" is the junior answer; security that stops the firm trading has failed at its job. Jane Street's framing is that security is a business enabler, which is your "paved roads, not gates" line: make the secure path the fast path and adoption follows.
</details>

<details>
<summary><b>Threat, vulnerability, risk: the difference? Which matters more for corporate infosec?</b> · [DM] [TW]</summary>

A vulnerability is a weakness, a threat is an actor or event that could exploit it, risk is the combination weighted by likelihood and impact. Which matters more is an opinion question: Miessler leans vulnerabilities (you control those; you can't control who attacks you), and a defensible counter is that threat modelling tells you which vulnerabilities to fix first. Hold a position and give the other side.
</details>

<details>
<summary><b>If you were made CISO of a Fortune 500 tomorrow with no prior knowledge, what are your first priorities?</b> · [DM]</summary>

Quick prioritisation beats a framework recital: find where the important data is and who can touch it, get network and asset diagrams, check what visibility exists (logging, EDR coverage, CloudTrail), ingress and egress filtering, prior assessments and their unfixed findings, identity hygiene (MFA, admin sprawl), and backups you've actually restored. Then fix the highest risk-to-effort items first and build a plan for the rest. The signal is that you triage before you buy tools.
</details>

<details>
<summary><b>Are insiders or outsiders more dangerous? Open-source or proprietary more secure?</b> · [DM]</summary>

Both are "it depends, and here's how I'd decide". Insiders: ask what "dangerous" means, likelihood or impact; insiders are rarer but have access and knowledge, so impact is higher, which argues for least privilege and audit over trust. Open-source versus proprietary: size, contributor quality and QA matter more than the licence; "many eyes" is weak on its own (Heartbleed sat in OpenSSL for years). The move is to refuse the false binary and name the factors.
</details>

<details>
<summary><b>Why haven't most companies fixed their vulnerabilities? How would you measure a security team?</b> · [DM]</summary>

Fixing costs money, time and friction, and security competes with features, so most firms accept more risk than they admit; your job is to make the cost of fixing lower (paved roads) and the cost of not fixing visible. Measuring a team: ask what kind of team first; good metrics are time to detect and contain, patch latency on exploited vulnerabilities, coverage of the techniques that matter, and how much came from outside the team, not raw alert counts (which reward noise).
</details>

<details>
<summary><b>Where do you get your security news, and who do you look up to?</b> · [DM]</summary>

Name real sources you actually read (two or three, not ten) and why; see [block 12](../12-security-news/README.md). For people, naming practitioners whose work you can discuss beats a list; and the posting's own line is that you can tell a cool hack by researchers from what matters day to day, so say which recent story you'd act on and which you'd note and move past.
</details>

<details>
<summary><b>What advantages do bug bounties have over normal testing?</b> · [DM]</summary>

Many testers with varied skills, paid on results so incentives point at real bugs, continuous rather than a point-in-time snapshot, and good at finding the rare and weird. Trade-offs: noise and triage load, no guaranteed coverage, and it doesn't replace a scoped pentest or secure design. Run both.
</details>

<details>
<summary><b>What systems do you run at home to tinker with?</b> · [DM]</summary>

They're checking for genuine enthusiasm. A home lab, a few VMs, a CTF habit, a side project, something you broke and fixed. "I get enough computers at work" is the answer Miessler calls a red flag. Your Threat-modeling.io work and your home setup both answer this.
</details>

---

## The Jane Street-specific judgement

The posting says you must weigh trade-offs and that security is a business enabler. Translate every answer in this block into their world: a control that slows a trader by a millisecond is a different decision from one that slows a back-office batch job; a lockdown that makes engineers route around it is worse than a lighter control they actually use; the measured answer ("here's where I'd be strict, here's where I'd accept risk, and here's how I'd know if I was wrong") is the one that matches how they describe the team.

Drop the bank vocabulary when you answer. Not "I'd implement a governance framework to ensure stakeholder alignment on risk appetite"; say "I'd find out what the firm can't afford to lose, lock that down hard, and leave the rest lighter so people don't fight me."

[← Threat modelling](../10-threat-modelling-and-design/README.md) · [Index](../README.md) · [Next: Security news →](../12-security-news/README.md)
