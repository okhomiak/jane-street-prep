# Jane Street Cybersecurity Engineer: interview prep

A complete, source-backed prep repo for the Jane Street Cybersecurity Engineer role in London. Every question in here is one that candidates actually reported being asked, for security roles at Jane Street or at comparable firms, with a link to where it came from. Nothing is invented. Every code snippet has been run and tested.

**How it's organised:** one folder per block of the interview loop. Read them in order the first time. Answers are in collapsible `<details>` blocks so you can test yourself before you read them. Applying in months, not weeks: follow the [six-month roadmap](24-six-month-roadmap/README.md), which closes the gaps properly and builds public proof, and keep the [14-day plan](20-gap-analysis/README.md) for the final sprint. Short on time: start with the [gap analysis](20-gap-analysis/README.md), then self-test with the [drill app](drill/README.md) and the [question bank](QUESTION-BANK.md). Considering the Threat Modelling opening instead: read [block 22](22-threat-modelling-role/README.md) first.

---

## The loop

```
Apply → Online assessment → Phone screen → Live investigation → Coding → Final block → Offer
        (shell + logs,       (culture,       (1 h Zoom,           (1 h, your   (fundamentals, scenarios,
         write a briefing)    your questions)  harder scenario)     language)    design, news, hiring manager)
```

Confirm the exact order with your recruiter. Jane Street says the structure varies by discipline. Full detail and every reported variant: [01 Process](01-process/README.md).

---

## The blocks

| # | Block | What it covers | Rounds it serves |
| --- | --- | --- | --- |
| 00 | [Jane Street intel](00-jane-street-intel/README.md) | The team, how they interview, culture, the AI programme, the Windows signal, OCaml stance; read first | all |
| 01 | [Process](01-process/README.md) | Every reported loop, what Jane Street says it wants, what each round rewards | all |
| 02 | [Log investigation](02-log-investigation/README.md) | The six-step method, a tested command kit, the CEO briefing, verbal drills | OA, live round |
| 03 | [Coding](03-coding/README.md) | The `brute_force_detect` build-up and ten more tasks, all with tested solutions; a secure code-review drill; [know your language: Python internals](03-coding/know-your-language-python.md) | coding round |
| 04 | [Security fundamentals](04-security-fundamentals/README.md) | The five Jane Street questions in full, plus [auth](04-security-fundamentals/auth-and-identity.md), [crypto/TLS](04-security-fundamentals/crypto-and-tls.md), [web/appsec](04-security-fundamentals/web-and-appsec.md), [back-of-envelope maths](04-security-fundamentals/back-of-envelope.md) | fundamentals |
| 05 | [Networking](05-networking/README.md) | TCP, DNS, TLS, BGP, OSI, the "what happens when you type a URL" chain | fundamentals |
| 06 | [Linux and systems](06-linux/README.md) | Two-letter binaries, finding an unauthorised process, hardening, IPC trade-offs | fundamentals |
| 07 | [Windows and endpoints](07-windows-and-endpoints/README.md) | Fresh-install hardening, the event IDs, AD attacks, EDR | fundamentals, scenarios |
| 08 | [Cloud and supply chain](08-cloud-and-supply-chain/README.md) | IAM, S3, SSRF/metadata, containers, CI/CD, the 2026 supply-chain lessons | fundamentals, design |
| 09 | [Incident response](09-incident-response/README.md) | The seven-step scenario skeleton, every reported scenario, the CEO briefing | scenario rounds |
| 10 | [Threat modelling and design](10-threat-modelling-and-design/README.md) | A full module: [frameworks](10-threat-modelling-and-design/frameworks.md) critically, the [45-minute playbook](10-threat-modelling-and-design/live-exercise-playbook.md) with the published rubric, [six worked models](10-threat-modelling-and-design/worked-models.md) including Jane Street's Ontology, the [trading-firm threat model](10-threat-modelling-and-design/hedge-fund-threat-model.md), [every reported prompt](10-threat-modelling-and-design/interview-questions.md), plus the secure-design prompts | design, TM role, HM |
| 11 | [Judgement and hygiene](11-judgement-and-hygiene/README.md) | The 2FA pushback, your personal setup, the trade-off questions | judgement, HM |
| 12 | [Security news](12-security-news/README.md) | Five autumn-2026 stories ranked for a trading firm; refresh before you go | news question |
| 13 | [Behavioural](13-behavioural/README.md) | The 60-second intro, why Jane Street, the story bank, questions to ask | phone screen, HM |
| 14 | [Study plan](14-study-plan/README.md) | A seven-session plan and day-before checklist | none |
| 15 | [Malware, forensics, reversing](15-malware-forensics-reversing/README.md) | Triage funnel, the phishing-to-malware chain, memory and disk forensics, reading a binary | scenario rounds |
| 16 | [AI, LLM and agent security](16-ai-and-llm-security/README.md) | Your differentiator: the lethal trifecta, securing agents, RAG and MCP, and Jane Street's own sandbox design | HM, design, judgement |
| 17 | [Trading domain](17-trading-domain/README.md) | FIX, market data, order entry, pre-trade risk and kill switches, Knight Capital, why latency changes the controls | design, scenarios, HM |
| 18 | [OCaml literacy](18-ocaml-literacy/README.md) | The two-evening minimum: the "why OCaml" answer, concepts to recognise, the ecosystem names, the honest close | HM, culture fit |
| 19 | [Mock interviews](19-mock-interviews/README.md) | Seven timed scripts with grading rubrics, one per round, weighted to coding and live investigation | all |
| 20 | [Gap analysis](20-gap-analysis/README.md) | A critical read of your strengths and risks for this role, a coverage audit, and a 14-day plan | none |
| 21 | [Detection engineering](21-detection-engineering/README.md) | The detection lifecycle, Sigma and ElastAlert, tuning and the base-rate problem, threat hunting, Zeek/Suricata, beacons, email security (SPF/DKIM/DMARC, BEC, phishing analysis) | fundamentals, scenarios, design |
| 22 | [The Threat Modelling role](22-threat-modelling-role/README.md) | Their second security opening, line by line: fit, the CV-screen risk, how its loop differs, and whether to make it your primary target | all (for that role) |
| 23 | [AI-assisted threat modelling](23-ai-threat-modelling/README.md) | How it works as a pipeline, the eleven ways it fails, the engineering that makes it trustworthy, a worked exchange-connectivity example | design, HM |
| 24 | [Six-month roadmap](24-six-month-roadmap/README.md) | Month-by-month plan to close the gaps for real and build the public proof, if you apply in six months | none |
| 25 | [The application](25-the-application/README.md) | What the CV screener pattern-matches, builder-voice bullets before and after, the free-text box drafted, re-applying honestly, referrals, timing | CV screen |

