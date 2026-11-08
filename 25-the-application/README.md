# 25 · The application: CV, the free-text box, and re-applying

[← Six-month roadmap](../24-six-month-roadmap/README.md) · [Index](../README.md)

The only Jane Street security failure reported by a candidate in your position happened before any interview: a CV screen for the Threat Modelling role ([block 22](../22-threat-modelling-role/README.md)). You have applied to Jane Street before, more than once, so this block is about getting through the door with a profile that reads as a hands-on builder, and about handling the re-application honestly. It rests on their own words: the application form, the interviewing FAQ, and the two postings. Tags: [JS] their pages, [est] established.

---

## What the screener is pattern-matching for

Read the two postings as a screener would ([block 00](../00-jane-street-intel/README.md), [22](../22-threat-modelling-role/README.md)). They are scanning for evidence of: builds things in a real language; has found and fixed vulnerabilities personally; Linux and scripting; threat modelling done with teams, not for them; offensive background; communicates and teaches; makes trade-offs. They are scanning *against*: management-only recent years, governance vocabulary, programme and framework language, and a title that suggests you will not be in the terminal.

The structural risk in your CV is that your most recent and most senior role is the one most likely to be read as managerial. The fix is not to hide seniority; it is to make the first thing they read about J.P. Morgan a thing you built.

---

## CV principles for this firm

1. **Lead every role with an artefact, then its effect, then the scope.** Built X, which did Y, across Z. Scope last, because scope is what reads as management.
2. **Pick the proof points that say "builder".** Keep: the automated remediation bot and the 40% fall in post-deployment incidents; the sandboxed multi-agent AI DAST verifier; the zero-config GitHub Actions scanning pipeline (Trivy, TruffleHog, Gitleaks); 35% faster change approvals (security that sped engineering up); 450+ engineers trained (the teaching requirement); the team grown 5 to 13 at Wise (one line, not a theme). Glasswing, your EMEA AI security programme at J.P. Morgan, as the frame for the agentic and LLM threat-modelling work. Threat-modeling.io as a founder line with its stack (Python, FastAPI) and outputs (STRIDE, ATT&CK, OWASP).
3. **Drop the proof points that say "bank".** Compliance-alignment percentages and vendor KPI adherence are real achievements that will cost you here. They read as governance. Leave them for other applications.
4. **Name the technologies you actually touched.** Python, the specific scanners, the cloud provider, the CI system, FIX if you worked near it. The postings list Linux and scripting; the CV should show them without being a keyword list.
5. **Offensive background gets its own visible line.** Synopsys offensive security consulting and the Cambridge reverse-engineering course, plus the recent offensive work (the DAST verifier is attack generation and verification). The Threat Modelling posting requires this; make it findable in five seconds.
6. **Titles truthful, not leading.** "Security Architect (SVP), J.P. Morgan" is accurate; "SVP" as the first word of the role line is the risk. Role function first, grade in brackets.
7. **One page of signal, two at most.** Older roles compress to a line each; the national-infrastructure work in Ukraine is a strong, distinctive line, not a paragraph.
8. **A links line at the top.** GitHub (with the artefacts from [block 24](../24-six-month-roadmap/README.md) public), Threat-modeling.io, and the published AI threat-modelling piece. The repo is the evidence layer; the CV points to it.

### Bullets, before and after

Rewritten in your own voice. The "before" versions are the bank register the screener will bounce on.

| Reads as management | Reads as a builder |
| --- | --- |
| Led security governance and stakeholder alignment across a cross-asset trading platform programme | Own product security for a greenfield cross-asset trading platform (equities, FX, fixed income): threat models with the desks' engineers, security design reviews on the order and market-data paths, and the controls that came out of them |
| Drove a 40% reduction in post-deployment security incidents through process improvement | Built an automated remediation bot for scanner findings; post-deployment security incidents fell 40% |
| Established an AI security programme for the EMEA region | Run Glasswing, J.P. Morgan's EMEA AI security programme: agent sandboxing, LLM threat modelling, and a multi-agent AI DAST verifier that validates findings in an isolated sandbox |
| Improved compliance alignment by 25% and ensured vendor KPI adherence | (drop for this application) |
| Delivered security awareness and training initiatives | Taught threat modelling and secure design to 450+ engineers; sessions run on the teams' own systems |
| Managed the security team and scaled headcount | Grew the Wise security team from 5 to 13 while still owning the AppSec tooling (SAST, SCA, bug bounty triage) |

