# 22 · The Threat Modelling role: analysis, fit, and how its loop differs

[← Detection engineering](../21-detection-engineering/README.md) · [Index](../README.md) · [Next: AI-assisted threat modelling →](../23-ai-threat-modelling/README.md)

Jane Street runs a second security opening alongside the generic Cybersecurity Engineer: **Cybersecurity Engineer - Threat Modelling**, posted in London, New York, Hong Kong and Singapore ([London](https://www.janestreet.com/join-jane-street/position/7988179002), [NYC](https://www.janestreet.com/join-jane-street/position/7999330002), [HK](https://job-boards.greenhouse.io/janestreet/jobs/7998232002)). The NYC version lists a base of $225,000 to $300,000 plus discretionary bonus. You founded a threat-modelling company and spend your days doing product security on a trading platform, so this block asks the uncomfortable question first: is this the role you should actually be targeting, and what changes if so. Tags: [JS] their posting, [GD]/[Blind] reports, [est] established.

---

## What the posting says, and what each line is really asking for

Their requirements, with the read-through:

| They wrote | What it means for the loop | Your evidence |
| --- | --- | --- |
| Hands-on engineer who finds *and fixes* vulnerabilities, strengthening applications, data, infrastructure and processes | Not an analyst role despite the Blind thread calling it one; expect to be asked how you fixed things, not just found them | Remediation bot; the 40% drop in post-deployment incidents |
| Guides teams to assess the security implications of design decisions and build security in from the start | Design reviews and architecture influence; they will test whether you can run one live | Product security on a greenfield cross-asset trading platform |
| Draws on an **offensive security background** to identify threats and dig into technical details | Required, not optional. Expect "how would you attack this" before "how would you defend it" | Synopsys offensive security consulting; the Cambridge reverse-engineering course |
| Practical threat modelling; turning principles into concrete architectural improvements | The core skill. A live threat-modelling exercise is near certain | Threat-modeling.io (Python/FastAPI, STRIDE, ATT&CK and OWASP outputs); LLM threat modelling under Glasswing at J.P. Morgan; the models you ran at the bank and at Wise |
| Talking with system owners who know their applications but may not recognise exploits, then helping resolve them | The teaching-and-influence half. They will probe how you bring people along, not just how you find flaws | 450+ engineers trained; growing a team 5 to 13 |
| Balancing security controls against business requirements | Measured judgement, the same culture test as the generic role ([block 11](../11-judgement-and-hygiene/README.md)) | The approval-time improvement (35% faster): security that sped the business up |
| Comfortable whiteboarding *and* hands-on troubleshooting | Both altitudes in one interview; the risk for you is staying too high | Pair every architecture point with a concrete mechanism |
| Skilled at evaluating trade-offs and making risk-based decisions | Expect "what would you accept, and why" | Your exception and risk-acceptance stories, told as engineering decisions |
| Excellent communicator, builds relationships; collaborative mindset "a must" | Behavioural weight is high; be nice and be clear are scored | Teaching record, cross-team work |
| **Able to teach others and transfer threat-modelling knowledge** | Explicit. Expect "how would you teach a team to threat-model" and possibly to do it in the room | Threat-modeling.io is literally this |
| Strong Linux background, comfortable in the terminal, confident scripting | Same bar as the generic role; [block 06](../06-linux/README.md) and [03](../03-coding/README.md) still apply | Your scanning pipeline and bots |
| Comfortable taking accountability; **document your decision-making rationale** | They want written reasoning, not verdicts. A design review that records why | Any ADR-style write-up you have done |
| Proficiency in any programming language is **a plus** | The one real difference from the generic engineer role: coding is not the gate here | Still code in the loop; it is your edge, not a hurdle |

Two things the posting does **not** mention: a specific framework (no STRIDE, no PASTA; they say "practical threat modelling skills", which means they care about the thinking, not the acronym), and AI. Nothing about scale either, though the team-wide context from the departments page applies: a small group covering a firm that builds nearly everything in-house ([block 00](../00-jane-street-intel/README.md)).

---

## Fit, honestly

**Why this is the stronger fit.** Every required line maps to something you have done and can prove with a number. The generic Cybersecurity Engineer role is Windows-heavy with automation in a real language as the core ([block 00](../00-jane-street-intel/README.md)); your Windows depth is your known gap. The Threat Modelling role's core is design-level security judgement, offensive instinct, and teaching, which is the centre of your career, not its edge. You would walk into the live threat-modelling exercise as the strongest person in the room.

**The risks, in order.**

1. **The CV screen, and seniority signalling.** A Blind commenter reported not getting past the CV screen for exactly this role ([Blind](https://www.teamblind.com/post/jane-street-security-engineer-krwvr2zb)), and job boards tag the posting as mid-level. An SVP title, a bank's vocabulary, and a founder line can read as "will not be hands-on" to a screener looking for a practitioner. This is risk 1 from [block 20](../20-gap-analysis/README.md), amplified. The fix is in how you write the application: lead with what you built and found, state plainly that you want a hands-on role, and keep governance language out entirely.
2. **The "offensive background" probe.** Synopsys was years ago. They will test whether the offensive instinct is current: can you still look at a system and say how you would break it, with specifics. The AI DAST verifier is recent offensive work and your best current evidence; the hardware and web attack material in [blocks 04](../04-security-fundamentals/README.md), [07](../07-windows-and-endpoints/README.md) and [15](../15-malware-forensics-reversing/README.md) keeps the vocabulary sharp.
3. **Over-indexing on your own product.** Threat-modeling.io is a strength and a hazard: mentioned once as proof you understand scaling the practice, it lands; mentioned three times, it is a pitch. The posting wants someone who can teach a *human* practice. Lead with the practice, not the tool.
4. **Applying to both roles.** Their own FAQ settles this: an application is considered for all open roles globally, candidates are often considered for more than one team during a process, and they say to apply to any and all roles you are interested in ([Interviewing FAQ](https://www.janestreet.com/join-jane-street/interviewing/)). So apply to both. What matters is that the *pitch* is coherent: one story about who you are (design-level security, offensive instinct, teaching, and you build), with the Threat Modelling role as the natural home and the generic role as the one you would also do well in.
5. **You have applied before.** The form asks "Have you interviewed with Jane Street before?" and the answer is yes; they will know regardless. Their FAQ is explicit that you can re-interview, that they have hired people on a second or third attempt, and that experienced hires should wait until their experience or circumstances have changed significantly (not a hard rule). The six-month roadmap ([block 24](../24-six-month-roadmap/README.md)) is built to make that true in a way you can point at: new public work, new depth, a sharper target. Say what changed, briefly and concretely, in the free-text box ([block 25](../25-the-application/README.md)).

**Recommendation.** Make the Threat Modelling role the centre of your pitch and apply to both, and prepare for the generic loop as well, because the technical rounds overlap heavily and the panel may test you across both. Everything in this repo still applies; this block adds what is specific.

---

## How this loop is likely to differ

No candidate report exists for this exact title (the Blind commenter did not reach interviews), so this is inference from the posting and from the generic security loops in [block 01](../01-process/README.md). Expect the same shape (screen, technical rounds, final block with hiring manager) with the weight shifted:

- **A live threat-modelling exercise is near certain.** A system described on a whiteboard or in a doc; you model it in the room while they watch how you think and how you bring them along. Possibly their own system, possibly one of yours. Method in [block 10](../10-threat-modelling-and-design/README.md); your differentiator is doing it *with* them, Shostack's four questions out loud, leaving three concrete actions.
- **"How would you attack this" before "how would you defend it".** The offensive requirement means attack paths first. Say the attacker's first three moves, then the controls.
- **A teaching probe.** "A team has never threat-modelled. How do you start?" or "Explain threat modelling to a developer who thinks it is paperwork." The answer is the lightweight, system-owner-led session, not a programme ([block 10](../10-threat-modelling-and-design/README.md), the teach-a-new-team prompt).
- **A written-reasoning probe.** "Document your decision-making rationale" suggests they may ask you to write, or to walk through a decision you wrote up: the options, the trade-off, what you accepted and why. Have one real example ready in that shape.
- **Fewer pure coding rounds, not none.** Coding is "a plus", so the shared-doc coding round may be lighter or absent, but Linux and scripting are required, so expect the terminal and a scripting task at minimum.
- **The AI angle is yours to bring.** The posting does not mention AI, but a threat-modelling role at a firm running an agent fleet is where AI-assisted threat modelling is most relevant. [Block 23](../23-ai-threat-modelling/README.md) is built for that conversation; use it when they ask what you would change or build.

---

## "How would you attack this?" The vocabulary for the offensive probe

The posting requires an offensive background, so expect to be handed an architecture and asked for the attacker's view first. You are naming attack *paths* and the control that breaks each, not operating tools; depth lives in the linked blocks. Have the first three moves ready per surface:

| Surface | An attacker's first moves, by name | What breaks the path | Depth |
| --- | --- | --- | --- |
| Web application or API | Look for injection in anything that reaches a database or shell; abuse the authorisation model (IDOR, missing object-level checks); hunt SSRF to reach internal services or cloud metadata | Parameterised queries, object-level authz checks, egress allow-lists and IMDSv2 | [04](../04-security-fundamentals/web-and-appsec.md) |
| Identity and sessions | Phish or steal a session token rather than a password; replay a bearer token; abuse an OAuth consent or a weak reset flow | Phishing-resistant MFA, token binding and short lifetimes, conditional access | [04](../04-security-fundamentals/auth-and-identity.md), [07](../07-windows-and-endpoints/README.md) |
| Active Directory estate | Spray passwords under the lockout threshold; request service tickets to crack offline (Kerberoasting); abuse a misconfigured certificate template (ESC1); move laterally with stolen hashes | Tiered admin, gMSAs and AES-only, template audits, Credential Guard and LAPS | [07](../07-windows-and-endpoints/README.md) |
| Cloud account | Find an over-privileged role or a leaked key; pivot through assumable roles; reach storage that trusts the network rather than identity | Least-privilege roles with boundaries, short-lived credentials, deny-by-default storage policies | [08](../08-cloud-and-supply-chain/README.md) |
| CI/CD and supply chain | Compromise a dependency or a third-party action; steal CI credentials from a build log; push a trusted-looking release | Pinned and signed dependencies, isolated short-lived CI identities, provenance and SBOMs | [08](../08-cloud-and-supply-chain/README.md) |
| Agent and LLM systems | Plant an instruction in content the agent will read (indirect injection); exfiltrate through an outbound channel the agent holds; swap a tool definition after it is trusted | Cut a leg of the lethal trifecta, deterministic authorisation, pinned tool definitions, a sandbox | [16](../16-ai-and-llm-security/README.md) |
| Trading path | Reach the order gateway from a compromised workstation; corrupt or delay market data; find a route that bypasses pre-trade risk | Physical segmentation, feed arbitration and integrity alerts, every route through the risk service, a kill switch | [17](../17-trading-domain/README.md) |
| People and process | Business email compromise against payments; a fake job interview or tampered installer against a developer (the 2026 Binance chain) | Out-of-band verification of payment changes, DMARC at reject, developer-endpoint hardening | [21](../21-detection-engineering/README.md), [12](../12-security-news/README.md) |

Always finish the attacker's view with the defender's: the probe is testing whether you see both, and whether you rank the paths by what reaches the crown jewels.

---

## Documenting a decision: the shape they ask for

"Comfortable taking accountability and able to document your decision-making rationale" is a requirement, so have one real, sanitised example in this shape and be ready to write one live. Keep it to a page.

```
Title:        Where the pre-trade risk check sits for the new venue route
Context:      New strategy connecting to venue X via a second gateway. Latency budget on the
              order path is tight. Existing risk service sits on the primary route only.
Options:      A) Duplicate the risk checks inside the new gateway (fast, two copies to keep in sync)
              B) Route the new gateway through the existing risk service (one copy, adds ~N µs)
              C) Ship without the check and rely on venue-side limits (fastest, no firm control)
Decision:     B, with the risk service's hot path profiled and the venue limits kept as a backstop.
Rationale:    One enforcement point is auditable and testable; the latency cost is measured and
              within budget; C fails the Knight Capital test (no firm-side kill switch on that route).
Risks:        Risk service becomes a single point of failure for both routes (mitigated: active-standby,
              fail-closed with desk alert). Accepted residual: ~N µs on the new route.
Verification: CI assertion that every order route passes the risk service; a detection for orders
              seen at the venue with no matching risk-service decision.
Owner / date: You. Review in 90 days or on the next gateway change.
```

The features a Jane Street reader scores: the options you rejected are written down with why, the residual risk is named and accepted by someone, and there is a verification, so the decision is checkable later rather than remembered.

---

## Questions to ask them, for this role

- How does a threat model here turn into engineering work: who owns the follow-through, and how do you know a mitigation actually shipped?
- Which systems have been modelled, and how did you choose them? What is the one you most want modelled next?
- How do system owners experience a review today: a conversation, a document, a gate?
- Where has the offensive view changed a design decision recently?
- How does the team think about the agent fleet from a threat-modelling perspective, and who models the models?

---

## Questions to prepare specifically for this role

| Question | Where the answer lives |
| --- | --- |
| Threat-model this system, live | [10](../10-threat-modelling-and-design/README.md) |
| How would you attack this architecture? Then how would you fix it? | [10](../10-threat-modelling-and-design/README.md), [04](../04-security-fundamentals/web-and-appsec.md), [08](../08-cloud-and-supply-chain/README.md) |
| A team has never threat-modelled; how do you start, and how do you make it stick? | [10](../10-threat-modelling-and-design/README.md) teach-a-new-team |
| A system owner says the risk you found is not real. What do you do? | [11](../11-judgement-and-hygiene/README.md), [13](../13-behavioural/README.md) stakeholder story |
| Walk me through a security decision you documented: options, trade-off, what you accepted | your ADR-style example; shape in [13](../13-behavioural/README.md) |
| How do you prioritise which systems to threat-model first across a firm? | crown jewels and attack paths, [10](../10-threat-modelling-and-design/README.md), [17](../17-trading-domain/README.md) |
| How would you scale threat modelling across hundreds of internal systems with a small team? | [23](../23-ai-threat-modelling/README.md) |
| What is wrong with how most companies do threat modelling? | [23](../23-ai-threat-modelling/README.md), problems section |
| Design a permissioning system / a vulnerability-management programme (architectural improvement prompts) | [10](../10-threat-modelling-and-design/README.md) |
| Why this role rather than the generic security engineer role? | this block, said plainly: design-level security and teaching are your centre |

[← Detection engineering](../21-detection-engineering/README.md) · [Index](../README.md) · [Next: AI-assisted threat modelling →](../23-ai-threat-modelling/README.md)
