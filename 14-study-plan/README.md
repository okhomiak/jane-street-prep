# 14 · Study plan

[← Behavioural](../13-behavioural/README.md) · [Index](../README.md)

A seven-session plan, about 90 minutes each, built around the way Jane Street actually tests. Your strongest area is depth of knowledge; your real risk is live coding and narrating while you work, since you sat the assessment silently last time and one London candidate was cut on the coding round alone. So coding and talking out loud get the most reps.

Tick items off as you go.

---

## Gaps to close, in order

1. **Live coding speed.** Rebuild the brute-force problem's four parts until 30 minutes feels easy. This is the single highest-value thing.
2. **Talking while you work.** The live investigation scores narration. Practise saying what you expect before every command.
3. **Your own setup.** Easy to know and still ramble. Fill the table in [block 11](../11-judgement-and-hygiene/README.md) and say it in 90 seconds.
4. **Windows and macOS endpoint depth.** Jane Street runs both; have the fresh-install answer and the event IDs cold.
5. **Bank voice.** Any sentence with "governance", "framework" or "stakeholder alignment" gets rewritten as what you built.

---

## The seven sessions

- [ ] **Session 1 · Process and log method.** Read [01](../01-process/README.md) and [02](../02-log-investigation/README.md). Out loud, run the six-step loop on a log set you make yourself (any auth or web log). Write the technical note and the CEO briefing from [block 02](../02-log-investigation/README.md) against a timer.
- [ ] **Session 2 · Coding, part one.** From [03](../03-coding/README.md), rebuild the four-part brute-force problem from a blank file against a 30-minute timer. Then the top-failed-IPs and the IP-as-integer problems. Say the clarifying questions and the complexity out loud each time.
- [ ] **Session 3 · Coding, part two.** The CIDR, firewall, rate-limiter and `cd` problems. Then the secure code review drill: find all nine bugs in five minutes and say each fix. Invent one new variant and solve it cold.
- [ ] **Session 4 · Fundamentals.** [04](../04-security-fundamentals/README.md) and its three files. Answer the five Jane Street questions in full out loud, then the rapid-fire tables. Fill your personal-setup table and defend every line.
- [ ] **Session 5 · Systems and platform.** [05](../05-networking/README.md), [06](../06-linux/README.md), [07](../07-windows-and-endpoints/README.md). Say the Linux recall list and the Windows event IDs from memory. Answer "two-letter binaries", "fresh Windows install", "unauthorised process on Linux", "TCP headers", "how does DNS work".
- [ ] **Session 6 · Scenarios, design, judgement.** [09](../09-incident-response/README.md), [10](../10-threat-modelling-and-design/README.md), [11](../11-judgement-and-hygiene/README.md). Three threat-model prompts on paper, 10 minutes each. Three IR scenarios with the seven-step skeleton. Record yourself on one and listen back for jargon and filler.
- [ ] **Session 7 · News, behavioural, dry run.** [12](../12-security-news/README.md) refreshed to the week's real stories, [13](../13-behavioural/README.md). The 60-second intro, why Jane Street, why leave, each news story in 60 seconds, fill the story-bank numbers. Then three back-to-back mocks and stop. Check your editor, Python setup and Zoom the night before, since Jane Street says to be comfortable with your own tools.

---

## The day before

- Re-read [01 Process](../01-process/README.md) so the loop shape is fresh and you know what each round rewards.
- Refresh [12 News](../12-security-news/README.md) with the current CISA KEV top items and one digest; have one story that matters for a trading firm and one that's noise.
- Confirm your setup: the language you'll code in, your editor, a working Python, Zoom tested, phone charged (they may call).
- Sleep. The coding round rewards a clear head more than one more hour of cramming.

## On the day

- Ask the clarifying question before you type or grep. It reads as rigour.
- Narrate. Say what you expect, then whether it happened. Silence looks like being lost.
- When you hit your edge: "I'm not sure, here's how I'd find out." It's a hiring principle, not a weakness.
- Lead every answer with the point, then the trade-off, then what you'd do at Jane Street.
- Be nice, be clear, know your language, know what you don't know.

---

## One-line reminders per block

| Block | The one thing |
| --- | --- |
| [01 Process](../01-process/README.md) | Confirm the round order with your recruiter; you're past the OA. |
| [02 Log investigation](../02-log-investigation/README.md) | Baseline before you hunt; end with what you'd do next. |
| [03 Coding](../03-coding/README.md) | Ask five questions, grow it in parts, test out loud. |
| [04 Fundamentals](../04-security-fundamentals/README.md) | Mechanism, attack, trade-off. One layer deeper than asked. |
| [05 Networking](../05-networking/README.md) | Ping has no port; name a control at each hop. |
| [06 Linux](../06-linux/README.md) | `cd` is a builtin; cross-check `/proc` against `ps`. |
| [07 Windows](../07-windows-and-endpoints/README.md) | 4624 type 3 + NTLM where Kerberos is normal = pass-the-hash. |
| [08 Cloud](../08-cloud-and-supply-chain/README.md) | Identity is the perimeter; pin and mirror dependencies. |
| [09 IR](../09-incident-response/README.md) | Confirm, scope, contain, preserve, recover, learn, communicate. |
| [10 Threat modelling](../10-threat-modelling-and-design/README.md) | Draw first, name the crown jewels, say what you'd skip. |
| [11 Judgement](../11-judgement-and-hygiene/README.md) | Measured beats maximal; fit the control to the person. |
| [12 News](../12-security-news/README.md) | Rank for a trading firm; know the noise. |
| [13 Behavioural](../13-behavioural/README.md) | You're a builder who writes code; short answers, credit others. |

[← Behavioural](../13-behavioural/README.md) · [Index](../README.md)
