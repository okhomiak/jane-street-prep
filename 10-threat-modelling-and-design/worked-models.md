# Six worked threat models, at interview depth

[← Threat modelling hub](README.md) · [Frameworks](frameworks.md) · [Playbook](live-exercise-playbook.md) · [Hedge fund](hedge-fund-threat-model.md) · [Questions](interview-questions.md)

Each model below is written the way you would run it in the room, following the [playbook](live-exercise-playbook.md): the questions you ask first, the diagram in words, the threats as attacker paths, a ranking with one accepted risk, mitigations with a verification each, and a design change where one exists. Three are systems reported in real interviews, one is Jane Street's own published system, and two are trading-firm systems you can own. Practise them until you can draw each in two minutes, then stop reading them and run them cold.

---

## 1. Jane Street's permissions service, Ontology (from their 2019 tech talk)

This is the showpiece because it is theirs. Nathan Linger's talk "A Language-Oriented System Design" describes Ontology, the firm's permissions management service ([talk and transcript](https://www.janestreet.com/tech-talks/a-language-oriented-system-design/)). It is a 2019 description and the system has surely moved on, so in the room you model *the design as described* and say so; the point is the method, and that you did the reading. The Threat Modelling posting's named engineer works on exactly this class of system ([block 00](../00-jane-street-intel/README.md)).

**The design, from the talk.** State is a sequence of declarations in a small dependently typed language: nouns (users, groups, HG repos, user sets) and propositions (memberships, access grants, derived access such as "user has access via a group they belong to", subset types like "traders", recursive rules like nested mailing lists). A type checker keeps the whole sequence well-formed. Users never see the syntax: the UI renders it as English and uses structure editors that only offer valid choices. **Changes to data Ontology owns go through a request; Ontology runs a query to find the approvers, emails them, and applies the change once approved.** Data imported from other systems is updated by those systems on a schedule. Other systems receive permissions by subscription or periodic query; after a grant syncs, the user can, for example, clone the repo. Read permissions can be set on individual facts, so each user sees a personalised view. Deletes cascade and are soft for about two weeks. Queries are Prolog-like and potentially unbounded, so an execution budget caps them. Primitives like ports and IP ranges are built in, which suggests it also drives network configuration. Audit logging was not discussed.

**Questions I would ask first.** Who can define *approver* rules, and can a requester influence the data the approver query reads? How fast does a revocation propagate to every subscriber, and what happens if a subscriber is down? Which upstream systems feed imported data, and how is their write path protected? Are agent identities ([block 16](../16-ai-and-llm-security/README.md)) modelled as users? Is there an append-only record of every declaration with who made it?

**Diagram in words.** External entities: requesters, approvers, upstream source systems, downstream consumers (repo hosts, network devices, services). Process: the Ontology service (type checker, query engine, approval workflow, renderer). Store: the declaration sequence, plus the trash. Flows: requests in, approval emails out and approvals back, imported facts in on a schedule, subscriptions and queries out to consumers. Boundaries: user to Ontology; upstream systems to Ontology; Ontology to each consumer; and inside, the line between "data we own" and "data we import".

**Threats, as paths (STRIDE tag in brackets):**

1. **Approver manipulation (E).** If the facts that determine who approves a request are themselves editable through a path the requester controls, a requester can route their own request to a friendly approver, or to themselves. The crown-jewel path of any approval system.
2. **Upstream write path (T, E).** Imported data is applied without Ontology's approval flow. A compromised or misconfigured source system becomes a way to grant access that never passed an approver.
3. **Revocation lag (E, I).** Permissions reach consumers by subscription or periodic query. Between an approved removal and the last consumer syncing, a revoked user still has access; a consumer that caches and loses its subscription may keep stale grants indefinitely.
4. **Transitive over-grant (E).** Nested groups and recursive rules are powerful and hard to reason about. Adding one person to one group can silently derive access to many resources. The talk's own example, "traders" as a predicate-defined subset, is exactly the kind of rule that needs a continuous check.
5. **Approval fatigue and approval phishing (S, R).** Approvers act on emails. A high volume trains rubber-stamping; a forged "please approve" email, or a lookalike approval page, is a classic path to an illegitimate grant.
6. **The permission graph as reconnaissance (I).** Being able to browse who has access to what, which the UI offers, is valuable to an insider planning lateral movement; personalised read permissions mitigate this only if they are set tightly on sensitive facts.
7. **Execution budget as denial of service (D).** Unbounded queries are capped by a budget, which protects the service, but a crafted request that makes approver lookup exhaust its budget stalls approvals, which is denial of service against the *change* process. The speaker names confusing budget failures as a known rough edge.
8. **Trash restore (E).** Soft-deleted grants exist for two weeks. Whoever can restore from the trash can resurrect a revoked permission; that path needs the same approval as a grant.
9. **Repudiation (R).** Audit logging is not described. If any declaration can be made without a durable record of who, when and on what evidence, nobody can reconstruct how an access came to exist.
10. **Blast radius through primitives (T).** If IP ranges and ports are first-class, a wrong or malicious approval can change network reachability, not just repo access. The same workflow carries very different consequences.
11. **Agents as principals (E).** If AI counterparts act under identities in this system, every threat above applies to a requester that can be prompt-injected ([block 16](../16-ai-and-llm-security/README.md)).

**Ranking.** Top three: approver manipulation (it defeats the whole control), upstream write path (approval bypass by design), revocation lag (because it turns every offboarding and every incident response into a race). I would **accept** the reconnaissance risk at its current level for low-sensitivity facts, with read restrictions tightened only on the sets that gate crown jewels, because locking the whole graph destroys the self-service value the system exists for; the owner of that acceptance is the security team, reviewed when the next barrier-sensitive dataset is onboarded.

**Mitigations with verification.** Approver-defining declarations in a protected class that requesters cannot author, with a test that no request type can alter its own approver set; upstream sources treated as privileged writers (authenticated, integrity-checked, diffed, alerted on grant changes), with a detection on any grant that lacks a corresponding approval record; push-based revocation with acknowledgement, consumers that fail closed when a subscription is stale, and a measured revocation SLA; a nightly "effective access" diff that reports transitive expansions and any derived access to a barrier-sensitive resource; approvals in the UI rather than by reply, with signed notifications; trash restores routed through the grant workflow; an append-only log of every declaration with actor and evidence. **Design change to propose:** express information barriers as checks that run over *derived* access (a test that fails if any traders-set member can reach a restricted resource), because a positive deductive database makes "must not" hard to state directly, and a continuous test is the honest way to hold it.

**Did we do a good job?** The effective-access diff is quiet, revocation latency is measured, and no grant exists without an approval record. Re-run on any change to rule types.

---

## 2. A new venue connection (the trading-path model you own)

Reported in spirit by Optiver ("connect your new startup to the French exchange") and natural for Jane Street ([block 17](../17-trading-domain/README.md)).

**Questions first.** Which venue and protocol (FIX, or a binary order protocol)? Where does pre-trade risk run today, and will the new route pass through it? Who holds the venue credentials and certificates? Is the market data for this venue a new feed, and is it arbitrated? What is the latency budget?

**Diagram.** External: the venue (order entry and market data), the desk's strategy. Processes: feed handler, strategy, order gateway, pre-trade risk service, kill switch. Stores: credential vault, order and FIX logs. Flows: market data in over the cross-connect, orders out through gateway, risk decisions, logs. Boundaries: venue to firm (two directions), trading network to corporate network, strategy to gateway.

**Threats as paths.** A new route wired before the risk service covers it (bypass of pre-trade risk, E); a FIX session replayed or hijacked on an unencrypted link (S, T); a stale or crossed feed the strategy trusts (T, leading to bad orders); the venue credential leaking via logs or config (I, then S); the kill switch not covering the new session (D in reverse: you cannot stop it); the gateway reachable from a compromised workstation on the corporate network (E); order logs not time-synced with market data so the incident cannot be reconstructed (R); a deploy that lands on some gateway instances and not others (the Knight Capital shape, T).

**Ranking.** Risk-service bypass first (it is money with no brake), kill-switch coverage second (same reason from the other side), feed integrity third. **Accept** the residual latency cost of routing through the shared risk service, measured and within budget, owned by the desk lead, because the alternative of duplicated checks fails the single-enforcement-point test.

**Mitigations with verification.** A CI assertion that every order route calls the risk service; the kill switch tested against the new session in a dry run before go-live; A/B feed arbitration with a stale-data alert; credentials in the vault with a scanner proving no plaintext; session over a private link or TLS with sequence-number anomaly detection ([block 21](../21-detection-engineering/README.md)); PTP-synced tamper-evident logs; atomic all-or-nothing gateway deploys with a version check. **Design change:** make "passes the risk service" a property of the gateway framework, not a per-route configuration, so a future route cannot be wired wrong.

---

## 3. Object storage with presigned URLs (a reported question)

A candidate was asked to threat-model a storage system where users upload and retrieve objects via presigned URLs, objects are keyed by UUIDs stored in a datastore, and everything sits in one bucket; identify five threats and remediations and rank them ([Glassdoor](https://static.glassdoor.com.ar/Interview/security-engineer-interview-questions-SRCH_KO0,17_IP87.htm)).

**Questions first.** Who generates the presigned URLs and with what credentials? How long do they live? Can a user upload any content type and size? Is the UUID the only thing standing between users' objects? Are URLs ever logged, emailed or rendered in pages?

**Diagram.** External: users, the browser. Processes: the API that issues presigned URLs, the datastore of UUID to owner mappings. Store: one bucket. Flows: API issues URL; browser uploads or downloads directly to the bucket; API records metadata. Boundaries: user to API; browser to bucket (a direct path that bypasses your application for the data itself); API to datastore.

**Five threats, ranked.**
1. **URL leakage (I).** Presigned URLs are bearer tokens; they end up in proxy logs, browser history, Referer headers and support tickets. Remediation: short expiry, single-use where possible, no third-party scripts on pages that hold them, never log full URLs.
2. **Authorisation at issue time only (E).** The API decides once; the bucket enforces nothing about *who*. A bug in the mapping or a guessable or enumerable key gives another user's object. Remediation: unguessable keys, object-level authorisation checked at issue time with a tested policy, per-tenant prefixes with bucket policy conditions, and consider separate buckets per trust tier.
3. **Malicious upload (T, E).** Direct upload bypasses application validation: content-type spoofing, oversized files, stored XSS via HTML served from the bucket, malware. Remediation: constrain the presigned POST (content type, size, key), serve downloads with a safe content disposition and no scripting origin, scan after upload before marking available.
4. **Overwrite or deletion by replay (T, D).** A reused upload URL overwrites an object; a leaked one can be used to replace content. Remediation: write-once keys, versioning enabled, short expiry.
5. **Cost and availability (D).** Unlimited uploads or mass downloads run up cost or exhaust quotas. Remediation: quotas per user, rate limits on URL issuance, monitoring on egress.

**Accepted risk:** a short-lived download URL being shared deliberately by its legitimate owner, because preventing sharing of something the user is entitled to is not a security control, it is a product decision; owned by product. **Design change:** stop treating the URL as the authorisation and proxy downloads through the application for sensitive tenants, accepting the bandwidth cost there only. **Verification:** a test that one user's issued URL cannot address another user's key; a detection on download volume per URL.

---

## 4. A Signal-type messaging app (reported at Meta)

Reported as "thread model a signal type messaging application" ([Glassdoor](https://www.glassdoor.co.in/Interview/Meta-Security-Engineer-Interview-Questions-EI_IE40772.0,4_KO5,22.htm)). The trap is to talk about the cryptography and forget everything around it.

**Questions first.** Are messages end-to-end encrypted and does the server ever hold plaintext? How are identity keys verified? What metadata does the server see? Backups? Multi-device? Group management?

**Assets.** Message content, identity and session keys, contact graph and metadata, the client binary, the server's identity.

**Threats as paths.** Server compromise reading content (mitigated by E2EE only if keys never leave devices); a man-in-the-middle on first key exchange if users never verify safety numbers (S); metadata as the real disclosure (who talks to whom and when; I); device compromise reading plaintext at the endpoint, which no protocol fixes; group membership abuse (an attacker added to a group by a compromised member, or the server adding members if group changes are not authenticated; E); backup leakage if backups are not E2EE (I); malicious client update or supply-chain compromise of the app (T); account takeover via phone-number reassignment or SIM swap on registration (S); denial of service on the registration or push path (D); and the one candidates miss: repudiation design, where deniability is a *feature* and the model must say so (R, deliberately).

**Ranking.** Endpoint compromise first (it is where plaintext lives and the control is device security, not protocol), registration and identity verification second, metadata third. **Accept** that the server learns some metadata (timing, sizes), with sealed-sender style minimisation as the mitigation and a stated limit, because eliminating it entirely changes the product. **Design changes:** safety-number verification nudges, registration lock with a PIN, E2EE backups by default. **Verification:** reproducible builds and binary transparency for the client; tests that group changes require a member's signature.

---

## 5. A support bot that can issue refunds through a tool call (reported, and the one that converges with your work)

Reported as a hiring round scenario; the published best answer was a candidate who asked "who sets the refund limit, the model or the code?" ([KORE1](https://www.kore1.com/application-security-engineer-interview-questions/)). Full treatment in [block 16](../16-ai-and-llm-security/README.md); the threat-model shape here:

**Questions first.** Where does the refund limit live? What can the bot read (tickets, account data)? What can it send out (emails, links)? Is there a human step for anything?

**Diagram and boundaries.** Untrusted content (tickets) enters the model; the model calls a refund tool; the tool touches a payments system. The boundary that matters is between the model's proposal and the deterministic code that executes.

**Threats.** Indirect prompt injection in a ticket instructs a refund to an attacker account (E); model output flows to a page or email and carries a payload (T, the improper-output-handling class); the tool holds a payment token broader than the task (E, confused deputy); exfiltration through any outbound channel the bot has (I, the lethal trifecta); unbounded refunds under a loop (D on the balance sheet); no attribution of bot actions to a reviewable identity (R).

**Ranking and decision.** Deterministic authorisation first, scoped capability second, human approval above a threshold third. **Accept** that some benign refunds will be delayed by the approval step, owned by support operations. **Design change:** the limit and eligibility live in the tool's code; the model never holds a payment credential. **Verification:** a replay suite of injection attempts with attack success rate as a release gate.

---

## 6. An alternative-data research pipeline at a trading firm (the hedge-fund-specific one)

No candidate has reported this exact prompt; it is here because it is the system at a quant firm that generic threat modelling misses, and because it lets you show the business-logic instinct from [hedge-fund-threat-model.md](hedge-fund-threat-model.md).

**The system.** External data vendors deliver datasets (credit-card panels, satellite imagery, web-scraped signals) via SFTP or API; an ingestion service normalises them into the research data lake; researchers query the lake, build features and models in notebooks; promising signals move through review into production strategies.

**Questions first.** Which datasets are licensed with use restrictions, and which carry personal data? Who can read the lake, and is research separated from production? How do notebooks reach the internet? How does a signal get promoted, and who reviews it?

**Threats as paths.**
1. **Data poisoning upstream (T).** A vendor compromise or a manipulated feed teaches a model a false signal; academic work shows small perturbations to market data streams can steer algorithmic traders while evading anomaly detection ([arXiv](https://arxiv.org/pdf/2010.09246)). Impact lands later, in production, which makes it hard to attribute.
2. **Alpha exfiltration from the research environment (I).** The lake plus the notebooks are the firm's most valuable confidential data in its most usable form. Paths: a researcher's endpoint, notebook outbound access, a package pulled from a public registry that phones home (the 2026 LiteLLM compromise was a credential stealer in exactly this kind of environment, [block 12](../12-security-news/README.md)).
3. **Licence and personal-data misuse (I, and regulatory).** A dataset used beyond its licence, or personal data joined in ways the licence forbids. A business-logic threat with legal impact.
4. **Research-to-production promotion as a privilege boundary (E).** If a researcher can push a signal into a live strategy without an independent review, that is code deployment to the order path by another name.
5. **Notebook execution as remote code execution (E).** Notebooks run arbitrary code with the researcher's data access; a malicious dataset file or a dependency can run code in that context.
6. **Vendor credential and SFTP key sprawl (S, I).** Many vendors, many credentials, long-lived keys in scripts.
7. **Model extraction from internal AI assistants (I).** If researchers query an internal model trained on proprietary research, extraction and inversion attacks are real for high-value trading models ([arXiv survey](https://arxiv.org/pdf/2504.21574)).
8. **Availability of the lake during research deadlines (D).** Lower priority, but say it.

**Ranking.** Alpha exfiltration first (the crown jewel, with the current initial-access pattern aimed at developers and researchers), promotion-as-deployment second (it reaches the order path), poisoning third (hard to detect, delayed impact). **Accept** some vendor-feed integrity risk for low-weight signals with monitoring rather than full provenance, owned by the research lead, because the cost of cryptographic provenance across every vendor outweighs the harm from a low-weight signal.

**Mitigations with verification.** Research environment with egress allow-lists and a package mirror (detection on any direct outbound from notebook hosts); dataset provenance and statistical drift checks with alerts on distribution changes; promotion through a reviewed, versioned pipeline with the same gates as code to the trading path ([block 17](../17-trading-domain/README.md)); per-vendor short-lived credentials in the vault; dataset licence metadata enforced at query time; read access to the lake modelled as explicit grants in the permissions system (model 1) with information barriers as tests over derived access. **Design change:** separate "exploration" from "promotion" environments so that nothing in a notebook can reach production by any path but the reviewed one.

---

## How to practise these

Run each one against the clock in the [playbook](live-exercise-playbook.md) with someone reading the pushback lines. After three runs, they will feel scripted; that is when you change one assumption in each (the venue is a crypto exchange; the storage is for legal hold; Ontology is being extended to agents) and run them again, because the interviewer will do exactly that.

[← Threat modelling hub](README.md) · [Frameworks](frameworks.md) · [Playbook](live-exercise-playbook.md) · [Hedge fund](hedge-fund-threat-model.md) · [Questions](interview-questions.md)
