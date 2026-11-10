# 13 · Behavioural, fit and the hiring manager

[← Security news](../12-security-news/README.md) · [Index](../README.md) · [Next: Study plan →](../14-study-plan/README.md)

Jane Street says the question behind every round is whether they'd want to work with you, and their four principles are be nice, be clear, know your language, know what you don't know ([Jane Street](https://www.janestreet.com/preparing-for-a-software-engineering-interview/)). The phone screen is largely a culture check ([Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Cybersecurity-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,33.htm)), and a scenario afternoon ends with the hiring manager. Keep answers short, credit other people, and stop when you've made the point. Tags: [JS] Jane Street, [DM] Miessler, [GD] Glassdoor.

---

## The 60-second intro

Read it aloud, time it, make it yours. No bank vocabulary.

> I'm a security engineer who writes code. At J.P. Morgan I lead product security for a new cross-asset trading platform, and the work I'm proudest of is the stuff I built: an automated remediation bot, and a verifier that tests each finding in an isolated sandbox before anyone triages it. Before that I built security teams and tooling at Wise, Alipay and Lockwood, and I started in critical infrastructure security back in Ukraine. On the side I built Threat-modeling.io, a Python and React tool I still maintain. I want to spend more of my time building, on a small team of strong engineers who own their systems end to end.

---

## Why Jane Street, why leave, compensation

<details>
<summary><b>Why Jane Street?</b> · [JS, asked in nearly every loop]</summary>

Three reasons in your own words: engineers build their own tools and own whole problems, which is how you work on Threat-modeling.io and what you miss in a bank; the culture of teaching and admitting what you don't know matches how you think a security team should run; and the problem is interesting, a small, very technical firm where controls have to fit how fast people work. Avoid "prestige" and "compensation" as the lead; they know, and it reads as the wrong motive.
</details>

<details>
<summary><b>Why leave an SVP role at J.P. Morgan?</b> · [JS theme]</summary>

Straight, no bank-bashing: "I'm at my best with my hands on the problem. At my current scale the job drifts toward coordination. I'd rather be an engineer on a team of very strong engineers than manage from a distance." The risk they're probing is whether a senior bank person will actually do hands-on work; answer it before they ask by talking about what you build.
</details>

<details>
<summary><b>Compensation, if it comes up early.</b> (Full scripts for the recruiter call, with pay data and a worksheet: [block 26](../26-the-recruiter-call/README.md).)</summary>

"I'd like to understand the role and level first; I'm confident we'll land somewhere that works." Don't name a number first. Jane Street biases to cash, which helps you.
</details>

<details>
<summary><b>Do you have questions for us?</b> · [JS: asking good questions is a stated value]</summary>

Have five ready; ask the ones that fit the interviewer:
1. What has the team built for itself lately, rather than bought?
2. What does a strong first six months look like in this role?
3. How do you decide when a control is worth the friction it adds?
4. How are you thinking about the security of the AI tools your engineers use?
5. How does incident response work across London, New York and Hong Kong?

Miessler's favourite from the other side of the table: "ask about the worst parts of the job." Asking it shows confidence and gets you an honest answer.
</details>

---

## Story bank

Fill the blanks before the final round. Each story in four beats: the problem, what you built or did, the result with a number, what you'd do differently. Keep each to about 90 seconds.

| Story | Use it for | The beats |
| --- | --- | --- |
| Multi-agent verifier with AI DAST in a sandbox | "Something you built", automation, AI security | Too many false positives to triage by hand → built a sandbox that tries each finding before a human sees it → result: [your number] → what broke first and what you'd change |
| Centralised scanning at Bell (Trivy, TruffleHog, Gitleaks) | Making security the easy path | Teams weren't scanning → zero-config caller workflow, one file to adopt → result: [repos covered] → the adoption lesson |
| Paved roads at J.P. Morgan | Trade-offs, influencing engineers | Governance was slowing delivery → secure-by-default modules and training → 40% fewer post-deployment incidents and 35% faster change approvals, 450+ engineers trained → where it didn't land |
| Growing the Wise team 5 → 13 | Hiring, teaching, "bringing others along" | How you hired, what each person owned, one person you helped grow |
| A time you were wrong | "Know what you don't know" | A real technical call you got wrong → what it cost → what you changed. Not a humble-brag. |
| An incident you ran | Pressure, decisions with partial information | The first hour, the call you made with incomplete data, what you'd do faster now |
| A disagreement with a senior stakeholder | Standing your ground kindly | The decision, how you made the case with evidence, how it resolved, what the relationship was like after |

---

## Behavioural questions reported across security loops

<details>
<summary><b>Tell me about yourself. Why this role? A mistake you made and what you learned.</b> · [SOC, FAANG, JS phone screen]</summary>

Intro above. "Why this role" ties your builder identity to their team. The mistake question is where Jane Street's "know what you don't know" is tested directly: pick something real, own it in one sentence, spend the rest on what changed. Never "I work too hard."
</details>

<details>
<summary><b>Describe a time you handled a high-pressure incident. A time you went above and beyond.</b> · [SOC, FAANG behavioural]</summary>

The incident story from your bank, structured as [block 09](../09-incident-response/README.md) teaches: calm, ordered, decisions with partial information, and what you learned. "Above and beyond" wants ownership beyond your remit; pick a time you fixed the root cause nobody asked you to.
</details>

<details>
<summary><b>How do you stay current? How would your team describe you?</b> · [cloud security loops]</summary>

Staying current: two or three sources you actually read and why ([block 12](../12-security-news/README.md)), plus your independent research track. Team description: pick two true traits with a one-line example each; "calm in incidents" and "the person who writes the tool instead of complaining" fit your profile.
</details>

<details>
<summary><b>Describe a time you defended a technical decision against a senior stakeholder. Why a trading firm over big tech?</b> · [Citadel] [JS]</summary>

The stakeholder story from the bank: evidence, not authority, and a relationship intact afterwards. Trading firm versus big tech: smaller surface, faster feedback, ownership end to end, and you find the security problems more interesting than ad fraud or content moderation. Mean it.
</details>

<details>
<summary><b>What would you do in your first 90 days?</b> · [hiring-manager staple; shape it for a flat, build-first firm]</summary>

The trap for a senior hire is to arrive with a programme. At a firm that puts little weight on seniority and builds everything in-house, the right shape is learn, ship something small, then propose.

- **First 30: learn the system and the people.** Sit the on-call rotation, triage real alerts, read the existing detections and the incident history, map the crown jewels (order path, market data, research, exchange credentials) against what is actually monitored. Ask a lot of questions; "asking great questions matters more than knowing all the answers" is their line. Find out what the team hates doing.
- **By 60: ship one useful thing.** Something small, concrete and theirs: automate the triage step everyone does by hand, add the detection for the gap you found, fix the tool that pages wrongly. Credibility here comes from a merged change, not a strategy document. Pick it with the team, not for them.
- **By 90: propose, with evidence.** Now you have earned a view: here is the biggest gap between what matters and what we can see, here is what I would build, here is what I would stop doing. Written up for engineers, with the trade-offs, and offered for debate rather than announced.

Then say what you would *not* do: rename things, bring the bank's governance vocabulary, or propose a reorg. The posting wants someone who can be methodical with open-ended problems and bring others along; a 90-day plan that is mostly listening and one shipped tool shows both.
</details>

---

## How to show the four principles without reciting them

- **Be nice:** credit colleagues in your stories, talk about bringing people along, don't bad-mouth J.P. Morgan.
- **Be clear:** short answers, define a term before you use it, draw when it helps.
- **Know your language:** in coding and fundamentals, go one layer deeper than asked.
- **Know what you don't know:** say "I'm not sure, here's how I'd find out" the moment you reach your edge, and mean it.

If they become abruptly quiet after an answer, it's usually a note-taking pause (they interview in pairs for exactly this), not disapproval. Keep your composure.

[← Security news](../12-security-news/README.md) · [Index](../README.md) · [Next: Study plan →](../14-study-plan/README.md)
