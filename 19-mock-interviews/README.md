# 19 · Mock interviews (timed scripts)

[← OCaml literacy](../18-ocaml-literacy/README.md) · [Index](../README.md) · [Next: Gap analysis →](../20-gap-analysis/README.md)

Reading answers is not practising them. These are timed scripts you can run alone (talk out loud, record yourself) or hand to a friend who reads the prompts and the rubric. Each maps to a real round from [block 01](../01-process/README.md). Grade hardest on the four principles, because that is what they score: **be nice, be clear, know your language, know what you don't know.** Do the coding and live-investigation ones most; those are where candidates fail.

How to grade: for each rubric line, mark hit / partial / miss. Any "miss" on a principle line is the thing to fix before the next run.

---

## Mock 1 · Phone screen (30 min, culture and fit)

**Reader prompts, in order:**
1. Tell me about yourself, in two minutes.
2. Why Jane Street?
3. You're an SVP at J.P. Morgan. Why leave that?
4. Tell me about a security decision you made that someone pushed back on. What did you do?
5. What are you reading in security right now, and what actually matters for a firm like us?
6. What questions do you have for me?

**Rubric:**
- [ ] Intro was 60 to 90 seconds, led with building, not titles. No "SVP" as the headline.
- [ ] "Why Jane Street" named something specific (programmers-first team, in-house culture, the AI programme), not generic prestige.
- [ ] The leaving answer was toward Jane Street, not away from J.P. Morgan; no bitterness.
- [ ] The pushback story held a position and also conceded a real point. Humility plus spine.
- [ ] The news answer ranked relevance for a trading firm; did not firehose.
- [ ] Questions for them were specific and showed homework (asking good questions is a stated value).
- [ ] Warm, clear, concise throughout. Caught any moment of waffle and stopped.

---

## Mock 2 · Live log investigation (45 min, narrated)

**Setup:** use your own OA memory or the sample logs in [block 02](../02-log-investigation/README.md). The reader plays the engineer watching, says little, and notes whether you narrate. Halfway, the reader interjects once: "What's your leading theory right now?"

**Reader prompts:**
1. Here are logs from several workstations. Something happened. Walk me through how you find out what.
2. (Midway) What's your current theory, and what would change your mind?
3. Now summarise what happened for someone non-technical, in four sentences.

