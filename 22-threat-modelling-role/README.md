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
| Practical threat modelling; turning principles into concrete architectural improvements | The core skill. A live threat-modelling exercise is near certain | Threat-modeling.io; the models you ran at the bank and at Wise |
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
4. **Whether both roles can be applied for.** The two postings share a department and almost certainly a hiring panel. Applying to both at once can read as not knowing what you want. The cleaner move is one primary application with a line in the free-text box saying you would also be glad to be considered for the other, which lets them route you.

**Recommendation.** Target the Threat Modelling role as primary when you apply, and prepare for the generic loop as well, because the technical rounds overlap heavily and the panel may test you across both. Everything in this repo still applies; this block adds what is specific.

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
