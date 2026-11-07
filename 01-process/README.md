# 01 · The process

[← Index](../README.md) · [Next: Log investigation →](../02-log-investigation/README.md)

What the Jane Street cybersecurity loop looks like, built only from what candidates have reported and what Jane Street publishes. Confirm the order with your recruiter. Jane Street says the structure varies by discipline and the recruiter explains it ([Interviewing at Jane Street](https://www.janestreet.com/join-jane-street/interviewing/)).

---

## The loop in one picture

```
Apply ──► Online assessment ──► Phone screen ──► Live investigation ──► Coding ──► Final block ──► Offer
          (shell + logs,        (culture,         (1 h Zoom, harder      (1 h, your    (several interviews
           write a briefing)     your questions)   scenario, bash)        language)     + hiring manager)
```

Timing reported by candidates: OA result in about 3 days, live round result the next day, coding the following week, whole process 2 to 6 weeks.

---

## Every reported loop, side by side

| Role, office, when | What happened | Source |
| --- | --- | --- |
| Cybersecurity Analyst, Dec 2025 | Online simulated incident: investigate, then write a briefing for the CEO. Passed 3 days later. Phone screen about the culture and whether it suits you. 1 h Zoom with a second, more involved simulated incident, still only basic bash. Passed the next day. 1 h coding the following week: given a log, track consecutive login attempts for one user, then for all users. "Lots of rounds", communicative, quick to schedule. | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Cybersecurity-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,33.htm) |
| Cybersecurity Analyst, Sep 2025 | Apply online, 1 OA, HR replied the same day, 1 intro call, 2 online technical rounds, 4 onsite technical rounds. | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Cybersecurity-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,33.htm) |
| Cybersecurity Analyst, Hong Kong | OA: a shell with logs from several workstations, work out the attack. First interview: the same thing in front of an engineer while explaining your thinking. Second interview: coding, detect brute force from an array of log-like strings, fill `def brute_force_detect(entries): pass`. Then a final onsite. | [Glassdoor](https://www.glassdoor.com.hk/Interview/Jane-Street-Interview-Questions-E255549.htm) |
| Cyber Security Analyst, Sep 2024 | Several scenario interviews online over one afternoon (one was malware on a corporate system), plus the hiring manager. Technical and non-technical questions. | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,34.htm) |
| Security Engineer, Hong Kong | Two weeks. Technical questions the candidate called undergraduate-level security: password strength and how to estimate complexity, how a password manager should store passwords and get its key, how TOTP turns a secret into a changing code and handles clock drift, the data flow when a user logs into a website and why POST not GET. Coding: failed-login events, decide whether to alert, then add users, then timestamps. | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm) |
| Security Engineer, New York, Jan 2019 | Second interview with someone in England. Asked what the candidate would say to a user who refuses 2FA because their passwords are unique and they spot phishing well. Candidate noted almost everything Jane Street runs is built in house, down to email. | [Glassdoor](https://www.glassdoor.sg/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm) |
| Security Engineer, Aug 2018 | Basic password hygiene questions; they probe your stance on applying "best" practice. | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm) |
| Cyber Security Engineer, London, Jul 2018 | Via recruiter, 6 weeks. Three 1 h phone calls, one of them coding ("script up a few things" in a language of choice). Then an in-person day of nearly 6 hours with interviewers in London, Amsterdam and New York. Rejected with one line of feedback: the coding wasn't up to scratch. Questions: name five two-letter Linux binaries; what you'd change on a fresh Windows install rolled out firm-wide. | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm) |

Adjacent Jane Street roles show what the same interviewers ask when they go deeper:

