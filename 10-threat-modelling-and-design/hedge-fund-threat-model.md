# Threat modelling a trading firm: what is different, and a reference model

[← Threat modelling hub](README.md) · [Frameworks](frameworks.md) · [Playbook](live-exercise-playbook.md) · [Worked models](worked-models.md) · [Questions](interview-questions.md)

Generic threat modelling finds generic threats. The threats that decide a trading firm's year are specific: a route that misses the risk check, a feed that lies, a payment instruction nobody verified out of band, a researcher's notebook with internet access. This file gives you the asset taxonomy, the actors and their current playbook, the incidents worth citing (and the one you should not), the firm-specific context for Jane Street, and a one-page reference model you can draw from memory. Tags: [JS] Jane Street primary, [est] established, [incident] a documented case, [analysis] reasoning.

---

## First, precision about what kind of firm this is

Jane Street is a proprietary trading firm and market maker trading its own capital, not a client-money hedge fund. That changes the asset list, and saying so in the room shows you have thought about *this* firm rather than "finance": there are no investor or limited-partner records to protect as a crown jewel, no fund administrator approving capital calls, and the regulatory surface is market-conduct and market-access rules rather than investor protection. What it shares with a hedge fund: the alpha is the business, the order path is where money moves, the people are the initial-access surface, and the firm's own treasury and payments are a BEC target. When the prompt says "hedge fund", model the fund-administrator and investor paths too (model 6 in [worked-models.md](worked-models.md) and the Levitas case below); when it says Jane Street, leave them out and say why.

---

## The asset taxonomy (what you protect, in order)

Say these before any threat. Interviewers at a trading firm listen for whether you know what the crown jewels are.

1. **Strategy and research: the alpha.** Strategy code, signals, research notebooks, datasets and the models trained on them. The firm's competitive edge in its most portable form. Loss is permanent and silent.
2. **The order path and its brakes.** Order gateways, pre-trade risk checks, the kill switch, exchange sessions and venue credentials. Where money moves, and where a bug or an attacker can move it wrongly in seconds ([block 17](../17-trading-domain/README.md)).
3. **Market data integrity.** The firm trades on what it sees; corrupted, delayed or spoofed data makes every downstream decision wrong while every system reports healthy.
4. **Positions, P&L and trading intent.** Knowing what a firm holds or is about to do is tradeable information for a competitor or a front-runner; inside the firm it is also the subject of information barriers.
5. **Identity and permissions.** The directory, the permissions service (Ontology at Jane Street, [worked-models.md](worked-models.md) model 1), privileged access, and now agent identities. Whoever controls these controls everything above.
6. **Treasury and payments.** Settlement instructions, counterparty and banking details, the approval chain for moving cash. The BEC target.
7. **The engineers' endpoints and the build pipeline.** Where the current initial-access playbook aims (below), and where a compromise reaches both the alpha and the order path.
8. **Evidence.** Order logs, FIX logs, audit trails, clock synchronisation. Needed to reconstruct an incident, and demanded by regulators; a tampered or unsynced record is a loss in itself.
9. **Availability during market hours.** Not an asset but a constraint that re-ranks everything: a control that halts trading at 14:00 may cost more than the attack it prevents.

---

## The actors and their current playbook

- **State-linked groups targeting financial infrastructure.** The 2026 Binance theft started with a tampered installer for an AI coding tool and a fake job-interview lure against developers, then moved from a DevOps engineer's laptop into a service container and out as unauthorised withdrawals, attributed to a North Korea-linked crew ([block 12](../12-security-news/README.md)). The lesson for a threat model: the developer endpoint is the entry point of choice, and the social lure is technical staff, not executives.
- **Financially motivated crime: BEC and payment redirection.** Needs no malware past the first step; targets the approval chain for money. See Levitas below.
- **Ransomware and extortion.** For a trading firm the lever is availability during market hours; the realistic path is a corporate-network foothold spreading to anything reachable, which is why segmentation between corporate, research and trading networks is the control that matters most.
- **Insiders.** Alpha theft on departure, barrier-crossing, a rogue strategy. Technical controls: least privilege and need-to-know on research, egress monitoring, barriers enforced in the permission layer, pre-trade limits that do not trust the trader.
- **Competitors and the grey market for information.** Trading intent and positions have buyers; the path is often a compromised third party or an insider rather than a direct breach.
- **Supply chain.** The dependency, the vendor, the data provider, the tool registry your agents load ([block 08](../08-cloud-and-supply-chain/README.md), [block 16](../16-ai-and-llm-security/README.md)).
- **Hacktivists and nuisance.** DDoS against public-facing surfaces; low on a trading firm's list because the order path is not on the internet, worth a sentence to show you have ranked it.

