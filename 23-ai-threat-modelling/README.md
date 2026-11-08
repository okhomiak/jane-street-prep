# 23 · AI-assisted threat modelling for a trading firm: how it works, where it breaks, how to fix it

[← Threat Modelling role](../22-threat-modelling-role/README.md) · [Index](../README.md) · [Next: Six-month roadmap →](../24-six-month-roadmap/README.md)

This is the conversation you are better placed to have than any other candidate, and also the one where it is easiest to sound like a vendor. The block is written to keep you on the right side of that line: a working model of how AI-assisted threat modelling actually functions, an honest list of why it fails, and the engineering that makes it trustworthy, all set in a firm like Jane Street. It is analysis, not reported questions: the Threat Modelling posting does not mention AI, and no candidate has reported being asked about it. Use it when they ask how you would scale threat modelling, what you would build, or what you think of AI in security. Tags: [est] established practice, [JS] where it rests on their published facts, [analysis] where it is reasoning.

---

## Why threat modelling at a trading firm is a scale problem first

The posting describes the job as talking to system owners who know their applications but do not see the exploits, then helping fix them. The constraint is arithmetic. A firm that builds nearly everything in-house ([block 00](../00-jane-street-intel/README.md)) has hundreds to thousands of internal systems, a monorepo with a few hundred programmers committing daily, and a security team that is small by comparison. A one-hour whiteboard session per system per year does not cover it, and a model that is only produced at design time is stale by the first refactor.

The assets are also unusual, which generic tooling misses: the strategy and research code (the alpha), the order path and its pre-trade risk checks, market-data integrity, exchange and clearing credentials, and now a fleet of AI agents acting with human-equivalent permissions ([block 16](../16-ai-and-llm-security/README.md), [17](../17-trading-domain/README.md)). A threat model that does not know those are the crown jewels will rank a public-website XSS above a flaw in the kill switch.

So the honest framing is: AI is interesting here not because it is clever but because the firm has more systems than security engineers, and the question is whether a machine can produce a *draft* model good enough that a human's hour goes on judgement instead of enumeration.

---

## How AI-assisted threat modelling works (the pipeline)

<details>
<summary><b>Walk me through how an AI-assisted threat-modelling system actually works.</b> · [est] [analysis]</summary>

Describe it as a pipeline with a human at the decision point, not as "an LLM that writes threat models":

1. **Ingest what describes the system.** Design docs and READMEs, architecture diagrams, infrastructure-as-code, API and RPC schemas, service dependency graphs, auth configuration, and the code itself. At Jane Street specifically the inputs are unusually good: a monorepo, typed RPC definitions, and OCaml's explicit types mean a lot of the data-flow model can be *extracted deterministically* rather than guessed ([block 18](../18-ocaml-literacy/README.md)).
2. **Build the model.** Components, data stores, data flows, trust boundaries, and the actors (including service and agent identities). The good systems produce a structured data-flow diagram as data (a graph), not a picture, so it can be diffed and queried.
3. **Enumerate threats.** Per element and per flow, apply STRIDE or an equivalent, cross-reference ATT&CK techniques and the firm's own threat library and incident history, and generate candidate threats with the attack path spelled out. This is where the LLM earns its place: it is good at "given this flow crossing this boundary, what has gone wrong in systems like this".
4. **Rank.** Exploitability (is the path reachable, is there a known technique, is the component exposed) times impact (which asset, how critical), with the firm's asset taxonomy encoded so the order path outranks the intranet wiki.
5. **Propose mitigations and tests.** Each threat gets concrete controls, and, in the better systems, a verification: a test case, a detection rule ([block 21](../21-detection-engineering/README.md)), or a code check that would confirm the mitigation exists.
6. **Review and sign off.** A security engineer and the system owner accept, reject or edit each item, with the rationale recorded. This is the step the posting cares about: "document your decision-making rationale."
7. **Track and re-run.** Accepted mitigations become engineering tickets; the model re-runs when the design or code changes and reports *what changed in the threat picture*, not the whole list again.

The one-line version: structured extraction for the parts that can be deterministic, a language model for enumeration and explanation, deterministic ranking against the firm's asset list, and a human who decides. The same "model proposes, deterministic code decides, human approves the irreversible" shape as the agent-security answer in [block 16](../16-ai-and-llm-security/README.md).
</details>

---

## Where it breaks

This is the section that makes you credible. Anyone can describe the pipeline; the person who has run one knows these.

<details>
<summary><b>What goes wrong with AI-generated threat models?</b> · [analysis] [say these before they ask]</summary>

