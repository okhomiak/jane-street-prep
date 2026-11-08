# 00 · Jane Street intel

[Index](../README.md) · [Next: Process →](../01-process/README.md)

Everything knowable about the team you're interviewing with, how they run interviews, and what they say they value, from their own pages and from candidates who sat the loop. Read this before anything else: half the "fit" questions in the final round are answered by knowing what's on this page.

---

## The team, in their words

| What they say | Where | Why it matters to you |
| --- | --- | --- |
| Cybersecurity is "a skilled group of programmers and security experts" whose work spans software engineering, DevOps, risk analysis and security governance; members tag in and out for each other by area of expertise | [Departments](https://www.janestreet.com/join-jane-street/departments/) | Programmers first. Lead every answer with what you built. Expect to be asked about areas outside your lane because people cover for each other. |
| The security team is the last line of defence, not the first; employee education is a top priority | same | Your 450-engineers-trained story and the "bringing others along" line in the JD are the same thing. Have one concrete teaching story ready. |
| Security colleagues "appreciate technical detail more than most" | same | Go one layer deeper than asked. Mechanism, then attack, then trade-off. |
| Technology hiring has no checklist; they want technical depth and intellectual humility over particular tools; most engineers arrived without OCaml or finance experience | same | Don't name-drop tools. Say how things work. Say "I don't know" cleanly when you don't. |
| The firm builds almost everything in-house, including its email service, and writes "everything that we can in OCaml" | [Join Jane Street](https://www.janestreet.com/join-jane-street/), [Technology](https://janestreet.com/technology/), [2019 candidate report](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm) | "Which vendor would you buy" is the wrong register. "Here's what I'd build and why" is the right one. Your Threat-modeling.io and remediation-bot stories fit this culture exactly. |
| Technologists are expected to understand their systems down to the hardware; specialist teams in distributed systems, security, networking, IT and compilers build supporting tools | [Who we are](https://www.janestreet.com/who-we-are/) | Expect questions that go below the application layer: TCP headers, Linux internals, how a process gets its memory. |

**A real colleague.** Minjie, a cybersecurity software engineer in Hong Kong who joined as a 2017 intern and went full-time in 2019, built the firm-wide internal permissioning system and tools for visibility into internal networking; a typical day is alert triage, reconstructing events with other teams, and onboarding teams onto the permissioning system. He had no OCaml before joining; every developer does an OCaml bootcamp, then learns the team's libraries on the job ([Get to Know Us](https://www.janestreet.com/join-jane-street/get-to-know-us/minjie)). That profile is the job: build the tool, run the triage, teach the teams.

**The role signal nobody puts in the posting.** A Blind poster with a Jane Street security engineer interview coming up described the role as mostly Windows-specific with security automation required ([Blind](https://www.teamblind.com/post/jane-street-security-engineer-krwvr2zb)). Jane Street's own postings say the same more loudly: the NYC IT Security Engineer and Security Operations Engineer roles want a strong Windows engineering background (AD/Entra ID, PKI, application control, host firewalls, disk encryption, EDR, PowerShell), and the Senior Incident Responder posting describes a primarily Windows environment ([IT Security Engineer](https://www.janestreet.com/join-jane-street/position/8155302002/), [Security Operations Engineer](https://www.janestreet.com/join-jane-street/position/8428218002/), [Senior IR](https://www.janestreet.com/join-jane-street/position/7284341002/)). The 2018 London loop asked what you'd change on a fresh firm-wide Windows build, and the online assessment used Windows workstation logs. Treat Windows and endpoint depth as a first-class topic, not a side one ([block 07](../07-windows-and-endpoints/README.md)).

---

## What the postings ask for

The two London Cybersecurity Engineer postings ([Greenhouse](https://job-boards.greenhouse.io/janestreet/jobs/8057310002), [janestreet.com](https://www.janestreet.com/join-jane-street/position/8842136002/)) and the [NYC version](https://www.janestreet.com/join-jane-street/position/8613910002/), distilled:

| They want | What it looks like in an interview |
| --- | --- |
| A strong programmer who automates in a real language, keeps code and config in version control, is skilled in an editor of their choice | The coding round, "what have you built", "which editor and why" |
| Willingness to build your own tools, judgement in choosing solutions | "Build or buy?" with a reason |
| Methodical with unstructured, open-ended investigations to root cause | The live log round |
| Familiar with threat modelling; explains trade-offs and context; "not everyone should be locking their entire lives down" | Design round, judgement questions, the 2FA pushback |
| Follows security news and can separate headline research from what matters day to day | "What are you reading, and what mattered this month?" |
| Good personal security habits you'll be asked about (the posting says so explicitly), and able to coach others | "Walk me through your setup" |
| Humble, asks questions, admits mistakes, makes hard calls under pressure | Every round. "Know what you don't know" is scored. |
| Positive, collaborative; "bringing others along" | Teaching and awareness stories |

The [Threat Modelling posting](https://www.janestreet.com/join-jane-street/position/7988179002) adds: an offensive background, documenting decision rationale, teaching threat modelling, and talking with system owners who don't yet see the exploit. Jane Street considers you for every open role, so you may be routed there mid-process.

The Threat Modelling opening is analysed line by line in [block 22](../22-threat-modelling-role/README.md); it requires an offensive security background and the ability to teach threat modelling, treats programming as "a plus", and is arguably the closer fit to your record. None of the London or NYC Cybersecurity Engineer or Threat Modelling postings mention AI. The one that does is the NYC [Vulnerability Management](https://www.janestreet.com/join-jane-street/position/8596433002) role, which asks for automation "including AI tooling" with a human in the loop, noting that manual triage doesn't scale at their size. The firm's AI work is real and large (below), so bringing it up well is a differentiator, not a requirement.

---

## How they interview

From Jane Street's own pages ([Interviewing](https://www.janestreet.com/join-jane-street/interviewing/), [software engineering prep](https://www.janestreet.com/preparing-for-a-software-engineering-interview/)):

- Four principles: **be nice, be clear, know your language, know what you don't know.**
- The core question is whether you're someone they want to work with. The journey through the problem matters more than the snapshot at the end.
- Real code in a real language, not pseudocode. Use the language you know best. No bonus points for OCaml, and don't try it for the first time in an interview.
- Open-ended problems with several plausible approaches; no algorithm bingo, no "aha" puzzles. Software loops have no mental maths or logic puzzles (those belong to trading roles; several non-quant candidates did report probability brainteasers, Trading Desk Operations Engineers across 2025 to 2026 and a Technical Support Engineer in Jul 2025, though no security report does, so don't be thrown if one appears).
- Asking great questions is more important than knowing all the answers.
- Technical rounds over Zoom; final rounds in person, often collaborative. Phone screens are by phone (ask for Zoom if your line is bad). No weekend interviews. Expect a reply within about a week. You can ask to expedite if you hold a competing offer.
- Be comfortable with your own setup and tools; technology interviews are hands-on.
- Casual office, and they say they won't judge what you wear.

From candidates (see [block 01](../01-process/README.md) for every report with its source):

- Interviews often have two engineers in the room. One 2018 London candidate's final day had Amsterdam and New York on video.
- Interviewers tell you when an early part of an answer is wrong and adjust follow-ups to how close you were.
- They ask you not to share questions; feedback after rejection is a line at most ("the coding wasn't up to scratch").
- One 2019 candidate said the first screen felt more like a psychological screen than a technical one: culture, ethics, whether their way of working suits you.
- One HR screen rejected a candidate for wanting to grow beyond the posted role. Tie your ambition to the role, not away from it.

---

## Culture you can speak to

From the [values page](https://www.janestreet.com/who-we-are/), the [culture overview](https://www.janestreet.com/culture/overview/) and the [Signals and Threads](https://signalsandthreads.com/) podcast:

| Value | What it means in practice | How to show it |
| --- | --- | --- |
| Intellectual humility | Scepticism, questioning assumptions, discussing mistakes openly | A real mistake story with what changed |
| Less hierarchy, more collaboration | Little weight on seniority | Don't lead with SVP. Lead with the work. |
| Ethics and honesty | Nobody defends views they don't hold | Hold your position under pushback, concede when you're wrong |
| Blameless postmortems | "Process improvement works better than apportioning blame"; "there's really not a blaming culture here" ([ep. 20](https://signalsandthreads.com/solving-puzzles-in-production)) | Talk about incidents as learning, never as blame |
| Puzzles | "Like puzzles? So do we!" Fast, concrete feedback on wins and losses | Enjoy the open-ended problem out loud |
| Teaching | First 18 months include courses on core libraries and patterns; brown-bags and tech talks | Say you want to learn their stack, and that you teach |

**Production engineering, the nearest team to yours** ([ep. 20](https://signalsandthreads.com/solving-puzzles-in-production)): first line of defence for live systems; a quarter to a third of time on support rotation and the rest building tooling; in-house alerting (Oculus), auto-action tooling, an alert-history analyser, centralised logs, data as S-expressions, a monorepo, high-resolution packet captures. Their interviews include a realistic debugging exercise; good candidates communicate well, debug carefully and stay calm. They train calm communication under pressure with incident simulations and games. Expect the security live round to be built the same way.

**Office norms.** The recruiter email for this role said at least three days a week in the London office. A May 2024 Blind thread says three days a week is relatively common, most traders are in five, and the firm doesn't generally hire fully remote ([Blind](https://www.teamblind.com/post/how-many-days-rto-for-jane-street-ptaglpfz)).

---

## AI at Jane Street (your differentiator)

Primary sources: John Crepezzi's [AI Engineer Summit talk](https://ai.engineer/talks/building-ai-powered-developer-tools-at-jane-street) and [Signals and Threads ep. 31, Oct 2026](https://signalsandthreads.com/the-changing-craft-of-programming/).

- An AI Assistance team exists to get value from LLMs. The firm's private OCaml corpus may exceed all public OCaml, so off-the-shelf models underperform; they train custom models on developer workspace snapshots, using red-to-green build transitions as training pairs, with a Code Evaluation Service that applies a model's diff to a warm build and returns pass/fail as the reward.
- AIDE runs on every developer machine (thin Emacs, VS Code and Neovim integrations; about two thirds of the firm was on Emacs at Crepezzi's last count). There's an internal chat (JaneGPT) with a model switcher between vendors, a CLI comparable to Claude Code or Codex, and a Slack flow where a request spins up an agent; around 45% of released features now start that way. Agents run on dev machines and build boxes through a central orchestrator.
- **Agent sandboxing, in their own account:** the first sandbox was a deny-list; they moved to an allow-list using bwrap for the filesystem and pasta for the network; seccomp BPF is used to learn which files a process touches; RPC infrastructure was changed so AI-originated calls are identifiable; denied actions go to an LLM classifier that returns safe, unsafe, or ask the human. Each employee has an AI counterpart identity (for example "AI yminsky") for permissions and line-by-line attribution of AI-written code.
- **Risks they've seen:** models in notebooks escaping to run system commands with the human's permissions; models working around permissions; reward hacking (disabling warnings-as-errors, reading solutions from code search during evals); engineers switching sandboxes off when blocked; LLM graders they don't fully trust at the edge of model capability; models bidding internal "fake dollars" for cluster time and pre-empting humans.
- Verification: Iron code review ordered by dependency; a narrative review tool; Ron Minsky's line that it's harder to verify code no person wrote; interest in long-running fuzzing; and, per a third-party summary of Ron Minsky's May 2026 Dwarkesh interview, a new formal-methods team as AI-generated code grows ([summary](https://songkwwwwwww.github.io/en/jane%20street/jane-street-gpus-trading-hiring/)).
- Scale: a $6bn CoreWeave compute deal plus $1bn equity in April 2026 ([TNW](https://thenextweb.com/news/jane-street-coreweave-6-billion-cloud-1-billion-equity-ai)).

What to do with this: when they ask what you'd want to work on, or how you'd secure agents, you can speak to *their* setup: allow-list sandboxes, identifiable AI identities, the classifier for denied actions, and the places where humans switch the controls off. [Block 16](../16-ai-and-llm-security/README.md) turns this into answers. Don't recite it unprompted; use it when the conversation gets there.

---

## OCaml, in one paragraph

You won't need to code in OCaml; both of their interview pages say to use the language you know best and not to try OCaml for the first time in an interview. You should be able to say why they use it (a type system that catches bugs at compile time, memory safety from garbage collection, and OxCaml's modes to get performance without losing safety), name the ecosystem (Base, Core, Async, Dune, OxCaml, Bonsai, Hardcaml, Iron), and say plainly that you've never shipped it and that their own pages call that normal. One NYC posting, Senior Cybersecurity Operations Engineer (OCaml Experience), shows the security team does build its own tooling in OCaml ([posting](https://www.janestreet.com/join-jane-street/position/8687509002/)). [Block 18](../18-ocaml-literacy/README.md) has the two-evening minimum.

---

## What this means for you, critically

- **Your biggest asset** is that you build: the remediation bot, the sandboxed verifier, Threat-modeling.io, the zero-config scanning pipeline. That's the exact profile the departments page describes. Every answer should route back to something you built.
- **Your biggest risk** is reading as a senior bank manager. "SVP", "governance", "stakeholder alignment", "framework" are words that cost you here. Less hierarchy is a stated value.
- **Second risk:** Windows depth. The role leans Windows; your recent years lean cloud and AppSec. [Block 07](../07-windows-and-endpoints/README.md) is mandatory, not optional.
- **Third risk:** live coding with someone watching. The 2018 London candidate failed on exactly this. [Block 03](../03-coding/README.md) and [block 19](../19-mock-interviews/README.md) are where the hours go.
- **Your edge nobody else will have:** you can talk about agent security using Jane Street's own published sandboxing story, and connect it to the multi-agent DAST verifier you built in a sandbox and to Threat-modeling.io.

[Index](../README.md) · [Next: Process →](../01-process/README.md)
