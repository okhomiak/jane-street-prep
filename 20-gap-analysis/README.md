# 20 · Gap analysis and the 14-day plan

[← Mock interviews](../19-mock-interviews/README.md) · [Index](../README.md) · [Next: Detection engineering →](../21-detection-engineering/README.md)

An honest, unflattering read of where you stand against this specific role, and what to do with the time you have. The point of this block is not reassurance. It is to spend your preparation on the three or four things that actually decide the outcome, and not on the things you are already good at.

---

## What you have that fits (use it, don't over-practise it)

These are real strengths for *this* role. You do not need to drill them; you need to make sure every answer routes back to them.

- **You build.** The automated remediation bot, the sandboxed multi-agent DAST verifier, the zero-config GitHub Actions scanning pipeline (Trivy, TruffleHog, Gitleaks). Jane Street's cybersecurity team describes itself as "programmers and security experts", programmers first. This is the exact profile, and most candidates cannot say it.
- **You do product security for a greenfield cross-asset trading platform.** The trading-domain round ([block 17](../17-trading-domain/README.md)) is your home turf, not a stretch. Few security candidates can talk FIX, pre-trade risk and market-hours trade-offs from lived experience.
- **Agentic and LLM security is your specialty.** Jane Street runs one of the most advanced internal agent programmes anywhere, and you can meet it as a peer ([block 16](../16-ai-and-llm-security/README.md)). This is your single biggest differentiator.
- **You teach and scale teams.** 450+ engineers trained, a team grown 5 to 13, 40% fewer post-deployment incidents, 35% faster change approvals. The posting explicitly wants "bringing others along". Have these as numbers, not adjectives.
- **You have already passed their online assessment** (May 2026). The hardest filter is behind you.

---

## The risks that actually decide it (spend your time here)

Ranked. Be honest with yourself on each.

### 1. You can read as a senior bank manager. This is the biggest one.

Jane Street's culture is flat and programmer-first; they put "little weight on seniority" in writing. Words that cost you: **"SVP", "governance", "stakeholder alignment", "framework", "oversight", "programme".** They are the native language of a J.P. Morgan SVP and they are exactly wrong here. The failure mode is subtle: you answer a technical question with a management answer, and they conclude you have stopped building. **Fix:** in every mock, catch the management register and restate in builder's language. "I drove governance across stakeholders" becomes "I wrote the bot that did X, and taught the team to use it." Lead with the artifact and the mechanism, every time.

### 2. Windows, Active Directory and endpoint depth.

The role leans Windows, and that is now backed by five of their own postings, not just a rumour ([block 00](../00-jane-street-intel/README.md)). Your recent years lean cloud and AppSec. This is a genuine content gap, not a framing one. **Fix:** [block 07](../07-windows-and-endpoints/README.md) is mandatory, to the point where the event IDs, pass-the-hash, Kerberoasting, the fresh-install hardening answer, and AD CS/Entra attack paths are automatic. In the fundamentals mock, your Windows answers must be as sharp as your cloud ones. If one round exposes you, it is this one.

### 3. Live coding with someone watching.

Coding is the round Jane Street candidates fail; the 2018 London candidate was cut on it alone, "the coding wasn't up to scratch". You can code, but coding *while narrating, while watched, growing one problem in parts* is a different skill, and it decays without reps. **Fix:** Mock 3 in [block 19](../19-mock-interviews/README.md), at least three full runs on a 45-minute timer, from a blank file, out loud. The failed-login build-up in [block 03](../03-coding/README.md) until it is muscle memory.

### 4. One-layer-deeper fundamentals.

A security architect operates at design level all day; this loop will push below it, to TCP header fields, the syscall path, how a password manager keys its vault, the TLS handshake step by step. You know these; the risk is you answer at management altitude when they want the mechanism. **Fix:** the rapid-fire mock (Mock 4), and when in doubt go one layer lower than feels natural.

### 5. OCaml, and the humility that surrounds it.

Zero shipped OCaml. This is fine if you handle it as [block 18](../18-ocaml-literacy/README.md) says: the "why", the names, and the honest close. It becomes a problem only if you fake fluency, which breaks "know what you don't know", the principle they test hardest. **Fix:** two evenings on block 18, and rehearse the honest close so it comes out calm, not apologetic.

---

## Coverage audit: what a critical pass found missing, and what is deliberately left out

A repo built only from reported questions has a blind spot: the same interviewers can go anywhere adjacent. A second pass compared the postings' own language and the adjacent loops against the blocks, and added what the job describes but candidates had not reported in detail:

| Gap found | Why it mattered | Where it now lives |
| --- | --- | --- |
| Detection engineering as a discipline (lifecycle, Sigma, ElastAlert rule types, tuning, hunting, pyramid of pain, Zeek/Suricata, beacon and DNS-tunnel analysis) | Their ops posting names Elasticsearch, ElastAlert, SIEM, EDR, NSM; the D&R posting names threat hunts | [block 21](../21-detection-engineering/README.md) |
| Email security mechanics (SPF/DKIM/DMARC alignment, BEC, header analysis) | Named threats in their posting; one of the most repeated SOC tasks; repo had a one-line checklist | [block 21](../21-detection-engineering/README.md) |
| Python internals (GIL, generators, decorators, context managers, dict hashing, unsafe APIs) | "Know your language" is a scored principle; HRT and Meta asked these; repo listed but never answered them | [03 know-your-language](../03-coding/know-your-language-python.md) |
| Back-of-envelope estimation (entropy and crack time, birthday bound, base-rate fallacy, SIEM sizing) | The password-complexity question *is* estimation; estimation is their culture; the base-rate fallacy underpins every detection answer | [04 back-of-envelope](../04-security-fundamentals/back-of-envelope.md) |
| Design a firm-wide permissioning system, including information barriers | Their named security engineer built exactly this | [block 10](../10-threat-modelling-and-design/README.md) |
| Design a vulnerability-management programme | Asked directly at Optiver; Jane Street has a posting that reads like the answer key | [block 10](../10-threat-modelling-and-design/README.md) |
| Design security automation (what to automate, what never) | "Security automation required"; your remediation bot is the proof point | [block 10](../10-threat-modelling-and-design/README.md) |
| Hardware and firmware (Secure Boot vs measured boot, TPM, DMA, BadUSB, MTE, NUMA) | "From the hardware up"; Jump's CPU question; Apple's silicon question | [block 07](../07-windows-and-endpoints/README.md) |
| Length-extension attack and timing side channels | Classic follow-ups to HMAC and to "compare the token" | [04 crypto](../04-security-fundamentals/crypto-and-tls.md) |
| `kill -15` vs `-9`, pipe vs redirect | Asked at HRT and in a Jane Street DevOps loop; listed but unanswered | [block 06](../06-linux/README.md) |
| "Your first 90 days" | Hiring-manager staple; the wrong shape (arrive with a programme) is exactly your risk 1 | [block 13](../13-behavioural/README.md) |

Deliberately out of scope, and why: writing OCaml (their pages say use the language you know; literacy only, [block 18](../18-ocaml-literacy/README.md)); a hands-on log-generator lab (you asked for none); deep reverse engineering and exploit development (not this role; analyst-level only in [block 15](../15-malware-forensics-reversing/README.md)); mobile application security (no signal in any posting or report); and compliance and audit frameworks as topics (the firm's culture and your own style both reject that register; technical controls are covered where the regulation forces them, like pre-trade risk in [block 17](../17-trading-domain/README.md)). If an interviewer goes to one of these, the right answer is the honest one: "not my depth, here is how I would approach it."

---

## The 14-day plan

Weighted to the five risks above, not evenly across the blocks. Adjust to your calendar; keep the coding and Windows load.

**Days 1 to 3, foundations and the biggest gap.**
- Day 1: [Block 00](../00-jane-street-intel/README.md) and [01](../01-process/README.md) end to end. Then [block 07](../07-windows-and-endpoints/README.md) (Windows), first pass. Start a running list of your "built" stories with numbers.
- Day 2: [Block 03](../03-coding/README.md) coding, first full run of Mock 3, then [know-your-language](../03-coding/know-your-language-python.md) so the follow-ups on your own code do not catch you. [Block 07](../07-windows-and-endpoints/README.md) again, until the event IDs and AD attacks are automatic.
- Day 3: [Block 02](../02-log-investigation/README.md) log investigation, Mock 2. [Block 04](../04-security-fundamentals/README.md) auth and crypto, plus [back-of-envelope](../04-security-fundamentals/back-of-envelope.md) until the entropy and base-rate sums are out loud and fast.

**Days 4 to 7, breadth with the register fix.**
- Day 4: [Block 05](../05-networking/README.md) and [06](../06-linux/README.md). Rapid-fire Mock 4, and listen for the management register; restate anything that slips.
- Day 5: [Block 08](../08-cloud-and-supply-chain/README.md) (fast, it's your strength) then [block 16](../16-ai-and-llm-security/README.md) (your differentiator) in depth. Rehearse describing their sandbox back to them.
- Day 6: [Block 09](../09-incident-response/README.md) scenarios, Mock 5. [Block 21](../21-detection-engineering/README.md) detection engineering and email security: the ElastAlert rule types and the phishing-header walk-through, because the postings describe this job. [Block 17](../17-trading-domain/README.md) trading domain (your home turf; make sure you can tell the Knight Capital story cleanly).
- Day 7: [Block 10](../10-threat-modelling-and-design/README.md) design, Mock 6, including the permissioning-system and vulnerability-management prompts. Second full coding run (Mock 3).

**Days 8 to 11, depth and the soft rounds.**
- Day 8: [Block 11](../11-judgement-and-hygiene/README.md) judgement and the 2FA pushback. [Block 18](../18-ocaml-literacy/README.md) OCaml, evening one.
- Day 9: [Block 13](../13-behavioural/README.md) behavioural, Mock 1 and Mock 7. Tighten the 60-second intro until it never mentions "SVP" first. [Block 18](../18-ocaml-literacy/README.md) evening two.
- Day 10: Third full coding run (Mock 3). [Block 07](../07-windows-and-endpoints/README.md) once more, the weakest topic from Mock 4.
- Day 11: [Block 15](../15-malware-forensics-reversing/README.md) forensics, and whichever two fundamentals topics you scored worst on.

**Days 12 to 14, polish.**
- Day 12: Full loop simulation: Mocks 1, 2, 3 back to back with a friend if you can.
- Day 13: Mocks 5, 6, 7. Refresh [block 12](../12-security-news/README.md) with that week's top stories, re-ranked for a trading firm.
- Day 14 (or the day before): light. Re-read the [question bank](../QUESTION-BANK.md) Jane Street confirmed set, the day-before checklist in [block 14](../14-study-plan/README.md), and stop early. Confidence is a control too.

---

## The one-line reminders to carry in

- Lead with what you built, never with your title.
- Go one layer deeper than the question asked.
- "I don't know, here's how I'd find out" is a winning answer here, not a losing one.
- Measured beats maximal: name the trade-off and what you'd skip.
- Narrate everything in the coding and investigation rounds.
- Be nice, be clear, know your language, know what you don't know.

[← Mock interviews](../19-mock-interviews/README.md) · [Index](../README.md) · [Next: Detection engineering →](../21-detection-engineering/README.md)