MITRE's Center for Threat-Informed Defense published a financial-sector profile mapping to ATT&CK in 2025, and MITRE's 2018 notional financial-services threat model is the older reference; both are useful to cite for "which techniques, from which actors" ([CTID](https://ctid.mitre.org/projects/threat-modeling-with-attack/), [ISADM paper references](https://arxiv.org/pdf/2512.18751)).

---

## Incidents worth citing, and one you should not

- **Levitas Capital, Sydney, September 2020 [incident].** A co-founder opened a fake Zoom invitation; the attackers gained control of the fund's email, then sent fraudulent invoices and a "capital call" to the fund's trustee and administrator, who approved payments. About $1.2 million left before the fraud was spotted; some $8 million in further transfers was stopped. The fund's largest investor withdrew and the $75 million fund closed. Red flags that were missed: the invoice was addressed to the fund rather than the trustee, and the fund had never made a capital call before ([SecureWorld](https://www.secureworld.io/industry-news/hedge-fund-closes-after-bec-cyber-attac), [ACS](https://ia.acs.org.au/article/2020/dodgy-zoom-invite-brings-down--75m-hedge-fund.html), [Nefture](https://medium.com/nefture/how-a-simple-email-forced-a-75m-hedge-fund-to-close-09ecebefdd0f)). The threat-model lesson: the **administrator and trustee payment chain is part of your system** even though none of it is your infrastructure, and the control is out-of-band verification of payment instructions, not a filter.
- **Structuring under the alert threshold [incident, as reported].** An advisory account of a hedge-fund compromise describes attackers who, having gained access to execute wire transfers, made a series of transfers each just under the firm's $500,000 flag level ([Baker Tilly](https://www.bakertilly.com/insights/five-best-practices-to-manage-hedge-fund-cybersecurity-risks)). The lesson generalises: a fixed detection threshold is an attack surface; detect on aggregates, velocity and new payees, not single-transaction size.
- **Knight Capital, 2012 [incident, not an attack].** A partial deploy reactivated dormant code and sent unintended orders for 45 minutes at a cost of about $440 million ([block 17](../17-trading-domain/README.md)). It belongs in a threat model because the order path does not care whether the bad orders came from an attacker or a flag; the controls (atomic deploys, pre-trade limits, a rehearsed kill switch) are the same.
- **The one not to cite as fact.** A widely repeated story from 2014 says attackers broke into a hedge fund's high-frequency trading system, slowed its order execution by milliseconds and exfiltrated its trade data. The firm that told it, BAE Systems, later said it was an illustrative scenario rather than a real client case ([Compliance Week](https://complianceweek.com/when-hackers-attack-hedge-funds/3597.article)). It still appears in vendor blogs as if it happened, including the "slowed by a few milliseconds" detail. Knowing this is worth more in a Jane Street interview than repeating it: say the scenario is plausible and worth modelling, and that you know its provenance. That is "know what you don't know" applied to threat intelligence.
- **Academic: adversarial perturbation of market data [analysis].** Researchers showed small, anomaly-detector-evading changes to a market data stream can steer algorithmic traders, under threat models including a man-in-the-middle on the broker feed and a malicious or compromised broker ([arXiv](https://arxiv.org/pdf/2010.09246)). Useful when you model feed integrity: the attacker does not need to break the feed, only to bend it.
- **Model extraction and poisoning against trading models [analysis].** Surveys of generative AI in finance flag model inversion and extraction (reconstructing a proprietary model or its training data through queries) and data poisoning as the threats specific to firms that deploy their own models ([arXiv](https://arxiv.org/pdf/2504.21574)). Relevant the moment a firm exposes a research model to many internal users or to agents.

---

## The abuse-case library for a trading firm

These are the business-logic threats that STRIDE will not hand you. Curate them, name them in the room, and you have done what the Threat Modelling posting describes: seen the exploit the system owner did not.

| Abuse case | Why it is specific to this business | The control shape |
| --- | --- | --- |
| A route to a venue that does not pass pre-trade risk | Money with no brake | Risk check as a property of the gateway framework; CI assertion on every route |
| A strategy runs away (bug, bad data, or malice) and the kill switch does not cover it | Minutes matter; Knight Capital | Kill-switch coverage tested per session; per-strategy limits that do not trust the strategy |
| Market data corrupted or delayed upstream, strategy trades on it | Everything downstream looks healthy | A/B feed arbitration, stale and crossed-book alerts, provenance on feeds |
| An information barrier crossed via a nested group, a shared cache or a report | Regulatory and competitive | Barriers as tests over derived access in the permission layer; data-layer enforcement |
| Trading intent or positions read by someone who can trade on them | Tradeable information | Need-to-know on positions and order flow; monitoring of bulk reads |
| Alpha exfiltrated from the research environment | Permanent, silent loss | Egress allow-lists, package mirrors, endpoint hardening, departure controls |
| A research signal promoted to production without independent review | Deployment to the order path by another name | Promotion through the same gates as code |
| Payment or settlement instruction changed by email | BEC; Levitas | Out-of-band verification that no email can override; DMARC at reject |
| Wire transfers structured under a fixed alert threshold | Thresholds are attack surface | Aggregate, velocity and new-payee detection |
| A vendor data feed poisoned to teach a false signal | Delayed, hard to attribute | Drift and distribution monitoring; provenance; low-weight signals monitored not trusted |
| Exchange or clearing credentials misused | Act as the firm at the venue | HSM or vault, short lifetimes, per-environment scoping, usage alerts |
| Clock drift or spoofing | Breaks sequencing and forensics; regulators require sync | Protected PTP sources, monitoring, the time source as an asset |
| An AI agent with human-equivalent permissions acts on an injected instruction | A new principal with old permissions | Sandbox, allow-list, identifiable RPC, deterministic authorisation ([block 16](../16-ai-and-llm-security/README.md)) |
| A deploy lands on some order-path hosts and not others | Knight Capital | Atomic deploys with version attestation |

---

## Jane Street specifics to bring into any model

All from their own public material; cite lightly, use heavily.

- **They build almost everything in-house**, in OCaml, in a monorepo. There is no vendor to blame and no vendor boundary to lean on; the trust boundaries are between their own teams' systems. Model-as-code fits that world, and typed RPC schemas make data-flow extraction tractable ([block 00](../00-jane-street-intel/README.md), [block 23](../23-ai-threat-modelling/README.md)).
- **Permissions run through Ontology**, a deductive database with an approval workflow and outward sync ([worked-models.md](worked-models.md) model 1). Any model that touches "who can do what" can reference it.
- **The security team builds its own tooling**: a SOAR, an email content-scanning system, the alerting stack, detection logic in code, with analysts owning the full detection lifecycle ([Detection and Response posting](https://www.janestreet.com/join-jane-street/position/7642974002/)). When you propose a detection as a verification, you are proposing something they would write themselves.
- **Data for hunting lives in a serious analytical store** (Superstore, a distributed columnar database, discussed on their podcast). Verification by query is realistic here.
- **An agent fleet with per-employee AI identities, an allow-list sandbox and identifiable RPC** ([block 16](../16-ai-and-llm-security/README.md)). Agents are principals in any current model of their systems.
- **The network is engineered in-house too** (EVPN, VXLAN, segmentation and DMZ work, a proxy layer), with the trading and corporate networks separated ([block 05](../05-networking/README.md)).
- **An offensive culture.** A Jane Street engineer's public talk walks through CTF-style exploitation of the OCaml runtime and mentions winning DEF CON CTF six times with his team ([Hacking OCaml](https://www.janestreet.com/tech-talks/hacking-ocaml)). Expect the interviewer to think like an attacker and to be unimpressed by controls that only stop the obvious variant.
- **They understand exchanges from the inside** (the "How to Build an Exchange" tech talk), so a trading-path model that is vague about matching, sessions and risk checks will be noticed.

---

## Regulation, kept technical

You do not need the rule numbers to be a strong candidate, and your own style forbids the register, but a few regulatory requirements are *technical controls* you should know by their shape: pre-trade risk checks and no unfiltered market access (the US market-access rule, [block 17](../17-trading-domain/README.md)); tested kill functionality and change control for algorithmic trading, with tight clock synchronisation (the European algorithmic-trading and timestamping rules); market-abuse surveillance of orders and communications; and incident-reporting duties that make your evidence trail a first-class asset. Say "the regulators require X, which is why control Y exists" and move on.

---

## One-page reference model of a trading firm

Draw this from memory when a prompt is "threat model a trading firm" or when you want a map to place a smaller system on.

| Crown jewel | Top threats (paths) | Control spine | How you would know |
| --- | --- | --- | --- |
| Alpha (research code, data, models) | Developer-endpoint compromise; notebook egress; dependency with a stealer; insider on departure | Segmented research environment, egress allow-lists, package mirror, need-to-know, departure process | Egress detections quiet; access reviews clean; drift alerts on feeds |
| Order path and brakes | Route bypassing risk; kill-switch gap; session hijack; bad deploy | Risk check as framework property; tested kill switch; private links or TLS; atomic deploys | CI assertions; kill-switch drills; sequence-anomaly detections |
| Market data | Upstream corruption or delay; perturbation | Feed arbitration; stale and crossed alerts; provenance | Alerts tuned against real noise |
| Identity and permissions | Approver manipulation; upstream write path; revocation lag; transitive over-grant | Protected approver rules; privileged upstreams; push revocation; effective-access diff | No grant without approval record; revocation SLA met |
| Treasury and payments | BEC; structuring | Out-of-band verification; DMARC reject; aggregate detection | Payment-change requests always verified; no single-path approvals |
| Evidence and time | Log tampering; clock drift | Append-only logs; PTP protection | Incident reconstructable end to end |
| Agents | Injected instruction with real permissions | Sandbox, allow-list, identifiable RPC, deterministic authz | Agent actions attributable; regression suite on injections |

---

## Using this in a generic prompt

If the interviewer gives you a generic system, bring the firm in with one sentence per step: name the asset in their taxonomy it touches, name the actor who would want it, and name the business-logic threat generic modelling would miss. That is the difference between a candidate who can threat-model and a candidate who can threat-model *here*.

[← Threat modelling hub](README.md) · [Frameworks](frameworks.md) · [Playbook](live-exercise-playbook.md) · [Worked models](worked-models.md) · [Questions](interview-questions.md)
