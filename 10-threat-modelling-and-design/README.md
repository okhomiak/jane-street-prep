# 10 · Threat modelling and security design

[← Incident response](../09-incident-response/README.md) · [Index](../README.md) · [Next: Judgement and hygiene →](../11-judgement-and-hygiene/README.md)

The London posting wants someone "familiar with threat modeling" who can explain trade-offs, and Jane Street's Threat Modelling Engineer posting wants an offensive background plus the ability to teach it ([Jane Street](https://www.janestreet.com/join-jane-street/position/7988179002)). Meta's security screen is one coding round and one threat-modelling round; the reported prompt was "threat model a Signal-type messaging application" ([Glassdoor](https://www.glassdoor.com.au/Interview/Meta-Interview-E40772-RVW70419539.htm)). Amazon's loop is "go over a specific product and threat model it" plus STRIDE ([Glassdoor](https://www.glassdoor.com.br/Entrevista/Amazon-Entrevista-E6036-RVW33223721.htm)). Tags: [JS] Jane Street, [M] Meta, [A] Amazon, [G] Google, [TW] tadwhitaker, [DF] Dataford, [GD] Glassdoor.

---

## The 10-minute method

Draw first, talk second, and end with what you'd deliberately not do.

1. **Ask what it's for and who uses it** (30 s). Users, admins, integrations, data classification.
2. **Draw it** (2 min). Boxes for components, arrows for data flows, dotted lines for trust boundaries (internet to edge, edge to app, app to data, user to admin, your code to third parties).
3. **Name the crown jewels** (30 s). At a trading firm: strategy code and parameters, the ability to send orders or move money, signing keys and credentials, market data entitlements, client data, the build system.
4. **Walk the attackers across the boundaries** (4 min). Pick the three most likely: a phished or infostealer-infected employee laptop, a malicious or hijacked dependency, an insider; add a compromised vendor or a bug in the public surface if relevant. Use STRIDE as a silent checklist per flow: spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege.
5. **Controls** (2 min). Prefer ones that make the secure path the easy path. Prevent where cheap, detect where prevention costs too much.
6. **Trade-offs** (1 min). What you'd skip, what residual risk you'd accept, and how you'd know you were wrong (the detection that tells you).

Say the frameworks once, lightly: STRIDE for threats, attack trees for depth, ATT&CK for realism, DREAD only to say why you don't use it (subjective scores). Jane Street's own words for the trait: measured judgement.

---

## Prompts that have been asked, with a worked answer each

<details>
<summary><b>Threat model a Signal-type messaging app.</b> · [M]</summary>

Draw: clients, a registration and key server, message relays, push notification services, backups, contact discovery. Crown jewels: message content, who-talks-to-whom metadata, the private keys on devices. Threats by boundary: a malicious server (so the protocol must be end-to-end with keys the server never sees; verify safety numbers to beat a MITM at key exchange), a compromised device (screen lock, disappearing messages, sealed sender limits metadata), the push provider (payload must be empty or encrypted), contact discovery (hash enumeration, so use private set intersection or enclave-based lookup), backups (end-to-end encrypted with a user key or none), account takeover through SMS registration (registration lock PIN), group membership leaks, and traffic analysis. Controls: a ratcheting protocol for forward and post-compromise secrecy, sealed sender, reproducible builds, a short data-retention story you can show a court. Trade-off: usability of key verification versus MITM risk; most users never verify, so make the default safe and the warning loud on key change.
</details>

<details>
<summary><b>Threat model a sample application / a specific product (e.g. a grocery delivery tracker).</b> · [M] [A] [DF]</summary>

Pick a stack and draw it: mobile app, API gateway, services, database, a third-party maps API, a payments provider, an admin console. Crown jewels: payment data (keep it with the provider), customer addresses, driver locations, the admin console. Threats: IDOR between customers' orders, live-location leaks through overly broad APIs, account takeover via SMS resets, SSRF from the webhook receiver, an over-privileged admin console reachable from the internet, a mobile app with secrets baked in. Controls: object-level authorisation tests in CI, coarse location rounding for anyone but the assigned driver, passkeys for customers, allowlisted webhook sources with signatures, admin behind SSO and device trust, no secrets in the app. Trade-off: you'd accept a less precise live map to protect driver safety.
</details>

<details>
<summary><b>Given an architecture, name three attack vectors and mitigations.</b> · [DF, Jane Street guide]</summary>

Answer in the shape they asked for. Vector 1: the public edge (an auth bypass or injection in the API), mitigate with a hardened framework, tests, a WAF for time, and least-privilege database accounts. Vector 2: the people (phishing an engineer with deploy rights), mitigate with passkeys, device trust, and deploy credentials that live in CI not laptops. Vector 3: the supply chain (a poisoned dependency), mitigate with pinning, a mirror with cooldown, and egress restrictions. Then the one they didn't ask for: the insider, mitigated by need-to-know and audit.
</details>

<details>
<summary><b>Design a monitoring system to detect anomalies in network traffic.</b> · [DF, Jane Street guide]</summary>

Inputs: flow records from switches and cloud (NetFlow, VPC flow logs), DNS query logs, proxy logs, EDR network events. Baseline per host and per role: destinations, bytes, ports, time of day, learned over weeks. Detections in tiers: deterministic rules first (new external destination for a server, DNS to a domain registered this week, long DNS labels, periodic beacons with low jitter, upload volume above N standard deviations, connections to known-bad lists), then statistical outliers, then ML only for ranking, never as the sole alert. Output: an alert with the host, process, destination, and the three facts an analyst needs; a feedback loop so every closed alert tunes the rule. Say the hard parts: encrypted traffic (use metadata, JA3/JA4 fingerprints, SNI), NAT hiding the source, and alert fatigue, which is why precision matters more than recall for the first version.
</details>

<details>
<summary><b>Design a SIEM.</b> · [GD senior infosec]</summary>

Collection (agents and forwarders, syslog, APIs from cloud and SaaS), normalisation into a schema (ECS or OCSF), hot storage for weeks and cold for a year, a detection engine running rules on a stream (Sigma as the portable format), correlation with asset and identity context, case management, and dashboards last. Non-functional: volume (count events per second and cost), retention by law, integrity (write-once storage so an attacker can't delete logs, which they try to), access control on the SIEM itself. Build versus buy: buy the platform, build the detections.
</details>

<details>
<summary><b>"How would you build the ultimate botnet?" / "How does a botnet work?"</b> · [DM] [G: reported twice in Google security phone screens]</summary>

This is a defender's question in disguise; Miessler's note is that "I stop them, I don't build them" is a poor answer because it dodges showing you understand the adversary. Answer at the architecture-and-defence level, which is what the interviewer scores, and don't drift into an operational how-to.

Cover the moving parts conceptually and pair each with how you'd detect or disrupt it:

- **Command and control model.** Centralised C2 is simple but has a single point you can sinkhole; peer-to-peer and domain-generation approaches exist precisely to survive takedown. Defender view: find the beacon (regular interval, low jitter, small payloads), sinkhole or block the domains, and go after the infrastructure with registrars and hosts.
- **Resilience tricks.** Fast-flux DNS, domain generation, encrypted channels and hiding C2 inside normal services (HTTPS, cloud APIs) all aim at evading blocklists. Defender view: DNS analytics, newly-registered-domain blocking, TLS metadata and fingerprinting, and threat-intel sharing.
- **Growth of the fleet.** Spread is usually phishing, exploited edge devices and weak credentials. Defender view: patch the edge, MFA, and segment so one infected host can't recruit the next.
- **Takedown.** The real lever is the infrastructure and the people: registrar and hosting cooperation, law enforcement, and the coordinated disruptions vendors run (the kind of action Microsoft and the FBI took against phishing and proxy services through 2026).

Close with the trading-firm angle: the thing you most want to catch is one of your own machines beaconing out, so the detection in [block 02](../02-log-investigation/README.md) and the monitoring design above are the real deliverable.
</details>

<details>
<summary><b>Design a secure software supply chain / secret storage / a CI-CD pipeline.</b> · [TW]</summary>

These are design prompts as much as cloud ones; the worked answers live in [block 08](../08-cloud-and-supply-chain/README.md). The shape in an interview is the same: draw the flow from commit to running code, mark the trust boundaries (developer, CI, registry, runtime), and put one control on each boundary, then say which control you'd add first and why.
</details>

<details>
<summary><b>How would you teach threat modelling to a team that's never done it?</b> · [JS Threat Modelling Engineer]</summary>

Run it on their own system, not a slide deck. 45 minutes, a whiteboard, four questions: what are we building, what can go wrong, what are we going to do about it, did we do a good job (that's the Shostack framing). Leave them with three concrete actions and make the next session lighter so it becomes a habit rather than a gate. Mention Threat-modeling.io only if it fits, as the thing you built because the manual version didn't scale, not as a pitch.
</details>

---

## Design principles to name

Least privilege, defence in depth, fail safe (deny on error), complete mediation (check every access, not just the first), separation of duties, secure defaults, minimise attack surface, economy of mechanism (simple is auditable), open design (no security through obscurity), psychological acceptability (if it's painful, people route around it, which is Jane Street's "paved roads, not gates"). Say the last one is the one most programmes get wrong.

## What good looks like in this round

- You drew something in the first two minutes.
- You named the crown jewels before listing generic threats.
- You tied each threat to a boundary on your diagram.
- You picked controls that fit the business, and you said what you'd skip.
- You ended with how you'd know the design was failing in production.

[← Incident response](../09-incident-response/README.md) · [Index](../README.md) · [Next: Judgement and hygiene →](../11-judgement-and-hygiene/README.md)
