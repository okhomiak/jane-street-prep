# 10 · Threat modelling and security design

[← Incident response](../09-incident-response/README.md) · [Index](../README.md) · [Next: Judgement and hygiene →](../11-judgement-and-hygiene/README.md)

This is the block that decides the Threat Modelling role and carries heavy weight in the generic one. The London posting wants someone "familiar with threat modeling" who can explain trade-offs; the Threat Modelling posting wants an offensive background plus the ability to teach it ([Jane Street](https://www.janestreet.com/join-jane-street/position/7988179002), [block 22](../22-threat-modelling-role/README.md)). Outside Jane Street the round is mainstream: Meta's security phone screen is one threat-modelling round and one coding round, Amazon opens with "threat model this product", Microsoft asks you to model a new Azure service before launch, and Stripe calls it the design round ([interview-questions.md](interview-questions.md) has every reported prompt with its source). You founded a threat-modelling company and run design reviews on a trading platform, so this is where you should be the strongest person in the room. The module is built to make sure that shows. Tags: [JS] Jane Street, [M] Meta, [A] Amazon, [G] Google, [TW] tadwhitaker, [DF] Dataford, [GD] Glassdoor, [est] established.

## The module

| File | What it gives you | Read it when |
| --- | --- | --- |
| [frameworks.md](frameworks.md) | The four questions, STRIDE to the depth they probe, DFDs and trust boundaries, attack trees, ATT&CK-based modelling, PASTA and LINDDUN in a line each, why DREAD is dead, how to rank, model-as-code, and what to lead with | First pass, and before any mock |
| [live-exercise-playbook.md](live-exercise-playbook.md) | The 45-minute round on a clock: scoping questions, drawing, enumerating, ranking, deciding, closing; the published rubric; handling pushback; the attacker-first variant; a self-grading checklist | Before and after every mock |
| [worked-models.md](worked-models.md) | Six complete models: Jane Street's own permissions service Ontology (from their tech talk), a new venue connection, presigned-URL storage (a reported question), a Signal-type app (Meta), a refund-capable support bot, and a research data pipeline | To learn the depth, then to drill cold |
| [hedge-fund-threat-model.md](hedge-fund-threat-model.md) | What is different about a trading firm: the asset taxonomy, the actors' current playbook, the incidents to cite and the one not to, the business-logic abuse-case library, Jane Street specifics, a one-page reference model | Before any trading-flavoured prompt |
| [interview-questions.md](interview-questions.md) | Every reported threat-modelling prompt with source, the conceptual questions with answer shapes, the behavioural probes, and the wrong answers interviewers hear most | To self-test |

Related: how to run it as a programme at scale and where AI helps and misleads is [block 23](../23-ai-threat-modelling/README.md); the role itself, with the offensive-probe vocabulary and the documented-decision shape, is [block 22](../22-threat-modelling-role/README.md); the timed mocks are 8 and 9 in [block 19](../19-mock-interviews/README.md).

---

## The method in one breath

Ask before you draw. Name the four questions as your structure. Draw the data flows and the trust boundaries before any threat. Mark the crown jewels. Say threats as attacker paths across each boundary, then sweep with STRIDE for what you missed. Rank with a stated method, pick three, name one you accept and who owns that. Give each mitigation a verification. Propose one design change that removes a threat rather than defending it. Finish on time with three owned actions and a question back to the owner. The full clock is in the [playbook](live-exercise-playbook.md).

What they score at Jane Street on top of that: collaboration (you ask, you adjust), humility (you concede when the owner is right), measured judgement (you say what you would skip), and whether you can switch altitude from whiteboard to mechanism in one breath.

---

## Design prompts with worked answers

The threat-modelling exercises are in [worked-models.md](worked-models.md). The prompts below are the broader secure-design and programme-design questions that share the round.

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

<details>
<summary><b>Threat model an LLM agent system.</b> · [current; given the firm's agent fleet, likely]</summary>

Use the same four questions, but enumerate the pieces that are specific to agents: the assets (the model, the system prompt, tool credentials, the memory/context store, the data the tools reach), the identities (the agent's own identity, the user it acts for, the tools' credentials), and crucially **every path where untrusted content reaches the model** (user input, retrieved documents, tool descriptions, tool outputs) and **where the model's output then flows** (a shell, SQL, HTML, another agent). Then the controls per path: least-privilege tools, deterministic authorisation the model cannot override, human approval for irreversible actions, output validation, and a sandbox. The full version, with Jane Street's own design, is [block 16](../16-ai-and-llm-security/README.md); here, the point to show is that an agent is just a system with an untrusted component wired to real tools, so the method does not change, only the data-flow map does.
</details>

<details>
<summary><b>Design a firm-wide permissioning system.</b> · [JS: their cybersecurity SWE owns exactly this; the system is public] [likely design prompt]</summary>

Minjie's profile says the firm-wide internal permissioning system is one of his main projects, used by everyone in the firm, and that he onboards other teams onto it ([block 00](../00-jane-street-intel/README.md)). Jane Street also described the system publicly: Ontology, a deductive database of typed declarations with an approval workflow and outward sync, in Nathan Linger's 2019 tech talk; a full threat model of that design is [worked model 1](worked-models.md). So this is a live system they can ask you to design or critique, and you can speak to its actual shape. Walk it:

1. **What it decides.** Who (a person, a service, or at Jane Street an AI counterpart identity) can do what (read, write, approve, run) to which resource (a dataset, a repo, a desk's positions, a production host), under what conditions (time, location, device health, an approval). Say that identities now include agents, so the model must carry attribution for AI-originated actions ([block 16](../16-ai-and-llm-security/README.md)).
2. **Pick the model deliberately.** RBAC (roles bundle permissions; simple, but role explosion at scale), ABAC (policies over attributes of subject, resource and context; flexible, harder to reason about), and ReBAC (permissions derived from relationships, the Google Zanzibar model, tuples like `desk:fx#trader@user:oleh`, with a `check` API and `expand` for "who can see this"). A trading firm's needs, team and desk membership that implies data access, map naturally onto ReBAC with ABAC conditions for device and time. Say which and why.
3. **Make the dangerous things hard.** Deny by default; least privilege with **just-in-time elevation** and approvals for admin actions, with automatic expiry; separation of duties for anything that moves money or changes production; break-glass that is loud and audited.
4. **Information barriers.** A trading firm has a specific requirement that most companies do not: certain desks or roles must not see each other's positions or research. Model that as an explicit *negative* relationship or a policy check that cannot be overridden by a positive grant, and test it continuously, because it is the control a regulator and an insider both care about. Keep it technical: enforcement at the data layer, not a policy document.
5. **Operate it.** A single policy decision point everything calls (so there is one place to audit), fast cached decisions on the hot path, a complete decision log (who asked, what was decided, which rule), access reviews driven by data (dormant permissions, outliers versus peers), and self-service requests with approvals so the security team is not the bottleneck. That last point is the onboarding work Minjie describes.
6. **What you would skip.** Do not try to model every permission on day one; start with the crown-jewel resources and the admin paths, leave low-risk resources coarse, and widen as the model proves itself.
</details>

<details>
<summary><b>Design a vulnerability management programme.</b> · [GD Optiver, Jan 2026: asked directly] [JS Vulnerability Management posting]</summary>

An Optiver security loop asked this and expected thinking under pressure ([Glassdoor](https://www.glassdoor.co.in/Interview/Optiver-Interview-E243355-RVW102659786.htm)); Jane Street has a whole posting for it that reads like the answer key: validate and dedupe scanner findings, prioritise by real exploitability not severity score, scanning coverage and data quality, SBOM and software inventory, dashboards and SLAs, and automation because "manual triage doesn't scale at our size" ([posting](https://www.janestreet.com/join-jane-street/position/8596433002)). Build the answer on that spine:

1. **Inventory first.** You cannot prioritise what you have not found: assets (hosts, containers, cloud resources, appliances), software (an SBOM per build, so "are we affected" takes minutes not days, the Log4j test), and owners. Coverage is the first metric.
2. **Find.** Scanners (Tenable, Qualys, Rapid7 are the ones they name), agent-based where possible, plus SCA in CI for dependencies and cloud posture checks. Then **validate and dedupe**: the same CVE across three tools is one finding; a scanner's "critical" on an unreachable port is not.
3. **Prioritise by risk, not CVSS.** CVSS measures severity, not likelihood. Combine: is it in CISA's Known Exploited Vulnerabilities list; EPSS (probability of exploitation in the next 30 days); is the asset exposed and reachable; is the vulnerable code path actually used; asset criticality (the order path versus a lab box). A medium CVSS on the exchange gateway outranks a critical on an isolated test host. Say that sentence.
4. **Fix with SLAs by tier**, and be realistic about a trading firm: patches to the trading path land in change windows outside market hours, so compensating controls (segmentation, virtual patching, monitoring) cover the gap and you track the exception, not pretend it is not there.
5. **Automate the triage.** Ticket creation and routing to owners, auto-closure on rescan, enrichment with KEV/EPSS/exposure, and, where safe, automated remediation for well-understood classes (this is your remediation bot: say what it fixed, what it never touched, and the human gate for anything risky). The Jane Street posting explicitly allows AI tooling here with a human in the loop.
6. **Measure.** Coverage (assets and software inventoried and scanned), time-to-remediate by tier, SLA adherence, exceptions open and ageing, and the one that matters to leadership: exposure of crown-jewel assets to known-exploited vulnerabilities, trending down.
</details>

<details>
<summary><b>Design the security automation for our SOC. What do you automate first, and what never?</b> · [JS: "security automation required"; "self-healing systems" in the ops posting]</summary>

Automation is the stated requirement of this role, and the trap is automating response before you have automated understanding. Order of operations:

1. **Enrich, always.** Every alert arrives with context attached: asset owner and criticality, user's role and normal locations, hash and domain reputation, recent related alerts, the EDR process tree. This is pure upside and it is where most analyst minutes go.
2. **Triage and score, mostly.** Rules and models that close the known-benign (the admin running the scheduled task they always run), raise the suspicious, and route the rest. Measure the auto-close false-negative rate on a sampled review.
3. **Contain, with guardrails.** Isolate a host, disable a token, block a domain, only when the signal is high confidence *and* the action is reversible *and* the target is not on the never-auto list (domain controllers, exchange gateways, executives' devices, anything on the trading path during market hours). Rate-limit and circuit-break so a bad rule cannot isolate the fleet.
4. **Remediate, last.** Reversible first (revoke and reissue a credential, roll back a config), and anything destructive or production-touching (reimage, account deletion, a change to the order path) gets a human approval step. This is the same principle as the AI-agent design in [block 16](../16-ai-and-llm-security/README.md): the automation proposes, a deterministic policy decides, a person approves the irreversible.

Engineering rules that make it trustworthy: every action is idempotent and logged under the automation's own identity; a dry-run mode that shows what it would have done before you turn it on; playbooks in version control with tests and a staging environment; and metrics (actions taken, actions reversed, analyst time saved, false-action rate). Your remediation bot is the story here: lead with what it was allowed to change, what it escalated, and the measured result (fewer post-deployment incidents, faster approvals), not with the tooling.
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
