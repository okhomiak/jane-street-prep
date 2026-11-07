# Jane Street Cybersecurity Engineer: interview prep

A complete, source-backed prep repo for the Jane Street Cybersecurity Engineer role in London. Every question in here is one that candidates actually reported being asked, for security roles at Jane Street or at comparable firms, with a link to where it came from. Nothing is invented. Every code snippet has been run and tested.

**How it's organised:** one folder per block of the interview loop. Read them in order the first time, then use the [study plan](14-study-plan/README.md) to drill. Answers are in collapsible `<details>` blocks so you can test yourself before you read them.

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
| 01 | [Process](01-process/README.md) | Every reported loop, what Jane Street says it wants, what each round rewards | all |
| 02 | [Log investigation](02-log-investigation/README.md) | The six-step method, a tested command kit, the CEO briefing, verbal drills | OA, live round |
| 03 | [Coding](03-coding/README.md) | The `brute_force_detect` build-up and ten more tasks, all with tested solutions; a secure code-review drill | coding round |
| 04 | [Security fundamentals](04-security-fundamentals/README.md) | The five Jane Street questions in full, plus [auth](04-security-fundamentals/auth-and-identity.md), [crypto/TLS](04-security-fundamentals/crypto-and-tls.md), [web/appsec](04-security-fundamentals/web-and-appsec.md) | fundamentals |
| 05 | [Networking](05-networking/README.md) | TCP, DNS, TLS, BGP, OSI, the "what happens when you type a URL" chain | fundamentals |
| 06 | [Linux and systems](06-linux/README.md) | Two-letter binaries, finding an unauthorised process, hardening, IPC trade-offs | fundamentals |
| 07 | [Windows and endpoints](07-windows-and-endpoints/README.md) | Fresh-install hardening, the event IDs, AD attacks, EDR | fundamentals, scenarios |
| 08 | [Cloud and supply chain](08-cloud-and-supply-chain/README.md) | IAM, S3, SSRF/metadata, containers, CI/CD, the 2026 supply-chain lessons | fundamentals, design |
| 09 | [Incident response](09-incident-response/README.md) | The seven-step scenario skeleton, every reported scenario, the CEO briefing | scenario rounds |
| 10 | [Threat modelling and design](10-threat-modelling-and-design/README.md) | The 10-minute method, worked prompts, design principles | design round |
| 11 | [Judgement and hygiene](11-judgement-and-hygiene/README.md) | The 2FA pushback, your personal setup, the trade-off questions | judgement, HM |
| 12 | [Security news](12-security-news/README.md) | Five autumn-2026 stories ranked for a trading firm; refresh before you go | news question |
| 13 | [Behavioural](13-behavioural/README.md) | The 60-second intro, why Jane Street, the story bank, questions to ask | phone screen, HM |
| 14 | [Study plan](14-study-plan/README.md) | A seven-session plan and day-before checklist | none |
| 15 | [Malware, forensics, reversing](15-malware-forensics-reversing/README.md) | Triage funnel, the phishing-to-malware chain, memory and disk forensics, reading a binary | scenario rounds |

All sources, with a reliability note, are in [SOURCES.md](SOURCES.md).

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