Every reported question in one place to self-test: [QUESTION-BANK.md](QUESTION-BANK.md). Practise with the try-before-reveal [drill app](drill/README.md) (levels, follow-ups, and the wrong answer interviewers hear most). All sources, with a reliability note, are in [SOURCES.md](SOURCES.md).

**Keeping it current over six months.** Before you apply, refresh what dates: the five stories in [block 12](12-security-news/README.md) (the week of each interview); the postings cited in [00](00-jane-street-intel/README.md) and [22](22-threat-modelling-role/README.md) (they are reposted and occasionally reworded; check the requirements lines still match); the NYC salary band; and any OWASP list numbering. Everything else here is mechanism and method and does not go stale.

---

## The things that decide it

1. **They want a programmer who does security.** The team describes itself as "programmers and security experts", programmers first. Lead every answer with what you build.
2. **Coding is the round people fail.** A London candidate was cut on it alone. [Block 03](03-coding/README.md) is the one to over-prepare.
3. **Narrate everything.** The live investigation scores how you think out loud, not your speed.
4. **Measured beats maximal.** "Not everyone should lock down their entire life." Pick controls that fit the person and the risk.
5. **Know what you don't know.** "I'm not sure, here's how I'd find out" is a stated hiring principle, not a weakness.
6. **Drop the bank vocabulary.** No "governance", "framework", "stakeholder alignment". Say what you built and what changed.

---

## How these questions were gathered

Primary sources are Jane Street's own postings and interview pages. Question-level detail comes from anonymous candidate reports (Glassdoor, Blind, Reddit) for Jane Street and for comparable security roles at trading firms and big tech, plus two well-known curated banks ([tadwhitaker](https://github.com/tadwhitaker/Security_Engineer_Interview_Questions), [gracenolan](https://github.com/gracenolan/Notes/blob/master/interview-study-notes-for-security-engineering.md)) and Daniel Miessler's question list. Candidate reports are self-selected and some are years old, so they show the format and flavour, not a guaranteed script. Every block names its sources inline.

Good luck. Be nice, be clear, know your language, know what you don't know.