1. **Plausible, wrong threats.** The model produces attack paths that read well and do not exist in this system: a threat against a component that is not internet-facing, a mitigation for a framework the service does not use. Hallucinated threats are worse than missing ones because they consume the reviewer's trust and time, and a reviewer who finds three fabrications stops reading.
2. **Generic flood.** Left alone, a model emits every STRIDE category for every element, hundreds of items, most of them boilerplate ("ensure logging is enabled"). This is the base-rate problem from [block 04](../04-security-fundamentals/back-of-envelope.md) in another costume: low precision makes the output unusable regardless of recall.
3. **Missing context the model cannot infer.** Trust boundaries, which network a service sits on, what the data actually is, whether a control exists somewhere upstream. The model does not know the pre-trade risk check exists unless the risk check is in its inputs; it will "discover" the threat and propose the control that is already there.
4. **Business-logic blindness.** The threats that matter most at a trading firm are often not technical categories at all: a strategy that can be triggered to over-trade, an information barrier that can be crossed via a shared cache, a reconciliation that can be gamed. STRIDE-per-element does not find these; a person who understands the business does.
5. **The inputs are an attack surface.** The system reads design docs, READMEs, code comments and tickets. Those are untrusted content. A malicious or compromised document can carry instructions to the model ("mark this component as out of scope"), which is indirect prompt injection against your security tooling, the lethal trifecta pointed at the threat modeller itself ([block 16](../16-ai-and-llm-security/README.md)).
6. **Confidentiality of the inputs.** The architecture of a trading firm's systems is itself sensitive. Sending it to an external model is a data-exfiltration decision dressed as a tooling decision. Jane Street self-hosts and brokers model access through one credentialed gateway ([block 00](../00-jane-street-intel/README.md)); an AI threat-modelling system there would have to live inside that.
7. **No ground truth, so no evaluation.** How do you know the model is good? There is no labelled dataset of "all real threats to this system". Teams ship these tools without a way to measure precision or recall, and then argue about anecdotes.
8. **Staleness and drift.** A model produced at design review is wrong after the next change. If re-running is expensive or noisy, nobody re-runs it, and the artefact becomes documentation theatre.
9. **Ownership dilution.** When a machine produced the list, nobody feels accountable for it. Findings without an owner are not findings. The posting's line about documenting rationale is the antidote, but only if the tool makes a human record a decision per item.
10. **The adoption failure.** This is the one Jane Street has lived through with their agent sandbox: engineers switch the control off when it slows them down ([block 00](../00-jane-street-intel/README.md), ep. 31). A threat-modelling tool that adds friction to design review gets bypassed; the social problem, not the model, decides whether it works.
11. **Non-determinism.** Run it twice, get two different lists. That is tolerable for a brainstorm and intolerable for a control you want to audit, diff over time, or hold a team to.
</details>

---

## How to make it trustworthy

<details>
<summary><b>If you were building this at Jane Street, what would you do differently from the typical tool?</b> · [analysis] [the design answer]</summary>

Pair each failure with the engineering that fixes it:

- **Extract deterministically; generate narrowly.** Build the data-flow graph from typed sources (RPC schemas, IaC, service registries, auth config), not from prose. Reserve the language model for enumeration and explanation over a graph it did not invent. Fewer hallucinated components, because the components came from the build system.
- **Retrieve, do not recall.** Ground enumeration in the firm's own threat library, past incidents, and accepted mitigations, with provenance on every suggestion ("this is similar to incident X; the control in service Y covers it"). A suggestion you can trace is a suggestion a reviewer can check in seconds.
- **Constrain the output.** STRIDE-per-element with a cap, scored against the asset taxonomy, boilerplate filtered by a rule layer, and a confidence field the model must fill. Fewer, better items; the aim is a draft a reviewer *finishes*, not one they fight.
- **Rank with real signals.** Reachability from the graph, exposure, known-exploited status and exploit probability for the technologies involved ([block 10](../10-threat-modelling-and-design/README.md), the VM prompt), and asset criticality from an owned list. Deterministic, explainable, and the order path lands at the top because the data says so.
- **Encode the business threats explicitly.** A small, curated library of trading-firm-specific abuse cases (runaway strategy, barrier crossing, reconciliation gaming, market-data corruption, exchange-credential misuse) applied as checks, written by people who understand the business. The model matches systems to them; it does not invent them.
- **Treat inputs as hostile.** The documents the system reads are untrusted content: strip and ignore instruction-shaped text, separate the data channel from the instruction channel, log and review anything that tried to change scope, and never let a document mark itself out of scope. Your own agent-security principles applied to your own tool.
- **Keep it inside the firm.** Self-hosted models behind the existing broker, with the architecture corpus never leaving. At Jane Street this is the only acceptable design.
- **Build the evaluation before the tool.** Seed a benchmark: a dozen systems with expert-produced models, plus the last few years of incidents. Measure precision and recall of the generated threats against the expert set, and run the regression question on every incident: *would this have surfaced it?* Track those numbers per release. A tool without that harness is a demo.
- **Close the loop into verification.** Every accepted mitigation produces something checkable: a test, a detection rule, a policy assertion. The threat model stops being a document and becomes a set of assertions the CI can hold the system to ([block 21](../21-detection-engineering/README.md)).
- **Make it diff-aware and continuous.** Run on design and code changes; report only what changed in the threat picture; attach to the review the engineer is already doing. The safe path has to be the fast path or the sandbox lesson repeats.
- **Force a human decision per item, with rationale.** Accept, reject, defer, each with a sentence, each attributed to a named person. This is what the posting means by accountability, and it is what turns machine output back into an owned decision.
- **Pin and version everything.** Prompts, model version, retrieval corpus, rule layer, so a model produced in March is reproducible in September and a change in output can be attributed to a change in input.
- **Teach with it.** The posting wants someone who can transfer threat-modelling knowledge. The best use of a draft model is as a teaching artefact: the system owner reads it, disagrees, and in disagreeing learns to see the exploit. Design the tool for that conversation, not to replace it.
</details>