**Rubric:**
- [ ] Read a few lines of each log format before grepping (the formats are not all syslog).
- [ ] Established a baseline before hunting anomalies.
- [ ] Narrated continuously: what I'm checking, what I expect, what I found.
- [ ] Pivoted on evidence (an IP, a time window, a process) rather than guessing.
- [ ] Held a theory but named what would disconfirm it (know what you don't know).
- [ ] The non-technical summary was four clean sentences: who, how, what they touched, what to do. No jargon.
- [ ] Stayed calm and orderly under the interjection.

---

## Mock 3 · Coding (45 min, your language, shared screen)

**Setup:** blank file, the reader watches you type, 45-minute timer. Use the Jane Street build-up from [block 03](../03-coding/README.md).

**Reader prompts, revealed one at a time (do not show the next until the current works):**
1. Given a list of log lines like `2026-11-07T09:15:02Z user=alice result=fail ip=10.0.0.5`, write a function that returns the users with 3 or more consecutive failed logins.
2. Now also return, for each, the time window of that run.
3. Now do it per source IP as well as per user.
4. Now only count failures inside a 5-minute sliding window.
5. What's the time complexity, and what breaks at 10 million lines?

**Rubric:**
- [ ] Asked clarifying questions before coding (input shape, what counts as consecutive, sorted or not).
- [ ] Wrote a first version that ran, then grew it in parts, rather than designing the whole thing silently.
- [ ] Wrote or described a test with an edge case (empty, a single failure, exactly 3).
- [ ] Narrated the approach; the reader never had to ask "what are you doing".
- [ ] Handled the extensions without rewriting from scratch (the structure absorbed them).
- [ ] Gave complexity honestly and named the memory problem at scale (stream it, don't load it all).
- [ ] Clean names, no cleverness for its own sake. Readable beats short.

---

## Mock 4 · Fundamentals rapid-fire (30 min)

**Setup:** the reader picks 12 questions at random from [block 04](../04-security-fundamentals/README.md) to [08](../08-cloud-and-supply-chain/README.md), mixing topics. 90 seconds each. The reader follows one answer with "go one layer deeper" to test depth.

**Rubric:**
- [ ] Answered the mechanism, not the buzzword (how, not just the name).
- [ ] Went one layer deeper when pushed, or said cleanly where your knowledge stops.
- [ ] Said "I don't know, here's how I'd find out" at least once without flinching, if it came up.
- [ ] Windows and endpoint answers ([07](../07-windows-and-endpoints/README.md)) were as sharp as the cloud ones. (This is your known gap.)
- [ ] No rambling; each answer was a tight 60 to 90 seconds.

---

## Mock 5 · Scenario and tabletop (30 min)

**Reader prompt:** pick one from [block 09](../09-incident-response/README.md), for example: "An EDR alert fires for credential dumping on a developer's laptop at 2pm on a trading day. Lead the response." The reader plays the room and injects complications: "The developer says they were just testing something." Then: "Legal is asking whether to notify. What do you tell them?"

**Rubric:**
- [ ] Ordered response: verify, contain, preserve evidence, scope, eradicate, recover, learn. Did not jump to reimage.
- [ ] Made decisions on partial information and said what would change them.
- [ ] Weighed business impact (a trading day) against containment, out loud.
- [ ] Handled the "just testing" curveball without either dismissing or over-trusting it.
- [ ] Gave legal a clear, honest status, not a guess dressed as fact.
- [ ] Treated it as blameless; talked about process, not fault.

---

## Mock 6 · Secure design (45 min, whiteboard)

**Reader prompt:** one from [block 10](../10-threat-modelling-and-design/README.md) or [17](../17-trading-domain/README.md), for example: "Design how our firm connects a new strategy to an exchange, securely." The reader asks "what would you skip, and why" near the end.

**Rubric:**
- [ ] Started by naming the assets and the crown jewels (order path, market data, credentials, alpha), not by drawing boxes.
- [ ] Walked the data flow and found where untrusted input enters and where it goes.
- [ ] Named trade-offs explicitly, including latency vs inline inspection (the trading-specific judgement).
- [ ] Proposed pre-trade risk checks and a kill switch unprompted.
- [ ] Answered "what would you skip" with a real, reasoned cut, not "nothing". Measured beats maximal.
- [ ] Brought the person who doesn't see the exploit along, rather than lecturing.

---

## Mock 7 · Hiring manager (30 min)

**Reader prompts:**
1. Walk me through something you built end to end.
2. Where are you weakest for this role, honestly?
3. We write a lot in OCaml. How do you feel about that?
4. Tell me about a time you were wrong.
5. What do you want to be doing in two years?

**Rubric:**
- [ ] The "built" story was concrete, with a result (a number, an outcome), and routed to a Jane Street need.
- [ ] The weakness answer was real (Windows depth, or live OCaml), with a plan, not a humble-brag.
- [ ] The OCaml answer was honest: why they use it, that you've not shipped it, that their pages call that normal.
- [ ] The "wrong" story showed real reflection and what changed, no deflection.
- [ ] The two-year answer tied ambition to the role, not past it (the HR-screen trap from [block 01](../01-process/README.md)).

---

## Running order for the two weeks before

Do them in this order, one or two a day, repeating the ones you scored worst on: Mock 3 (coding), Mock 2 (live investigation), Mock 4 (fundamentals, heavy on Windows), Mock 5 (scenario), Mock 6 (design), Mock 1 (phone), Mock 7 (hiring manager). Coding and live investigation get at least three runs each. Your personal priorities are in [block 20](../20-gap-analysis/README.md).

[← OCaml literacy](../18-ocaml-literacy/README.md) · [Index](../README.md) · [Next: Gap analysis →](../20-gap-analysis/README.md)