| Role | Questions reported | Source |
| --- | --- | --- |
| Network Engineer (NY, London) | List every TCP header field you know and what each is for. Explain BGP local preference. BGP, OSI layers, Linux. Which Cisco command inspects TCP not initiated from the same interface. Process: screen, 2 technical rounds, 5 h onsite with 5 rounds of 2 interviewers, manager. | [Glassdoor](https://www.glassdoor.sg/Interview/Jane-Street-Interview-E255549-RVW81660298.htm) |
| Linux Engineer | Tell me about yourself and your Linux side projects. Two-letter Linux commands. | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Interview-E255549-RVW57309745.htm), [Glassdoor](https://www.glassdoor.co.nz/Interview/Jane-Street-Interview-E255549-RVW4711912.htm) |
| Trading Systems DevOps | FizzBuzz-level Unix questions: basic commands, reading files, how you troubleshoot and find help (man pages). | [Glassdoor](https://www.glassdoor.co.in/Interview/Jane-Street-Interview-E255549-RVW23314043.htm) |
| Technical Support Engineer | Networking, openssl, certificates, the TCP handshake. Behavioural: a time you went above and beyond. | [Glassdoor](https://static.glassdoor.fr/Interview/technical-support-engineer-interview-questions-SRCH_KO0,26_IP939.htm) |
| IT Operations Engineer | A PowerShell skills test. Day-to-day scenarios lifted from the job description. Why Jane Street. | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Interview-E255549-RVW90356392.htm) |

---

## Your own run of the online assessment

You sat it in May 2026 at `europe.sneakers.janestreet.com`, ACME Trading Corp scenario. What to remember for the live round, which candidates say is the same format with an engineer watching:

- 45-minute timer, questions appear one at a time with an answer box, plus a Notes panel.
- Part 1 is counting and locating (how many workstation logs, how many days, events on a date, first event time, a binary path, how many employees, an external IP). Part 2 is the story (who did it, how, what they touched).
- Environment quirks: no `file` command, no Ctrl-C, right-click to copy, `/tmp` is writable but not persistent.
- Wiki and file-share logs are not classic syslog. Open and read a few lines before you grep anything.

---

## What Jane Street says it wants

From the [software engineering interview guide](https://www.janestreet.com/preparing-for-a-software-engineering-interview/) and the [interviewing page](https://www.janestreet.com/join-jane-street/interviewing/):

- Four principles: **be nice, be clear, know your language, know what you don't know.**
- The question behind every round is whether they'd want to work with you.
- Open-ended problems with several reasonable answers. No puzzles, no algorithm bingo.
- Real code in a real language. Use the one you know best. OCaml earns no bonus points.
- Asking good questions matters more than knowing every answer.
- They consider you for every open role, so you may be routed to another security team mid-process.
- Phone interviews are by phone; ask for Zoom if your line is bad. Weekdays only. Expect an answer within about a week.

From the [London posting](https://www.janestreet.com/join-jane-street/position/8842136002/): a strong programmer who automates things, code and config in version control, comfortable in an editor, methodical with unstructured investigations, weighs trade-offs and can explain them, follows security news and can tell the cool hack from what matters, good personal security hygiene you can explain, humble, asks questions, admits mistakes.

---

## What each round rewards

| Round | They're watching for | Block to study |
| --- | --- | --- |
| Online assessment | Method, accuracy, a readable briefing | [02 Log investigation](../02-log-investigation/README.md) |
| Phone screen | Whether you've understood the culture, and your questions | [13 Behavioural](../13-behavioural/README.md) |
| Live investigation | Narrated method, baseline before hunting, pivoting, a clean write-up | [02 Log investigation](../02-log-investigation/README.md), [09 Incident response](../09-incident-response/README.md) |
| Coding | Clarifying questions, clean code that grows in parts, tests | [03 Coding](../03-coding/README.md) |
| Fundamentals interviews | Depth on auth, crypto, web, network, Linux, Windows | [04](../04-security-fundamentals/README.md), [05](../05-networking/README.md), [06](../06-linux/README.md), [07](../07-windows-and-endpoints/README.md), [08](../08-cloud-and-supply-chain/README.md) |
| Scenario interviews | Calm, ordered incident handling and communication | [09 Incident response](../09-incident-response/README.md) |
| Design / threat modelling | Drawing, naming the crown jewels, trade-offs, what you'd skip | [10 Threat modelling](../10-threat-modelling-and-design/README.md) |
| Judgement questions | Measured positions you can defend under pushback | [11 Judgement](../11-judgement-and-hygiene/README.md) |
| News | Ranking what matters for a trading firm | [12 News](../12-security-news/README.md) |
| Hiring manager | Fit, stories, why here, why now | [13 Behavioural](../13-behavioural/README.md) |

[← Index](../README.md) · [Next: Log investigation →](../02-log-investigation/README.md)