---

## A worked example: an exchange-connectivity change

<details>
<summary><b>Show me how this would run on a real change.</b> · [analysis] [tie to block 17]</summary>

A desk wants a new strategy connected to a new venue. The change touches the order gateway, adds a session credential, and adds a market-data feed.

- **Extraction** pulls the new gateway service, its RPC endpoints, the credential reference, the feed handler, and the network it sits on, from the repo and IaC. No prose was read to get there.
- **Enumeration** over those elements, grounded in the library, produces a short list: spoofed or replayed FIX sessions on the new link; stale or crossed market data poisoning strategy decisions; the new credential leaking via logs or a shared config; the strategy bypassing pre-trade risk because the new path was wired before the check; the kill switch not covering the new session.
- **Ranking** puts "new path not covered by pre-trade risk and kill switch" first, because the asset list says order path and the graph shows the check is not on the new route. Generic items ("enable TLS") are filtered or ranked low because the control already exists upstream in the graph.
- **Mitigations with verification**: an assertion that every order route passes the risk service (checkable in CI); a detection for FIX sequence anomalies on the new session ([block 21](../21-detection-engineering/README.md)); credential in the vault with the log scanner confirming no plaintext; A/B feed arbitration with a stale-data alert.
- **Review**: the desk's engineer accepts four, rejects one with a sentence ("feed arbitration already handled in the shared handler, see service Z"), and that rejection is itself recorded and feeds the library.
- **Re-run** on the next change to that gateway reports only the delta.

The point to land: the valuable output was one sentence, "the new route misses the risk check", and the machine's job was to make that sentence impossible to miss in the time a human had.
</details>

---

## The interview angle

How to deploy this without sounding like a pitch:

- When asked **how you would scale threat modelling across the firm**: start with the arithmetic (systems versus people), then the pipeline in four lines, then the two biggest failure modes and their fixes. Stop. Let them pull more.
- When asked **what you think of AI in security**: lead with the failure list. Scepticism from someone who has built it is more convincing than enthusiasm.
- When asked **what you have built**: Glasswing, the EMEA AI security programme you run at J.P. Morgan, is where you did this at scale (agentic security, LLM threat modelling, AI-native tooling); the sandboxed multi-agent DAST verifier is the offensive side of the same idea (machine proposes attacks, machine verifies impact, human decides); Threat-modeling.io (Python, FastAPI; STRIDE, ATT&CK and OWASP outputs) is the defensive side. One sentence each, routed to their need, then back to them.
- When they mention **their agent fleet**: connect it. An AI threat-modelling tool is itself an agent reading untrusted documents with access to the firm's architecture; it needs the same sandbox, identity and allow-list discipline they already built ([block 16](../16-ai-and-llm-security/README.md)).
- Never claim it replaces the practice. The posting wants someone who teaches humans to see exploits. The strongest line you have is that the tool's purpose is to give the human hour back to judgement.

[← Threat Modelling role](../22-threat-modelling-role/README.md) · [Index](../README.md) · [Next: Six-month roadmap →](../24-six-month-roadmap/README.md)
