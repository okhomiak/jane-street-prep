# Threat-modelling interview questions, reported, with answer shapes

[← Threat modelling hub](README.md) · [Frameworks](frameworks.md) · [Playbook](live-exercise-playbook.md) · [Worked models](worked-models.md) · [Hedge fund](hedge-fund-threat-model.md)

Everything here was reported by a candidate or published by a hiring team, with the source. The exercise prompts are run with the [playbook](live-exercise-playbook.md); the conceptual and behavioural ones get the answer shape below. Where a question is a Jane Street one, it is marked [JS]. Cover the right column and say yours out loud first.

---

## Exercise prompts that were actually given

| Prompt | Where | Source | Run it as |
| --- | --- | --- | --- |
| Threat model a Signal-type messaging application | Meta Security Engineer (final round, May 2025; the phone screen is one TM round plus one coding round) | [Glassdoor](https://www.glassdoor.co.in/Interview/Meta-Interview-E40772-RVW3032057.htm) | [Worked model 4](worked-models.md) |
| Threat model a sample application (phone screen, Aug 2025); final-round technical interviews "focused on threat modeling different types of applications" (May 2025) | Meta Security Engineer | [Glassdoor](https://www.glassdoor.co.in/Interview/Meta-Interview-E40772-RVW3032057.htm) | Playbook, any model |
| Go over a specific product and how to threat model for this scenario (one-hour phone screen with network questions) | Amazon/AWS Security Engineer | [Glassdoor](https://static.glassdoor.com.br/Interview/Amazon-Interview-E6036-RVW33223721.htm) | Playbook, scope questions first |
| Threat modelling, STRIDE, XSS and its types | Amazon Security Engineer, 2026 | [Glassdoor](https://static.glassdoor.com.mx/Interview/Amazon-Interview-E6036-RVW91855346.htm) | [Frameworks](frameworks.md) STRIDE section |
| Do a threat model for a storage system using presigned URLs for upload and retrieval, UUID keys, one bucket; five threats and remediations, ranked | Staff Security Engineer, Render (Jul 2024) | [Glassdoor](https://static.glassdoor.com.ar/Interview/security-engineer-interview-questions-SRCH_KO0,17_IP87.htm) | [Worked model 3](worked-models.md) |
| Photo application on a cloud storage bucket for an external vendor (cloud TM scenario) | AppSec loop (Blind) | [Blind](https://www.teamblind.com/post/appsec-interview-vqjssoax) | Model 3 with a vendor boundary added |
| Walk through how you would threat model a new Azure service before launch | Microsoft Security Engineer (threat-modelling round) | [CleverPrep](https://www.cleverprep.com/companies/microsoft/security-engineer) | Playbook; emphasise the four questions and pre-launch verification |
| Threat model a new feature (the design round) | Stripe Security Engineer | [CleverPrep](https://www.cleverprep.com/companies/stripe/security-engineer) | Playbook; feature-that-does-not-exist variant |
| Open-ended threat modelling questions | Salesforce Security Engineer | [Glassdoor](https://www.glassdoor.com/Interview/Open-ended-threat-modeling-questions-QTN_1811235.htm) | Playbook |
| The technical interview consisted of a threat modelling exercise | Job&Talent Cyber Security Engineer | [Glassdoor](https://www.glassdoor.com/Interview/The-technical-interview-consisted-of-a-threat-modeling-exercise-QTN_6432748.htm) | Playbook |
| Threat model Alexa; then a live code review; then "what do you do if your tools give many false positives" | IT Security Engineer | [Glassdoor](https://static.glassdoor.de/Interview/it-security-engineer-interview-questions-SRCH_KO0,20_IP302.htm) | Device plus cloud backend; voice as untrusted input; see also [block 21](../21-detection-engineering/README.md) for the false-positive answer |
| Customers can email a read-only link to a report so a contractor can view it without an account; threat model it (interviewer silent for ten minutes) | 2026 AppSec hiring guide | [KORE1](https://www.kore1.com/application-security-engineer-interview-questions/) | Feature variant; the questions are the score |
| Our support bot can issue refunds through a tool call; threat model it | same | same | [Worked model 5](worked-models.md), [block 16](../16-ai-and-llm-security/README.md) |
| Threat model a new server motherboard for our data centres | Google Security Engineer, data-centre platforms | [dev.to summary](https://dev.to/jackm_345442a09fb53b/google-security-engineer-data-center-platforms-technical-infrastructure-interview-questions-42i3) | Hardware variant; [block 07](../07-windows-and-endpoints/README.md) |
| Identify the attack surface, enumerate threat actors and propose mitigations for a simplified internal system | Google onsite TM round, as described by prep guides | [FinalRound](https://www.finalroundai.com/blog/security-engineer-interview-guide) | Playbook |
| How would you approach threat modelling a new server architecture | Supermicro Security Engineer | [Dataford](https://dataford.io/interview-guides/super-micro-computer/security-engineer) | Hardware plus firmware plus supply chain |
| Threat model an S3 bucket versus an EC2 instance; apply STRIDE to a system design; a smart-home device; an internet-connected vending machine; a secure image-upload service; a newly acquired company's infrastructure; design a system then attack your own design | Amazon, from prep guides (unverified individually) | [InterviewKickstart](https://interviewkickstart.com/interview-questions/amazon-security-engineer-interview-questions), [Dataford](https://dataford.io/interview-guides/amazon/security-engineer) | Playbook; the acquisition one is a scoping exercise first |
| System-analysis whiteboarding; secure system design round; tabletop | [JS] Hong Kong security loops | [block 01](../01-process/README.md) | Playbook; [hedge fund](hedge-fund-threat-model.md) framing |
| How would you connect your new startup to the French exchange | Optiver Security Engineer | [block 17](../17-trading-domain/README.md) | [Worked model 2](worked-models.md) |
| Design a vulnerability management programme | Optiver, Jan 2026 | [block 10 hub](README.md) | The VM prompt in the hub |

---

## Conceptual questions, with the answer shape

<details>
<summary><b>What is threat modelling? Which process do you follow? How do you perform it?</b> · Genesys Senior AppSec, Coveros AppSec, Glassdoor banks</summary>

Analysing a representation of a system to find what can go wrong and decide what to do about it, before it is built or changed. Process: the four questions ([frameworks.md](frameworks.md)), with a data-flow diagram and trust boundaries for the first, attack paths plus a STRIDE sweep for the second, mitigate-eliminate-transfer-accept with owners for the third, and tests and detections for the fourth. Say that you adapt depth to the system: an hour at a whiteboard for a feature, a continuous model-as-code for a crown-jewel system. Sources: [Glassdoor Genesys](https://www.glassdoor.com/Interview/What-is-Threat-Modeling-Which-Threat-Modelling-process-you-follow-Explain-How-do-you-perform-threat-modeling-What-is-QTN_4593697.htm), [Glassdoor Coveros](https://www.glassdoor.com/Interview/Do-you-know-what-a-threat-model-is-Tell-me-about-it-QTN_3609040.htm).
</details>

<details>
<summary><b>Describe the benefits of threat modelling. Threat modelling versus vulnerability scanning versus penetration testing.</b> · Glassdoor banks, Senior Threat Modeler lists</summary>

Threat modelling finds design flaws before code exists and tells you where to spend testing effort; scanning finds known vulnerabilities in what you have deployed; penetration testing finds what a skilled attacker can actually do today. They answer different questions at different times, and a mature programme uses the model to aim the other two. The crisp line: scanners cannot find a missing authorisation check in a design, and a pentest finds it after you have shipped it. Source: [Ouroboros list](https://ouroboros-solutions.gitbook.io/offsec-wiki/threat-modeling/howtos).
</details>

<details>
<summary><b>What is Application Security Architecture Review, and how does it relate?</b> · Genesys</summary>

A review of a system's design against security principles and the firm's standards, usually at a design gate; threat modelling is the method that makes it more than a checklist, because it derives requirements from the system's own data flows rather than from a generic list.
</details>

<details>
<summary><b>Proactive versus reactive threat modelling.</b> · Practical DevSecOps list</summary>

Proactive is at design time, before code; reactive is after an incident or a finding, to work out what else the same flaw pattern touches. Both are legitimate; the reactive one is how you turn an incident into a library entry ([hedge-fund-threat-model.md](hedge-fund-threat-model.md) abuse cases).
</details>

<details>
<summary><b>Which framework do you use and why? What is wrong with DREAD?</b></summary>

Four questions as structure, STRIDE as a sweep, attack trees for a specific crown jewel, ATT&CK to ground threats in real adversaries. DREAD produced inconsistent scores between people and Microsoft dropped it; rank with decomposed likelihood against the asset taxonomy instead ([frameworks.md](frameworks.md)).
</details>

<details>
<summary><b>How do you involve stakeholders, and how do you communicate results to non-technical ones?</b> · Practical DevSecOps, Yardstick</summary>

Involve the system owner as the person who runs the session; they know the system, you bring the attacker. Communicate in their terms: what could happen to the thing they care about, what it would cost, what we are doing and what we are choosing to accept. One page, three actions, owners and dates. Never a forty-row spreadsheet.
</details>

<details>
<summary><b>How do you prioritise which systems to threat-model first across a firm?</b></summary>

By the asset taxonomy and attack paths: the systems that hold or move the crown jewels first (order path, research environment, permissions, payments), then anything exposed to untrusted input, then by rate of change. Say you would not try to model everything; you would model the ten that matter and make the rest cheap and self-service ([block 23](../23-ai-threat-modelling/README.md)).
</details>

<details>
<summary><b>How do you measure whether threat modelling is working?</b> · Meta-style programme question</summary>

Outcomes, not counts: findings fixed before launch versus found after; incidents whose root cause a current model had already named (and the ones it had not, which go into the library); mitigations with a passing verification; time from design to a reviewed model. The honest caveat from a published model answer: a status field saying "has a threat model" proves nothing if the document no longer matches the design ([interviews.chat](https://www.interviews.chat/questions/meta-security-engineer)).
</details>

<details>
<summary><b>What tools do you use?</b></summary>

Whiteboard first. Then whichever fits the team: Threat Dragon or the Microsoft tool for drawing, pytm or Threagile for model-as-code in the repo, IriusRisk or SD Elements where an enterprise wants workflow. Your own product once, as a scaling tool, then back to the practice ([block 22](../22-threat-modelling-role/README.md)).
</details>

<details>
<summary><b>What steps would you take if a serious threat is identified during threat modelling?</b> · AppSec question lists</summary>

Confirm it is real against the actual system (not just the diagram), rank it, get it in front of the owner the same day with the smallest effective fix, and if it is exploitable in production now, treat it as an incident ([block 09](../09-incident-response/README.md)), not a design finding.
</details>

---

## Behavioural questions about threat modelling

These are the shape of the hiring-manager and "tell me about a time" probes for a threat-modelling role ([Yardstick](https://yardstick.team/interview-questions/threat-modeling)). Each needs one real story from your record; the [block 13](../13-behavioural/README.md) story bank and the ADR shape in [block 22](../22-threat-modelling-role/README.md) feed them.

- A threat model that changed a design or architecture significantly. What you found, how you convinced them, what shipped.
- A disagreement between stakeholders about a risk's severity. How you resolved it; what you accepted.
- Explaining a complex finding to a non-technical stakeholder.
- Integrating threat modelling into a team's development process so it survived your absence (the anti-hero pattern).
- Threat-modelling a third-party component or vendor.
- A threat model that prevented an incident, or an incident that a model should have prevented and what you changed.
- Revising a model after the system changed, and keeping continuity.
- Which components you focused on first and what you deliberately left out.

---

## The wrong answers interviewers hear most

- Starting with STRIDE instead of with questions.
- "Threat modelling is a document we produce at design review."
- Threats as categories with no path and no asset.
- Everything is high; nothing is accepted.
- Only technical threats; no business-logic threat.
- No verification; the model ends at the mitigation list.
- Proposing an inline control on the order path without naming the latency cost.
- Describing a tool when asked about a practice.
- Repeating the 2014 hedge-fund HFT story as fact ([hedge-fund-threat-model.md](hedge-fund-threat-model.md)).

[← Threat modelling hub](README.md) · [Frameworks](frameworks.md) · [Playbook](live-exercise-playbook.md) · [Worked models](worked-models.md) · [Hedge fund](hedge-fund-threat-model.md)
