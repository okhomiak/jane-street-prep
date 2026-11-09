# The live threat-modelling exercise: a 45-minute playbook

[← Threat modelling hub](README.md) · [Frameworks](frameworks.md) · [Worked models](worked-models.md) · [Hedge fund](hedge-fund-threat-model.md) · [Questions](interview-questions.md)

This round runs the same way almost everywhere: the interviewer describes a system in a paragraph or draws it, then goes quiet and watches how you think for 30 to 45 minutes ([FinalRound guide](https://www.finalroundai.com/blog/security-engineer-interview-guide), [KORE1](https://www.kore1.com/application-security-engineer-interview-questions/)). Meta runs it as a phone-screen round and again onsite on different applications ([Glassdoor](https://www.glassdoor.com/Interview/Meta-Security-Engineer-Interview-Questions-EI_IE40772.0,4_KO5,22.htm)); Amazon opens with "go over a specific product and how to threat model for this scenario" ([Glassdoor](https://static.glassdoor.com.br/Interview/Amazon-Interview-E6036-RVW33223721.htm)); Jane Street's HK SOC loop called it "system-analysis whiteboarding" and the Hong Kong security loop had a secure-system-design round ([block 01](../01-process/README.md)). The Threat Modelling posting makes it near certain for you ([block 22](../22-threat-modelling-role/README.md)). What follows is the run, with the clock.

---

## What they are scoring

Not whether you find every threat. Whether you are systematic rather than reactive, whether you ask before you assert, whether you can rank and decide, and whether the system owner would want you back in the room. The published rubric from a 2026 hiring guide, for the threat-model row ([KORE1](https://www.kore1.com/application-security-engineer-interview-questions/)):

| Level | What it looks like |
| --- | --- |
| Mid | Lists sensible threats once the interviewer points at the trust boundary |
| Senior | Opens with questions for the product owner, ranks the risks, names one they would accept |
| Walk-away | A completed STRIDE grid and not a single question |

Jane Street adds its own weights: collaboration and asking good questions are stated values, humility about what you do not know is scored, and measured beats maximal ([block 00](../00-jane-street-intel/README.md), [block 11](../11-judgement-and-hygiene/README.md)).

---

## The clock

### 0 to 5 minutes: scope, out loud

Do not touch the pen yet. Ask, and write the answers as assumptions on the corner of the board:

- Who uses this, and who must never be able to? (Users, admins, third parties, other internal teams, agents.)
- What is the most valuable thing in it, and what is the worst thing that could happen? (Gets the owner to name the crown jewel for you.)
- Where does it run, and what does it talk to? (Hosting, networks, dependencies, the identity system.)
- What already exists? (Existing controls you should not re-invent; where authentication and authorisation actually happen.)
- What is in scope today, and what are we deliberately not modelling? (Say it, write it, move on.)
- Anything specific to the request: can this link be forwarded; how long does the token live; what does revocation mean once it is in someone's inbox ([KORE1](https://www.kore1.com/application-security-engineer-interview-questions/)).

Then one sentence: "I'll use the four questions as the structure: what we're building, what can go wrong, what we do about it, and how we'd know it worked."

### 5 to 12 minutes: draw it

A data-flow diagram, one board, four shapes, and trust boundaries as dashed lines drawn *before* threats ([frameworks.md](frameworks.md)). Label every arrow with what moves and over what. Mark where each side authenticates the other. Ask the owner "is this right?" at least once; if you have drawn something they correct, thank them and fix it: that exchange is the collaboration being scored.

Two moves that read as senior: draw the path that was not described (admin access, deploy pipeline, backups, logging, the agent that will eventually touch it) and ask whether it exists; and circle the crown jewel.

### 12 to 25 minutes: what can go wrong

Walk each boundary crossing and say the attacker's path, not the category: "an outsider who gets this bearer token from a proxy log can read any report it points at." Then sweep with STRIDE on the elements that matter to catch what you missed, saying the letters you are checking. Aim for eight to twelve real threats, not forty. Write each as one line on the board next to its flow, so the picture carries the argument.

For a trading-firm system, the business-logic threats are where you win: the route that bypasses pre-trade risk, the group rule that crosses an information barrier, the data feed that can be corrupted upstream ([hedge-fund-threat-model.md](hedge-fund-threat-model.md)). Say at least one of those; most candidates only find technical ones.

### 25 to 33 minutes: rank and decide

State the method, then apply it: likelihood (reachable, known technique, exposed to untrusted input, attacker interest) against impact on the named crown jewels. Pick the top three. **Name one you would accept, and why**, and who would own that acceptance. Then the senior move the rubric singles out: look for a design change that removes a threat rather than mitigating it ("what if the link were scoped to one report and one recipient").

### 33 to 40 minutes: what we do, and how we would know

For each of the three: the control, where it goes in the diagram, and a verification: a test, a detection rule, a policy assertion in CI ([block 21](../21-detection-engineering/README.md)). Say the trade-off for anything that costs the owner something (latency, friction, a migration). Mention what you would skip and why.

### 40 to 45 minutes: close

Three actions with owners, one line each. One sentence on what you did not get to and would do next. Ask the owner what they think you missed. End on time; running over reads as not owning the room.

---

## Narration that works

Keep talking. The interviewer cannot score silence. Phrases that structure your thinking and sound like a teammate, not a lecturer:

- "Before I draw anything, three questions."
- "I'm going to draw the boundaries first because that's where the threats live."
- "Let me say the attacker's path, then I'll check STRIDE to see what I've missed."
- "That's a real threat but I'd rank it below these two, because..."
- "I'd accept this one, and here's why; it should be a named decision, not a silent one."
- "Can we change the feature instead of defending it?"
- "The test for this control is..."
- "What have I missed? You know this system better than I do."

---

## When the owner pushes back

The reader in your mock (or the interviewer) will say "that's not a real risk, we have X." Your response is being scored as much as your threats:

1. **Take it seriously.** "Good, tell me how X works here." They may know a control you did not; the posting literally describes system owners who know their application.
2. **Check whether X actually covers the path.** Often it covers the obvious variant and not the one you named. Say which.
3. **If they are right, change your model visibly** and thank them. Conceding well is a positive signal at a firm that scores humility.
4. **If the risk stands, demonstrate rather than assert.** Walk the exact steps an attacker would take past X. Then offer the smallest fix.
5. **Never escalate or cite policy** as the first move. The relationship is the control.

---

## The attacker-first variant (for the Threat Modelling role)

Because the posting requires an offensive background, expect the exercise to open with "how would you attack this" before "how would you defend it". Same clock, different order: after scoping and drawing, give the attacker's first three moves as paths from an entry point to a crown jewel, in the order a real attacker would take them, then switch hats and break each path at its cheapest point. The vocabulary per surface is in [block 22](../22-threat-modelling-role/README.md). Always finish on the defender's side with a ranking; the probe tests whether you can hold both views.

---

## Variants you may meet

- **No system given, "pick one you know".** Choose something you can draw in two minutes and that has real boundaries: the exchange-connectivity change in [worked-models.md](worked-models.md) is yours to own, or your own DAST verifier.
- **A feature that does not exist yet** ("customers can email a read-only link to a report"): most of the score is in the questions you ask; the threats are quick once you know the answers ([KORE1](https://www.kore1.com/application-security-engineer-interview-questions/)).
- **A hardware or firmware system** (Google asks "threat model a new server motherboard"): the common pitfall is giving a software answer; name DMA, firmware, supply chain, side channels and physical access ([block 07](../07-windows-and-endpoints/README.md) hardware section, [dev.to summary](https://dev.to/jackm_345442a09fb53b/google-security-engineer-data-center-platforms-technical-infrastructure-interview-questions-42i3)).
- **An LLM feature** ("our support bot can issue refunds through a tool call"): the senior line is "who sets the refund limit, the model or the code?" ([KORE1](https://www.kore1.com/application-security-engineer-interview-questions/), [block 16](../16-ai-and-llm-security/README.md)).
- **Their own system.** If they draw Jane Street infrastructure, you have an unfair advantage: the public talk on their permissions service, Ontology, is modelled in [worked-models.md](worked-models.md). Do not announce that you prepared it; let it show in the quality of the questions.

---

## Mistakes that cost the round

- Reciting STRIDE before asking anything.
- Threats as categories ("tampering") instead of paths ("a modified feed message changes the price the strategy sees").
- Forty threats and no ranking.
- No accepted risk, as if everything must be fixed.
- Forgetting repudiation (audit) and the admin and deploy paths.
- Staying at architecture altitude when asked for the mechanism; the posting wants whiteboard and hands-on in the same person.
- Proposing controls that add latency to a trading path without saying so ([block 17](../17-trading-domain/README.md)).
- Lecturing the owner instead of asking them.
- Pitching your own product.

---

## Self-grading checklist (use after every mock)

- [ ] Asked at least four scoping questions before drawing.
- [ ] Named the four questions as the structure.
- [ ] Drew boundaries before threats; labelled flows with data and protocol.
- [ ] Found the crown jewel and the undrawn paths (admin, deploy, logs, agents).
- [ ] Stated threats as attacker paths; used STRIDE only as a sweep.
- [ ] Found at least one business-logic threat, not just technical ones.
- [ ] Ranked with a stated method; picked three; named one accepted risk with an owner.
- [ ] Proposed at least one design change that eliminates a threat.
- [ ] Gave a verification for each mitigation.
- [ ] Handled pushback by checking, not conceding or insisting.
- [ ] Finished on time with three owned actions and a question back to the owner.

[← Threat modelling hub](README.md) · [Frameworks](frameworks.md) · [Worked models](worked-models.md) · [Hedge fund](hedge-fund-threat-model.md) · [Questions](interview-questions.md)
