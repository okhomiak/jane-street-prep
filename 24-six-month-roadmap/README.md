# 24 · The six-month roadmap

[← AI-assisted threat modelling](../23-ai-threat-modelling/README.md) · [Index](../README.md) · [Next: The application →](../25-the-application/README.md)

You are applying in about six months, not two weeks. That changes the job of this repo. The [14-day plan](../20-gap-analysis/README.md) is a sprint for someone who has to perform with what they already know; six months is enough to actually *acquire* the two or three things you are missing, and to build public evidence that answers the hardest screening question before anyone asks it. This block is that plan. It assumes a demanding full-time job: roughly eight to ten focused hours a week, more at weekends if you have them, with a bias of at least 60% doing over reading.

The targets are the same five risks from [block 20](../20-gap-analysis/README.md), reframed as things to become true rather than things to manage:

| Risk in block 20 | In two weeks you would... | In six months you can... |
| --- | --- | --- |
| Reads as a bank manager | Rehearse the builder register | Have a GitHub that *is* the builder register, and a CV rewritten around it |
| Windows and AD depth | Memorise event IDs and attack names | Understand the mechanisms and have written the detections for them |
| Live coding under observation | Do three timed runs | Code daily for months and have shipped two small public tools |
| One-layer-deeper fundamentals | Drill rapid-fire | Have taught yourself the mechanisms by building on them |
| OCaml | Learn the "why" and be honest | Have written a few hundred lines and say so |

---

## The proof you build along the way

Everything below produces artefacts. These are the point, because they solve the CV-screen risk that stopped a Blind commenter's Threat Modelling application ([block 22](../22-threat-modelling-role/README.md)) and they give every behavioural answer a link behind it. By application time you should have, public on GitHub under your own name:

1. **A detections repository.** Twenty to thirty Sigma rules for the phishing-to-malware chain and the AD attack paths described in [block 07](../07-windows-and-endpoints/README.md), each with the log source, a sample true-positive event, and a note on false positives. Tested, versioned, documented. This is detection-as-code and it says "programmer who does security" without a single adjective.
2. **A small OCaml tool.** The failed-login detector from [block 03](../03-coding/README.md), written in OCaml, with tests, and a README that says plainly it was your first OCaml project and what you learned. It turns "I have never shipped OCaml" into "I have shipped a little, here it is, and I understand why you use it."
3. **A written piece on AI-assisted threat modelling for financial firms.** [Block 23](../23-ai-threat-modelling/README.md) is the outline; the published version is your calling card for the Threat Modelling role and the hiring-manager conversation. Critical in tone, failure modes up front.
4. **Your own domain at DMARC reject.** Threat-modeling.io with SPF, DKIM and a DMARC policy at `p=reject`, and the aggregate reports actually read. An afternoon's work that proves the email-security material in [block 21](../21-detection-engineering/README.md) is lived, and a good thirty-second story.
5. **A documented decision.** One architecture-decision record from your current work (sanitised), in the options, trade-off, decision, rationale shape, because the Threat Modelling posting asks for exactly that.

None of this is a lab in this repo; they are your projects, in your own repositories, that this repo prepares you to build.

---

## Month by month

### Month 1: Windows and Active Directory, from the defender's side

The largest content gap, and the one reading alone will not fix. Goal: you understand how each major AD attack path works mechanically, you know exactly what it leaves in the logs, and you can write the detection for it.

- **Understand the mechanisms.** Work through [block 07](../07-windows-and-endpoints/README.md) slowly, then go one source deeper on each: Microsoft's own documentation on Kerberos, NTLM, AD Certificate Services and the Windows security audit events, and the MITRE ATT&CK pages for the credential-access and lateral-movement techniques. For each technique, be able to say in one breath what the attacker needs, what they get, and which event IDs and fields it produces.
- **Practise reading, not attacking.** Use a reputable, purpose-built defensive training course or platform for Windows and AD log analysis (many SOC-analyst tracks include labelled event-log datasets). The skill you are building is: given a log set, find the chain. That is the Jane Street OA and live round, and it is what the detections repo needs.
- **Start the detections repo.** By month end, ten Sigma rules for the AD paths and the phishing chain, each with a sample event and a false-positive note ([block 21](../21-detection-engineering/README.md) has the format).
- **Daily Python.** One task a day from [block 03](../03-coding/README.md), timed, out loud, from a blank file. Fifteen to thirty minutes. Keep a log of your times.

**Done when:** you can narrate the Windows walkthrough in [block 02](../02-log-investigation/windows-walkthrough.md) without notes, and the ten rules are in the repo with tests.

### Month 2: Detection engineering as a shipped project

- **Finish the detections repo** to twenty to thirty rules, add a CI step that validates the Sigma syntax and runs each rule against its sample event, and write the README as if a Jane Street engineer were reading it: what is covered, what is not, how to add a rule.
- **Run a real pipeline on your own data.** Ship your own laptop's logs into a local Elasticsearch, convert a few rules with pySigma, and run them under ElastAlert. Learn the rule types in [block 21](../21-detection-engineering/README.md) by using them on real noise. Note where tuning was needed; that becomes an interview story.
- **Email security, lived.** Put threat-modeling.io at DMARC `p=reject`, read the reports for two weeks, and write down what you learned.
- **Daily Python continues.** Add the "know your language" material in [03](../03-coding/know-your-language-python.md): when you write a generator or a decorator in a task, say why.

