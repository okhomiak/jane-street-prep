# 01 · The process

[← Jane Street intel](../00-jane-street-intel/README.md) · [Index](../README.md) · [Next: Log investigation →](../02-log-investigation/README.md)

Every Jane Street security loop a candidate has described publicly, side by side, so you can see the shape and the variance. Confirm your own order with the recruiter: Jane Street says the structure varies by discipline ([Interviewing](https://www.janestreet.com/join-jane-street/interviewing/)).

---

## The composite loop

```
Apply ──► Online assessment ──► Phone screen ──► Live investigation ──► Coding ──► Final block ──► Offer
          shell + logs,          culture fit,     1 h Zoom, harder      1 h, your    half day to a day:
          write a CEO briefing   your questions   scenario, narrated     language     fundamentals, scenarios,
                                                                                      tabletop, design, HM
```

Reported timings: OA result in about 3 days; live round result the next day; coding the following week; the whole thing 2 to 6 weeks. Final blocks range from "several scenario interviews over an afternoon plus the hiring manager" to "a day of nearly six hours with interviewers in three offices".

---

## Every reported security loop

| # | Role, office, when | What happened | Questions reported | Source |
| --- | --- | --- | --- | --- |
| 1 | Cybersecurity Analyst, Dec 2025 | Online simulated incident, then write a briefing for the CEO (passed in 3 days). Phone screen on culture and whether the firm suits you. 1 h Zoom with a second, more involved incident, still basic command line (passed next day). 1 h coding the week after, language of choice. About 3 weeks. | Coding: from a log, track consecutive login attempts for one user; then for all users | [Glassdoor](https://static.glassdoor.ch/Interview/Jane-Street-Interview-E255549-RVW99892167.htm) |
| 2 | Cybersecurity Analyst, Sep 2025 | Applied online; one OA; HR replied the same day; one intro call; two online technical rounds; four onsite technical rounds. Rated easy, no offer. | None listed | same page |
| 3 | Cybersecurity SOC Analyst, Hong Kong, Apr 2026 | Three video stages, then in person in HK. Rounds: behavioural, coding, Linux log review, system-analysis whiteboarding, more coding. Interviewers covered exactly what they'd said they would. Rated difficult, no offer. | Coding (unspecified); Linux log review | [Glassdoor](https://www.glassdoor.co.uk/Interview/Jane-Street-Interview-E255549-RVW103405660.htm) |
| 4 | Cybersecurity Analyst, Hong Kong (undated) | OA: a shell with logs from several workstations, work out the attack. First interview: the same exercise live in front of an engineer while explaining your thinking. Second: coding. Then a final onsite. | Fill `def brute_force_detect(entries): pass`, `entries` an array of log-like strings | [Glassdoor HK](https://www.glassdoor.com.hk/Interview/Jane-Street-Interview-Questions-E255549.htm) (captured from a search excerpt; the page blocks re-reading, so not re-verified Nov 2026) |
| 5 | Cyber Security Analyst, Sep 2024 | Several scenario-based interviews online over one afternoon, plus the hiring manager. Technical and non-technical. Rated difficult. | Scenario: a malware incident on a corporate system | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,34.htm) |
| 6 | Security Engineer, Hong Kong, May 2023 | Four stages over about two weeks: short screen; technical questions at undergraduate-course level; live coding in a shared document, discussed rather than run; onsite of three 1 h rounds including a tabletop exercise and secure system design. | List events with failed logins, then alert on failures per user inside a time window. What makes a password strong and how to estimate complexity. How a password manager derives its encryption key. How TOTP turns a secret into a changing code and handles clock drift. The data flow when a user logs into a website, and why POST not GET. | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm) |
| 7 | Security Engineer, New York, Jan 2019 | First screen felt more psychological than technical. Second interview with an interviewer in England, centred on 2FA. Candidate withdrew. Learned nearly everything is built in-house, including email. | "What if I said I didn't want to use 2FA because all my passwords are unique and I'm good at spotting phishing?" | same page |
| 8 | Security Engineer, Aug 2018 | Pre-interview chat on company, role, environment, ethics and process. Out after round one. | Basic password-hygiene questions probing your view on "best" practice; felt like interviewers wanted their own view confirmed | same page |
| 9 | Cyber Security Engineer, London, Jul 2018 | Via recruiter, 6 weeks. Three ~1 h phone calls, one of them coding ("script up a few things" in a language of choice). Then a near six-hour in-person day with staff from Amsterdam and New York on video. Rejected with one line: the coding wasn't up to scratch. | Name five Linux binaries that are two letters long. If you were installing Windows 10 across the company, what you'd change or configure on the out-of-the-box OS. | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm) |
| 10 | DevOps Engineer, Oct 2025 | Via recruiter, one week. One interview with two engineers: a technical task, two technical questions, past experience. | Event-log parsing about failed logins (the same family as the security coding round) | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Interview-E255549-RVW101486068.htm) |
| 11 | Security Engineer (Blind thread) | Poster described the role as mostly Windows-specific with security automation required (no recruiter quoted). Another commenter couldn't get past CV screen for a Threat Modelling Analyst role. | Comparison, from a commenter: a Jump Trading security engineering (IR) loop asked how to position a CPU on a motherboard for multithreaded performance | [Blind](https://www.teamblind.com/post/jane-street-security-engineer-krwvr2zb) |

