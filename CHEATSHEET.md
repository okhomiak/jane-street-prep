# Cheat sheet: the hour before any Jane Street round

[Index](README.md) · [Question bank](QUESTION-BANK.md) · [Sources](SOURCES.md)

One page. Every line points to the block that backs it. If something here surprises you, read the block, not this.

---

## Jane Street in eight lines [00]

- Cybersecurity describes itself as programmers and security experts who cover for each other. Programmers first.
- They build almost everything in-house and write what they can in OCaml. "Here's what I'd build" beats "here's what I'd buy".
- Values you can show: intellectual humility, little weight on seniority, blameless postmortems, teaching.
- Interview principles, their words: be nice, be clear, know your language, know what you don't know.
- The role signal: much of the environment is Windows, with automation expected. Know AD, Entra, EDR and PowerShell well enough to talk.
- AI: AIDE on every developer machine, JaneGPT with a vendor switcher, agents in allow-list sandboxes (bwrap, pasta, seccomp), an AI counterpart identity per person, AI-originated RPC calls made identifiable.
- What they've seen go wrong: agents escaping notebooks, working round permissions, reward hacking, engineers switching sandboxes off.
- Office: at least three days a week in London, per the recruiter.

## The loop [01]

Recruiter call → online assessment (shell and logs, then a briefing) → phone screen (culture, your questions) → live investigation (one hour, harder) → coding (one hour, your language) → final block (fundamentals, scenarios, design, news, hiring manager). Confirm the order with the recruiter.

## The recruiter call [26]

- **Why Jane Street, 30 seconds:** programmers who do security; teaching and humility; agents in sandboxes with their own identity, which is your field.
- **Why leave:** at a bank's scale the job drifts to coordination; you want your hands on the problem.
- **Applied before:** yes, one clause, then what changed.
- **Salary:** let them go first; if pressed, a total, not a range.
- **Notice:** exactly the contract; garden leave possible; deferred awards raised now.
- **Last question:** "Is there anything in my background you'd like me to expand on before it goes to the team?"

## Log investigation: the six steps [02]

Orient (what is each file) → baseline (what's normal) → find the rare → pivot on one indicator across every log → timeline → write up. Say what you expect before each command. Five-line note: finding, evidence, scope, do now, check next. Then the CEO version in under 120 words: what happened, what they got, what we did, risk now, next 48 hours, what we need.

## Coding: the hour [03]

Ask five questions first (line format, sorted?, malformed lines?, what counts as brute force, what to return). Say the plan in two sentences. Simplest version, three made-up lines, then grow it. Test out loud: happy path, malformed line, a success between failures. Name the complexity and the next step unprompted.

## The five Jane Street questions [04]

Password strength and how to estimate it · how a password manager stores secrets and where the key comes from · TOTP and clock drift · the data flow of opening a site and logging in, POST or GET · "I don't want 2FA". Go one layer deeper than asked: mechanism, then attack, then trade-off.

## Scenarios: the skeleton [09]

Confirm → scope → contain proportionally → preserve → eradicate and recover → learn → communicate. Say the trade-off every time.

## News, two stories [12]

1. Binance, October 2026: an engineer's laptop led to a container that could ask the signer to withdraw. The signer checked the signature, not the decision.
2. LiteLLM, March and October 2026: the model-gateway library poisoned twice; the October release used a `.pth` file that ran at every Python start. Pin, mirror with a cooldown, short-lived credentials.

## Current, 2026 [16, 04, 08]

MCP revision 2026-07-28 (tool name in headers) · enterprise-managed MCP authorisation stable since June · Microsoft's red team: the human approval step is the most-bypassed control · model API keys can be federated with a workload's JWT · an npm worm planted agent configuration files · caches in front of access decisions caused 2026 CVEs in OpenFGA, Vault and Istio.

## Say this

- "I'd build it, and here's the smallest version that would work."
- "I don't know. Here's how I'd find out."
- "Here's the trade-off, and here's what I'd skip."
- "Approval is never the first control. The sandbox already said no to most things."

## Don't say this

- Your title, first or at all.
- "Governance", "framework", "stakeholder alignment".
- A vendor name as an answer.
- "It depends", without then deciding.

## Ask them [13]

1. What has the team built for itself lately, rather than bought?
2. How do you decide when a control is worth the friction it adds?
3. How are you thinking about the security of the AI tools your engineers use?
4. What are the worst parts of the job?

## If you're stuck

Say what you know, say what you'd check, pick the next pivot out loud. Silence reads as lost; a plan read aloud reads as an engineer.