**Done when:** the repo is public, CI is green, and you can explain a false positive you actually hit.

### Month 3: OCaml to a working level, and Linux internals

- **OCaml.** The first part of *Real World OCaml*, then the Exercism OCaml track for fluency, then the project: the failed-login detector in OCaml with Dune and a test suite. A few hundred lines. Publish it with an honest README. This is the month where [block 18](../18-ocaml-literacy/README.md) changes from literacy to a shipped artefact, and your "why OCaml" answer gains a sentence of experience.
- **Linux internals.** Re-read [block 06](../06-linux/README.md) and go one layer down on the syscall path, namespaces and seccomp by reading the kernel documentation and a few `strace` sessions of your own tools. Tie it to the firm's agent sandbox ([block 16](../16-ai-and-llm-security/README.md)).
- **Weekly Python**, not daily this month; OCaml takes the daily slot.

**Done when:** the OCaml tool is public with passing tests, and you can explain what a namespace is by pointing at your own `strace` output.

### Month 4: AI and agent security, written down

- **Write the piece.** AI-assisted threat modelling for financial firms, from the outline in [block 23](../23-ai-threat-modelling/README.md). Two to three thousand words, failure modes first, the evaluation harness as the centrepiece. Publish it where your name is on it.
- **Make the offensive instinct current, safely.** Red-team your own toy agent with the standard open-source harnesses (promptfoo, garak): build the attack suite, measure the attack success rate, write the gate. This is your DAST-verifier work in miniature, and it keeps the "offensive background" the Threat Modelling posting requires demonstrably live ([block 16](../16-ai-and-llm-security/README.md), red-team question).
- **Engage publicly, lightly.** Follow the OWASP agentic project; comment or contribute a small item if one fits. Not for the CV line, for the conversations.

**Done when:** the piece is published and the eval harness is a repo you can show.

### Month 5: Trading domain, hardware, and full mock loops

- **Trading depth.** [Block 17](../17-trading-domain/README.md), then one layer deeper: read a FIX specification's session layer, read the SEC's Knight Capital order, and read your own platform's pre-trade risk design with fresh eyes. Write the exchange-connectivity threat model from [block 23](../23-ai-threat-modelling/README.md) by hand, as the ADR you will show.
- **Hardware and firmware.** [Block 07](../07-windows-and-endpoints/README.md)'s hardware section, then Microsoft's documentation on Secure Boot, measured boot and Kernel DMA Protection.
- **Mock loops, weekly.** One full mock from [block 19](../19-mock-interviews/README.md) per week with a peer who will interrupt you; rotate through all nine. Record them. Listen for the management register and strike it.
- **The six worked threat models, cold.** Run each model in [worked-models.md](../10-threat-modelling-and-design/worked-models.md) on the playbook clock without notes, then change one assumption and run it again. Ontology and the venue connection get the most reps.
- **Estimation out loud.** [Back-of-envelope](../04-security-fundamentals/back-of-envelope.md), until the entropy and base-rate sums are reflexive.

**Done when:** a peer says your Windows answers are as sharp as your cloud ones, and your mock scores are flat across rounds.

### Month 6: Position, apply, sprint

- **Rewrite the CV and the application text.** [Block 25](../25-the-application/README.md) has the principles, before-and-after bullets, the three free-text drafts, and how to handle having applied before. Lead with what you built this half-year and before it; the title comes last. No governance vocabulary.
- **Choose the primary role.** [Block 22](../22-threat-modelling-role/README.md) makes the case for Threat Modelling as primary with a line expressing interest in the Cybersecurity Engineer role. Decide, then apply. The postings have stayed open and been reposted across the year, so apply when you are ready rather than waiting for a new listing.
- **Run the 14-day sprint** from [block 20](../20-gap-analysis/README.md) in the final two weeks before the first interview. It will feel easy, which is the point.
- **Refresh [block 12](../12-security-news/README.md)** the week of each interview.

---

## Weekly rhythm that makes this survivable

- **Three short weekday sessions** (45 to 60 minutes): the daily coding slot plus one block's reading.
- **One longer weekend session** (3 to 4 hours): the month's project.
- **One mock a week from month 5**, with a human if at all possible.
- **A fortnightly review**: open [block 20](../20-gap-analysis/README.md), re-score the five risks honestly, and move time toward whichever is still red.

## What to stop doing

Reading more of this repo. After the first pass, the repo is a reference, not a curriculum; the curriculum is the five artefacts above. If a week goes by with reading and no commit, the week was wasted.

## You are ready when

- The five artefacts exist and you would be happy for a Jane Street engineer to read any of them cold.
- You can run the Windows walkthrough, the failed-login build-up in two languages, and a live threat model without notes.
- A mock interviewer cannot tell from your answers that you have ever had a title with "Vice President" in it.
- Your honest answer to "where are you weakest" is a smaller thing than it is today, and you can say what you did about it.

[← AI-assisted threat modelling](../23-ai-threat-modelling/README.md) · [Index](../README.md) · [Next: The application →](../25-the-application/README.md)