Pattern across rows 1, 6 and 10 (and the unverified row 4): **the coding round is one problem grown in steps**: failed-login lines, then per-user consecutive count, then all users, then a time window, then the alert decision. The investigation rounds are the same log exercise done live while you narrate. Both are in [block 03](../03-coding/README.md) and [block 02](../02-log-investigation/README.md).

---

## Adjacent Jane Street roles: the same interviewers, one layer deeper

Security sits next to IT, networking, Linux and production engineering, and members rotate. These reports show what the same people ask when they go deep.

| Role | Questions and format reported | Source |
| --- | --- | --- |
| Network Engineer (NY, London) | List every TCP header field and what each is for. Explain BGP local preference. BGP, OSI layers, Linux. Which Cisco commands inspect TCP not initiated from the same interface; Check Point terminology. Process: short tech screen, two technical rounds, ~5 h onsite with five rounds of two interviewers, then the manager. No feedback. | [Glassdoor](https://www.glassdoor.sg/Interview/Jane-Street-Interview-E255549-RVW81660298.htm), [Glassdoor](https://www.glassdoor.co.nz/Interview/Jane-Street-Interview-E255549-RVW87661641.htm) |
| Production Engineer (HK, NY) | FIX logs and system architecture; troubleshoot a client and matching-engine setup; troubleshoot trading-system latency. NY: a LeetCode-like screen with a twist after the first answer, then systems design plus live debugging: search logs on a VM to find the source of an issue. Two phone calls, a behavioural call, onsite coding and hiring manager. | [Glassdoor](https://www.glassdoor.ie/Interview/Jane-Street-Interview-E255549-RVW82145680.htm) |
| Linux Engineer (NY, Sep 2021); Software Developer (2014) | Tell me about yourself and your Linux side projects (Linux Engineer). "List some two-letter Linux commands" (Software Developer, Aug 2014). | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Interview-E255549-RVW57309745.htm), [Glassdoor](https://www.glassdoor.co.nz/Interview/Jane-Street-Interview-E255549-RVW4711912.htm) |
| Trading Systems DevOps (NY, Nov 2018) | Coderpad coding screen in any language: a FizzBuzz-style opener, then layers added. Onsite: three interviews, usually two interviewers each; a coding problem; a command-line exercise exploring an unfamiliar system; a problem using an unfamiliar module or expression parser. Screening question: how do you kill a process. | [Glassdoor](https://www.glassdoor.co.in/Interview/Jane-Street-Interview-E255549-RVW23314043.htm) |
| Desktop Support / IT Operations (HK, London, NY) | Should third-party keyboards be allowed, and why; BadUSB; PowerShell for user and network info; ping vs tracert vs netstat; pushing Windows updates to many laptops; what the BIOS holds; manually joining a domain on a fresh Windows install; what DNS is and how to troubleshoot it; subnets and CIDR; AD vs workgroup; which OS you'd choose for a colleague; a PowerShell skills test; day-to-day scenarios lifted from the JD; "why Jane Street". | [Glassdoor](https://www.glassdoor.com.hk/Interview/Jane-Street-Interview-E255549-RVW38237068.htm), [Glassdoor](https://www.glassdoor.co.nz/Interview/Jane-Street-Interview-E255549-RVW94236091.htm) |
| Technical Support Engineer (Jul 2025) | First round: basic background and behavioural questions, then a short probability brainteaser. Candidate called the process straightforward. | [Glassdoor](https://static.glassdoor.fr/Interview/technical-support-engineer-interview-questions-SRCH_KO0,26_IP939.htm) |
| Software Engineer (London, Dec 2025) | Screen (easy algorithm), recruiter call, onsite system design (design a file system) and coding (hash an immutable tree and compare with a remote via API). Interviewers ask you not to discuss questions; blanket no-feedback policy. | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Interview-E255549-RVW102110329.htm) |
| Ops-engineer interns (NY) | Jan 2026: take-home, a brainteaser phone screen (probability, combinatorics), a coding phone screen, then three final interviews (two on trading-system design, one coding). Undated: Excel HackerRank, then an age puzzle and time for your questions. No security report mentions a brainteaser, but be ready not to freeze. | [Glassdoor](https://www.glassdoor.co.uk/Interview/Jane-Street-Interview-E255549-RVW99434954.htm) |

---

## Your own run of the online assessment

You sat it in May 2026 (ACME Trading Corp scenario at `europe.sneakers.janestreet.com`). What carries into the live round, which candidates describe as the same exercise with an engineer watching:

- 45-minute timer, one question at a time with an answer box, plus a Notes panel.
- Part 1 counts and locates: how many workstation logs, how many days, events on a date, first event time, a binary path, how many employees, an external IP. Part 2 asks for the story: who, how, what they touched.
- Environment: no `file` command, no Ctrl-C, right-click to copy, `/tmp` writable but not persistent.
- Wiki and file-share logs aren't syslog. Open and read a few lines before you grep anything.
- The logs were Windows workstation logs (paths like `C:\Program Files\Chrome.exe`, `C:\tmp\stage2.exe`), which matches the Windows-heavy role signal.

---

## What each round rewards

| Round | They're watching for | Block |
| --- | --- | --- |
| Online assessment | Method, accuracy, a readable briefing for a non-technical reader | [02](../02-log-investigation/README.md) |
| Phone screen | That you understand the culture and want it; the quality of your questions | [00](../00-jane-street-intel/README.md), [13](../13-behavioural/README.md) |
| Live investigation | Narrated method, baseline before hunting, pivoting, a clean write-up, calm | [02](../02-log-investigation/README.md), [09](../09-incident-response/README.md) |
| Coding | Clarifying questions, clean code that grows in parts, tests you write yourself, complexity on request | [03](../03-coding/README.md) |
| Fundamentals | Depth on auth, crypto, web, network, Linux, Windows; one layer deeper than asked | [04](../04-security-fundamentals/README.md) to [08](../08-cloud-and-supply-chain/README.md) |
| Scenario and tabletop | Ordered incident handling, decisions on partial information, communication | [09](../09-incident-response/README.md) |
| Secure system design | Drawing, crown jewels, trade-offs, what you'd skip | [10](../10-threat-modelling-and-design/README.md) |
| Judgement and hygiene | A measured position you can defend under pushback | [11](../11-judgement-and-hygiene/README.md) |
| News | Ranking what matters for a trading firm | [12](../12-security-news/README.md) |
| Hiring manager | Fit, stories, why here, why now, questions for them | [13](../13-behavioural/README.md) |

Full mock scripts for each round: [block 19](../19-mock-interviews/README.md).

[← Jane Street intel](../00-jane-street-intel/README.md) · [Index](../README.md) · [Next: Log investigation →](../02-log-investigation/README.md)
