# Frameworks, critically: what each one is for, and what interviewers actually want to hear

[← Threat modelling hub](README.md) · [Playbook](live-exercise-playbook.md) · [Worked models](worked-models.md) · [Hedge fund](hedge-fund-threat-model.md) · [Questions](interview-questions.md)

A Blind thread from a candidate expecting threat-modelling questions sums up the trap: every reply said "go read STRIDE and MITRE", and the candidate replied that they already knew STRIDE and wanted to know what threats to actually enumerate ([Blind](https://www.teamblind.com/post/Threat-Modeling-tips-n7nw6Mnd)). Frameworks are not the skill. They are checklists you reach for *after* you understand the system, so you do not forget a category. This file gives you each one in the depth an interviewer probes to, and says plainly which to lead with. Tags: [est] established, [JS] where it touches Jane Street.

---

## The spine: four questions

Everything else hangs off Shostack's four questions, adopted by the Threat Modeling Manifesto as the common frame ([Manifesto](https://threatmodelingmanifesto.org/), [Shostack's white paper](https://shostack.org/files/papers/The_Four_Question_Framework.pdf)):

1. **What are we working on?** Scope, a representation (usually a data-flow diagram), assets, actors, trust boundaries.
2. **What can go wrong?** Threats, found with whatever method fits (STRIDE, attack trees, ATT&CK, experience).
3. **What are we going to do about it?** Mitigate, eliminate (change the design), transfer, or accept, each with an owner.
4. **Did we do a good enough job?** Verification: tests, detections, review, and whether the model still matches the system.

Say these out loud at the start of any exercise. They are the structure interviewers are listening for, and they give you a graceful way back when you lose the thread ("let me go back to what can go wrong on this boundary").

The Manifesto also names the **anti-patterns**, and they are interview gold because they show you have run this for real: the **hero threat modeller** (one expert does it all; the fix is teaching the team), **admiration for the problem** (lists of threats with no fixes), **tendency to overfocus** (perfecting one corner while the rest is unmodelled), and the **perfect representation** (chasing a flawless diagram instead of a useful one). Its positive patterns are a systematic approach, informed creativity (method plus imagination), varied viewpoints (the team sport), a useful toolkit, and theory into practice.

---

## STRIDE: the one you must own completely

Microsoft's mnemonic for six threat categories, each the inverse of a property ([est]):

| Letter | Threat | Property violated | The question to ask of an element |
| --- | --- | --- | --- |
| S | Spoofing | Authentication | Can someone pretend to be this user, service or component? |
| T | Tampering | Integrity | Can the data or code be modified in transit or at rest without detection? |
| R | Repudiation | Non-repudiation | Can someone deny having done this, because we cannot prove it? |
| I | Information disclosure | Confidentiality | Can data leak to someone who should not see it? |
| D | Denial of service | Availability | Can this be made unavailable or degraded? |
| E | Elevation of privilege | Authorisation | Can someone do more than they are allowed? |

**STRIDE-per-element** applies the six to each component, data store and data flow; the rule of thumb is that external entities attract S and R, processes attract all six, data stores attract T, I, D (and R for logs), and data flows attract T, I, D. **STRIDE-per-interaction** applies them to each pair of elements communicating across a boundary, which is tighter and produces fewer duplicates. In an interview, say which you are using and why (per-interaction for a system with few, important boundaries; per-element when you need coverage fast).

How interviewers probe it: "give me a repudiation threat for this system" (candidates skip R; the answer is usually a missing or tamperable audit trail), "is that really spoofing or tampering" (be precise: a forged request is spoofing the sender; a modified request is tampering), and "which of those six matters most here" (rank; do not treat them as equal).

**The failure mode to avoid is reciting the grid.** The 2026 hiring guide that published its scoring says a completed STRIDE table with no questions asked is the walk-away answer, and that a senior candidate opens with questions to the product owner, ranks the risks, and names one they would accept ([KORE1](https://www.kore1.com/application-security-engineer-interview-questions/)). STRIDE is how you check you did not miss a category. It is not how you start.

---

## Data-flow diagrams and trust boundaries: the representation

Four shapes are enough ([est]): **external entity** (a user, a third party, another firm's system; a rectangle), **process** (your code; a circle), **data store** (a database, a file, a queue; parallel lines), **data flow** (an arrow, labelled with what moves). Then **trust boundaries**: dashed lines where the level of trust changes (internet to DMZ, user to service, service to database, one team's system to another's, human to agent). Threats live where flows cross boundaries, so draw the boundaries first and the threats find themselves.

Interview habits that read as senior: label every flow with the data and the protocol; mark which side authenticates which; draw the thing that is *not* in the diagram and ask about it ("where does the admin path come in"); keep it to one whiteboard. Thoughtworks' agile guide calls this a team sport done at a whiteboard with sticky notes on the flows, and that register, collaborative and fast, is exactly Jane Street's ([Fowler/Thoughtworks](https://martinfowler.com/articles/agile-threat-modelling.html)).

---

## Attack trees: when the question is "how would they get X"

A root goal ("read the research repo as an outsider") with AND/OR branches of sub-goals down to concrete steps, optionally annotated with cost, skill, and detectability. Use them when the system is understood and the question is about a specific crown jewel, or when you need to argue which control cuts the most paths ([est]). They are the natural tool for the offensive probe in the Threat Modelling role ([block 22](../22-threat-modelling-role/README.md)): say the attacker's goal, draw three branches, show which single control prunes two of them. CTID's "Threat Modeling with ATT&CK" uses attack trees as its worked methodology.

---

## ATT&CK-based threat modelling: grounding "what can go wrong" in real adversaries

MITRE's Center for Threat-Informed Defense published a process for integrating ATT&CK into any methodology, with partners that included JPMorgan Chase, Citigroup and Verizon ([CTID project](https://ctid.mitre.org/projects/threat-modeling-with-attack/), [overview](https://medium.com/mitre-engenuity/turn-your-threat-model-to-supermodel-with-att-ck-a6dfaa6787c2)). The idea: answer question two with observed adversary techniques rather than imagination alone, map your defensive capabilities to the same techniques, and the overlay shows residual risk per technique. For a trading firm it also lets you say which actors actually target you (financially motivated crime, state-linked groups going after financial infrastructure) and what their current initial-access patterns are ([block 21](../21-detection-engineering/README.md)). Mention that your current employer was a partner on that work if it fits; do not labour it.

---

## PASTA, LINDDUN, and the rest: know what they are for, in one line each

- **PASTA** (Process for Attack Simulation and Threat Analysis): seven stages from business objectives through technical scope, decomposition, threat analysis, vulnerability analysis, attack modelling, to risk and impact. Risk-centric and business-aligned; heavyweight. Say you would borrow its "start from business impact" opening for a crown-jewel system and not run all seven stages for a feature.
- **LINDDUN**: the privacy counterpart to STRIDE (Linking, Identifying, Non-repudiation, Detecting, Data disclosure, Unawareness, Non-compliance). Use when personal data is the asset. Less central at a proprietary trading firm than at a consumer company; still worth naming when staff or counterparty data is in scope.
- **Kill chain and ATT&CK** describe an attacker's progression; useful to check a model covers persistence, lateral movement and exfiltration, not just initial access.
- **CAPEC**: a catalogue of attack patterns, useful as a library when enumerating; nobody expects you to recite it.
- **Trike, VAST, OCTAVE**: risk-management and enterprise-scale methodologies. Know the names; say that at the scale of one system they add process without adding threats.
- **DREAD**: a scoring scheme (Damage, Reproducibility, Exploitability, Affected users, Discoverability) that Microsoft itself stopped using because the scores were subjective and inconsistent between people. If asked, say that, and say what you use instead (below).

---

## Ranking: the part that separates senior from mid

Everyone can list threats. The job is deciding which three to fix this quarter. State your method and apply it consistently:

- **Likelihood × impact, with likelihood decomposed** into exposure (is the path reachable, is there a known technique, is it exposed to untrusted input) and attacker interest (who wants this and how hard will they work). Impact is against the asset taxonomy: at a trading firm the order path and the alpha outrank almost everything ([hedge-fund-threat-model.md](hedge-fund-threat-model.md)).
- **Do not rank with CVSS.** It scores a vulnerability's severity in the abstract, not your risk; the vulnerability-management prompt in the [hub](README.md) makes the same point.
- **Name one you accept, with the reason**, and name who accepted it. That single move is the one the published rubric singles out as senior.
- **Prefer eliminating to mitigating**: the strongest answer to "how do we protect this bearer token in a URL" is often "change the feature so the link is scoped to one report and one recipient" ([KORE1](https://www.kore1.com/application-security-engineer-interview-questions/)). Jane Street's measured-judgement culture rewards that over adding controls ([block 11](../11-judgement-and-hygiene/README.md)).

---

## Making threats testable: ASVS and verification

A threat without a verification is an opinion. Pull requirements from the OWASP ASVS (version 5.0 shipped in 2025) so each mitigation maps to something a test can check, and for infrastructure threats write the detection that would fire ([block 21](../21-detection-engineering/README.md)). This is question four of the four, and it is the part most candidates never reach; reaching it is how you show the model is an engineering artefact, not a document.

---

## Threat modelling as code, and the tools

Know the landscape, and keep the hierarchy straight: whiteboard first, tools second. **Microsoft Threat Modeling Tool** and **OWASP Threat Dragon** draw DFDs and generate STRIDE threats; **IriusRisk** and **SD Elements** are enterprise platforms with threat libraries and workflow; **pytm** and **Threagile** define the model in code (Python or YAML) so it lives in the repo, diffs in pull requests, and runs in CI. For a firm that puts everything in a monorepo, model-as-code is the natural fit, and you can say so. Your own product sits in this landscape; mention it once, as a tool you built because the manual version did not scale, and move on ([block 22](../22-threat-modelling-role/README.md) on not over-indexing).

---

## What to lead with, in order

If an interviewer says "threat model this", the order that reads as senior:

1. Questions to scope it (two minutes).
2. The four questions named as your structure (one sentence).
3. Draw the DFD with trust boundaries (five minutes), labelling data and protocols.
4. Assets and actors, with the crown jewels marked.
5. Attack paths across each boundary, then STRIDE as a sweep for anything missed.
6. Rank, pick three, name one you accept.
7. Mitigations with a verification each, and one design change if there is a good one.
8. "Did we do a good job": what you would check in three months.

Frameworks appear at step five and nowhere else. That is the whole point of this file.

[← Threat modelling hub](README.md) · [Playbook](live-exercise-playbook.md) · [Worked models](worked-models.md) · [Hedge fund](hedge-fund-threat-model.md) · [Questions](interview-questions.md)
