# 17 · The trading domain for a security engineer

[← AI and LLM security](../16-ai-and-llm-security/README.md) · [Index](../README.md) · [Next: OCaml literacy →](../18-ocaml-literacy/README.md)

Jane Street is a trading firm, and the security questions have a trading accent. Production engineers get "troubleshoot trading-system latency" and "FIX logs and matching-engine setup" ([Glassdoor](https://www.glassdoor.ie/Interview/Jane-Street-Interview-E255549-RVW82145680.htm)); Optiver asks "how would you connect your new startup to the French exchange" in its security-engineer loop ([Glassdoor](https://www.glassdoor.com/Interview/Optiver-Interview-E243355-RVW44773253.htm)); Jump asks support engineers "what is a crossed book" ([Glassdoor](https://www.glassdoor.co.nz/Interview/Jump-Trading-Interview-E251744-RVW91825140.htm)). You do product security for a greenfield cross-asset trading platform, so this block is yours to own: it gives you the vocabulary to make every generic security answer land in their world. Tags: [JS] Jane Street, [GD] Glassdoor reports, [est] established industry fact.

---

## Why a trading firm's security is genuinely different

Say this framing early if a design question is trading-flavoured. Three things change the usual playbook:

1. **Latency is a security constraint.** On the order path, microseconds are the product. You cannot drop an inline deep-packet-inspection firewall or a TLS-terminating proxy into the hot path, because the inspection *is* the latency. So controls move off the critical path: strict network segmentation and physical isolation instead of inline inspection, validation at the gateway endpoints, and monitoring from a tap or mirror port rather than a bump in the wire.
2. **Availability during market hours is the top-line risk.** A control that fails closed and halts trading at 2pm can cost more than the attack it prevented. Every control needs a blast-radius and a failure-mode answer. This is the measured-judgement culture ([block 11](../11-judgement-and-hygiene/README.md)) with money attached.
3. **The crown jewels are specific.** Not "customer PII". They are the strategy and research code (the alpha), the market-data feeds, the order-entry path to the exchanges, and the credentials and certificates that authenticate the firm to venues and clearers. Name those as the assets when you threat-model.

---

## The stack, in the words they use

<details>
<summary><b>Market data, order entry, matching engine: the path of a trade.</b> · [est] [JS production-eng loop]</summary>

- **Market data** flows *in* from exchanges, usually as high-rate binary feeds over UDP multicast (for example Nasdaq's ITCH), often with A/B redundant lines you arbitrate between. It is latency-critical and integrity-critical: if someone can corrupt or delay your view of the book, they can make you trade wrong.
- **Order entry** flows *out*: your system sends orders to the venue over a session protocol, FIX being the common one, or a binary protocol like Nasdaq's OUCH for speed. This path touches money, so it carries the heaviest pre-trade risk checks.
- **The matching engine** lives at the exchange; it pairs buy and sell orders by price-time priority and publishes the result back onto the market-data feed. Internally a firm may run its own matching engine for a dark pool or a simulator.
- **OMS/EMS** (order and execution management) sit between strategy and the wire, tracking positions and routing.
- **Co-location** means your servers sit in the exchange's data centre to cut the speed-of-light delay; "connect to an exchange" usually means a cross-connect in a co-lo plus a certified gateway session.

A **crossed book** (the Jump question) is when the best bid is at or above the best ask, which should not persist in one venue and signals stale data, a feed-arbitration bug, or a cross-venue latency gap.
</details>

<details>
<summary><b>FIX, and what a security engineer should know about it.</b> · [est] [JS: "FIX logs"]</summary>

FIX (Financial Information eXchange) is a tag=value session protocol over TCP: a logon (message type 35=A) establishes a session, then heartbeats keep it alive and **sequence numbers** detect gaps and replays. For security: classic FIX has weak native authentication (a CompID and often an IP allow-list), so it is almost always run over a private cross-connect or a VPN, not the open internet, and increasingly inside TLS. The things to watch: sequence-number manipulation (resend/gap-fill abuse), session hijack on an unencrypted link, and the fact that FIX logs are your primary forensic record of what was ordered and when, which is why their integrity and time-stamping matter. When they say "here are FIX logs", you are reading an order-flow timeline: logon, orders (35=D), cancels, execution reports (35=8), rejects.
</details>

<details>
<summary><b>Pre-trade risk controls and kill switches.</b> · [est] [the control that defines the domain]</summary>

Order entry must pass deterministic risk checks *before* it reaches the venue: per-symbol and per-account size and price limits (the fat-finger check), maximum order rate, maximum position and net exposure, duplicate-order detection, and a **kill switch** that can pull a strategy or the whole session instantly. In the US this is not optional: SEC Rule 15c3-5, the Market Access Rule, requires broker-dealers providing market access to enforce pre-trade risk controls and bans "unfiltered" or "naked" access where orders reach the exchange without them. These checks live on the critical path, so they are written to be fast and deterministic: exactly the "the model proposes, deterministic code decides" pattern from [block 16](../16-ai-and-llm-security/README.md), applied to a trading strategy instead of an LLM.
</details>

<details>
<summary><b>Time synchronisation, and why it is both a trading and a forensics control.</b> · [est]</summary>

Trading systems sync clocks tightly: PTP (IEEE 1588) gets sub-microsecond accuracy over the LAN, far beyond NTP. Two reasons a security engineer cares: regulation requires it (MiFID II mandates clock synchronisation, tightest for high-frequency activity), and forensics depends on it, because reconstructing an incident across market data, order entry and host logs is only possible if their timestamps agree. A drifting or spoofed clock breaks both trade sequencing and your incident timeline, so the time source is itself an asset to protect.
</details>

---

## The canonical story: Knight Capital

<details>
<summary><b>Why change management and kill switches matter: Knight Capital, 2012.</b> · [est] [the story every trading-security answer can reach for]</summary>

On 1 August 2012, Knight Capital deployed new order-routing code to its production servers but missed one of eight. On that server, the deploy reactivated dormant old code through a reused feature flag. At the open, that server began sending a flood of unintended orders into the market. It took roughly 45 minutes to stop, by which point the firm had lost about $440 million, and it did not survive as an independent company.

Use it as the proof that in trading, a deployment bug *is* a security-grade incident: the failure was change management (partial deploy, reused flag, no automated check that all nodes matched) and incident response (no fast, rehearsed kill switch). The lessons map straight onto controls you would propose: atomic, verified, all-or-nothing deploys; retire dead code rather than leaving it flag-gated; a tested kill switch with a clear owner; and pre-trade limits that would have capped the damage. It also connects to the AI risks in [block 16](../16-ai-and-llm-security/README.md): an autonomous agent with order-adjacent access is a Knight-Capital risk with a faster trigger finger.
</details>

---

## Answering the trading design questions

<details>
<summary><b>"How would you connect your new startup to the French exchange?"</b> · [GD Optiver]</summary>

Treat it as a secure-design question with a trading frame. Walk the path:

1. **Connectivity.** A private cross-connect or dedicated line into the venue (Euronext Paris here), or co-location, not the public internet. Redundant lines.
2. **Sessions and identity.** Certified gateway sessions (FIX or the venue's binary protocol) authenticated with venue-issued CompIDs and certificates; those credentials are crown jewels, stored in an HSM or vault, rotated, and scoped per environment.
3. **Pre-trade risk on the path.** The 15c3-5-style checks above, deterministic and fast, with a kill switch, before anything reaches the exchange.
4. **Segmentation.** The order-entry and market-data networks are isolated from the corporate network and from research; a compromise of a laptop must not reach the exchange gateway.
5. **Integrity and monitoring.** Validate and arbitrate market data (A/B feeds), monitor from taps so you do not add latency, and keep tamper-evident, time-synced logs of every order for forensics and regulators.
6. **Availability.** Failover that does not double-send orders; a tested procedure for a bad market-data feed or a runaway strategy.

Then the judgement line: you would not encrypt-and-inspect the hot path if it blows the latency budget; you would isolate it physically and inspect at the edges instead. That trade-off is the answer they are listening for.
</details>

<details>
<summary><b>Insider and alpha-protection risk.</b> · [est]</summary>

At a trading firm the highest-value theft is not customer data, it is the strategy code and research. The controls are classic but the asset is unusual: strict least-privilege and need-to-know on research repos, egress controls and DLP on research hosts (the LiteLLM supply-chain lesson in [block 08](../08-cloud-and-supply-chain/README.md) is partly an alpha-exfiltration story), monitoring for bulk access or unusual copies, and departures handled carefully. Say that you would protect the research environment as the crown jewel and design so a single compromised endpoint cannot reach either the alpha or the order path.
</details>

---

## Rapid-fire

| Question | One-line answer |
| --- | --- |
| What is FIX? | A tag=value session protocol over TCP for orders and executions; sequence numbers detect gaps and replays; weak native auth, so run it private or over TLS. |
| Market data vs order entry? | Data flows in (fast UDP multicast feeds like ITCH); orders flow out (FIX or a binary protocol like OUCH) and carry the risk checks. |
| What is a matching engine? | The venue component that pairs orders by price-time priority and republishes the result to the feed. |
| What is a crossed book? | Best bid at or above best ask: stale data or a feed/latency bug, should not persist. |
| SEC Rule 15c3-5? | The Market Access Rule: mandatory pre-trade risk checks, no naked/unfiltered access to the exchange. |
| What is a kill switch? | A tested control to pull a strategy or session instantly; Knight Capital is why. |
| Why co-location? | Servers in the exchange data centre to cut speed-of-light latency; "connecting to an exchange" is a cross-connect plus a certified session. |
| Why PTP not NTP? | Sub-microsecond clock sync for trade sequencing and cross-system forensics; regulation requires tight sync. |
| Why can't you inline-inspect the order path? | The inspection is the latency; segment and validate at the edges instead. |

[← AI and LLM security](../16-ai-and-llm-security/README.md) · [Index](../README.md) · [Next: OCaml literacy →](../18-ocaml-literacy/README.md)
