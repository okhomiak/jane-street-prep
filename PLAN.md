# Build plan

Target: make Oleh the best-prepared candidate in the Jane Street London Cybersecurity Engineer loop. Every stage ends in a commit. One research or verification agent at a time, each claim checked against its source before it goes in.

## Where things stand (7 Nov 2026)

- **Complete:** all ten stages built and committed. The repo is now 21 blocks plus a master question bank, a gap analysis and a 14-day plan. Every reported question is merged from the research and tagged with its source; every code snippet is tested; two independent verification agents fact-checked the intel, process, forensics and AI/trading blocks against their source URLs and their corrections are applied. The online assessment was passed in May (ACME Trading Corp shell scenario).
- **Research now merged:** the five research reports (~22k words, ~370 source links) that seeded the blocks are kept under `/home/claude/research/` as the provenance trail; their content is in the blocks.
- **Blocked:** pushing to github.com/okhomiak/jane-street-prep. The session can't reach GitHub until the account is linked in claude.ai Settings → Connectors. Commits are local until then; a zip is delivered. Once linked: `git branch -M main && git remote add origin https://github.com/okhomiak/jane-street-prep.git && git push -u origin main`.

## Method for every stage

1. Draft from the research on disk plus trained knowledge.
2. One agent verifies every factual claim against its URL and hunts for what's missing.
3. Fix, run the code, check links and style (no em dashes, contractions, sources inline, `<details>` answers).
4. Commit with a clear message. Deliver the zip. Push if GitHub is linked.

## Stages

| Stage | Output | Acceptance |
| --- | --- | --- |
| 0 | This plan committed | Plan in repo |
| 1 | `00-jane-street-intel/`: everything knowable about Jane Street's security team, loop, culture, how they interview, the Windows-plus-automation signal, in-house tooling, OCaml stance; `01-process` rewritten with every verified report | Every report has a URL; claims verified by a second agent |
| 2 | `03-coding` expanded: 20+ tested tasks incl. the exact reported Jane Street family (consecutive failures → all users → time window → alert decision), sessionisation, impossible travel, multi-file correlation, bash one-liners for the "script a few things" phone round, a timed drill sheet, interviewer push list | `pytest` green on every snippet extracted from the README |
| 3 | `02-log-investigation` expanded: Windows-centric walkthroughs (the OA was Windows workstation logs), Sysmon and event-ID correlation, wiki and file-share logs, a full narrated transcript, Linux log review as reported in HK | Commands run against sample logs in the stated formats |
| 4 | `04` fundamentals: every reported auth, crypto, web, network question from all five research files merged in with answers; rapid-fire tables doubled | Each question tagged with its source |
| 5 | `06`, `07` + new `15-malware-forensics-reversing`: Linux internals, Windows/AD, memory and disk forensics, malware triage, reversing basics as reported | Source per question |
| 6 | `08` + new `16-ai-and-llm-security`: cloud, supply chain, prompt injection, agent tool-use, MCP, model supply chain, securing an internal AI assistant; Oleh's sandboxed multi-agent DAST verifier and Threat-modeling.io positioning | Source per question; frameworks named (OWASP LLM Top 10, Agentic Top 10, MITRE ATLAS) |
| 7 | `09`, `10` + new `17-trading-domain`: market data, order entry, FIX, exchange connectivity, colocation, latency vs security, crown jewels of a trading firm, insider risk, strategy IP | Reviewed by a verifier for accuracy |
| 8 | `11`, `12` (refreshed), `13` + new `18-ocaml-literacy`: judgement, news, behavioural with all reported questions, Jane Street culture questions, OCaml minimum | News dated and sourced |
| 9 | `00-question-bank/`: one master table of every question in the repo with source and block link; `19-mock-interviews`: timed scripts with grading rubrics; `20-gap-analysis`: honest assessment of Oleh's weak spots against the role and a 14-day plan | Count of questions stated; every row links to its answer |
| 10 | Final pass: consistency, links, style, SOURCES.md, root README; tag v2 | Link checker clean, tests green |

## Rules that hold throughout

- No lab (log generator) anywhere in the repo.
- No invented questions. A question without a source is marked as practice, not as reported.
- No em dashes. Contractions. Builder verbs. Concrete over abstract. No governance padding.
- Critical, not flattering: where Oleh is weak against this role, say so and give the fix.