The test for every bullet: could a Jane Street engineer, reading it cold, picture you doing the thing? If the picture is a meeting, rewrite it.

---

## The free-text box

The application form says they do not need a cover letter and offers one box for anything helpful, with three suggested prompts: why you are interested, how you heard, and if you have a job now, why you are looking ([application form](https://www.janestreet.com/join-jane-street/apply/7988179002)). Three short paragraphs, builder's voice, specific. Drafts to adapt, not paste:

**Why you are interested.**

> I want to do security at a firm that builds its own systems and expects its security engineers to build too. The Threat Modelling role describes my work: sitting with system owners who know their application but not its exploits, modelling it together, and turning that into concrete changes. I run J.P. Morgan's EMEA AI security programme and founded Threat-modeling.io, so the question of how a small security team keeps up with hundreds of in-house systems, and where AI genuinely helps and where it misleads, is the one I most want to work on. Your published work on sandboxing AI agents is the most serious treatment of that problem I have seen from a trading firm, and I would like to contribute to it as a peer.

**How you heard.**

> Through your engineering writing and Signals and Threads, and through conversations with people on your technology teams. (Name them only if they have agreed to be named.)

**Why you are looking.**

> I have applied to Jane Street before and did not get through; since then I have deliberately changed what I can show. In the last six months I have shipped [the detections repo], written [the OCaml tool] as my first OCaml project, and published [the piece on AI-assisted threat modelling], all public on my GitHub. I am leaving a role I am proud of because I want to spend more of my time building and less of it steering, in a place where the security team is a group of programmers.

Fill the brackets with the real artefacts from [block 24](../24-six-month-roadmap/README.md); do not submit until they exist. Keep the whole box under 250 words.

---

## Re-applying, honestly

- **Answer "Have you interviewed with Jane Street before?" with yes.** They know. The FAQ is explicit that re-interviewing is normal and that they have hired people on their second or third attempt ([Interviewing FAQ](https://www.janestreet.com/join-jane-street/interviewing/)).
- **Meet their stated condition.** For experienced hires they recommend waiting until your experience or circumstances have changed significantly. Make that literally true and literally visible: the five public artefacts, the sharper target, a CV rewritten around what you build. Then say so in one sentence, as in the draft above.
- **Do not explain the previous outcome at length.** One clause ("did not get through") and then what changed. Anything longer reads as litigation.
- **Apply to both security roles.** Their FAQ says applications are considered for all open roles globally and that being considered for more than one team is common. One coherent story, two roles ([block 22](../22-threat-modelling-role/README.md)).

---

## Referrals and outreach

The FAQ does not mention referrals, so there is no stated process. In practice an internal referral is most valuable at exactly the stage that stopped you before, the CV screen, because a named colleague vouching that you are hands-on counters the title. You have been in contact with people at the firm; the right ask, once your public work exists, is specific and small: would they be willing to pass your application to the security team's hiring manager with a line about what they know of your work. Never ask someone to vouch for work they have not seen; send them the repo first.

---

## Timing and the last checks

- **Apply when the artefacts exist, not when the posting appears.** The security postings have stayed open and been reposted through the year; you are not racing a deadline, you are racing your own readiness.
- **The week before you submit:** re-read the postings for any change; make sure the GitHub repos have READMEs a stranger can follow; check that nothing public contradicts the CV; re-run the two-minute intro ([block 13](../13-behavioural/README.md)) and make sure the CV and the intro tell the same story.
- **After you submit:** expect the online assessment again or a recruiter screen first ([block 01](../01-process/README.md)); start the 14-day sprint in [block 20](../20-gap-analysis/README.md) the day you hear back.

[← Six-month roadmap](../24-six-month-roadmap/README.md) · [Index](../README.md)
