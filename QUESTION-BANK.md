# Master question bank

[Index](README.md) · [Sources](SOURCES.md)

Every reported and drilled question in this repo, in one place, so you can self-test. Each question links to the block that answers it; the source tag and URL live in that block. Cover the answer, say yours out loud, then check.

For timed, try-before-reveal practice with follow-ups and the common wrong answers, use the [drill app](drill/README.md); this page is the complete inventory.

How to use it: work the **Jane Street confirmed** set until every one is automatic. Then drill the per-round lists, hardest-first. The questions marked **[JS]** were reported in an actual Jane Street loop or asked in your own online assessment; everything else is from adjacent Jane Street roles, trading-firm and big-tech security loops, or curated banks, and is fair game because the same interviewers go one layer deeper.

---

## Jane Street confirmed (drill these first)

These came from Jane Street security or near-security loops, or from your own OA. Highest probability, so they get the most reps.

| # | Question | Block |
| --- | --- | --- |
| 1 | From a log, track consecutive failed login attempts for one user, then extend to all users | [03](03-coding/README.md) |
| 2 | Fill `def brute_force_detect(entries): pass` where `entries` is log-like strings | [03](03-coding/README.md) |
| 3 | List events with failed logins, then alert on failures per user inside a time window | [03](03-coding/README.md) |
| 4 | Event-log parsing of failed logins (DevOps loop, same family) | [03](03-coding/README.md) |
| 5 | Investigate multi-workstation shell logs live and narrate the attack | [02](02-log-investigation/README.md) |
| 6 | Write a briefing on the incident for the CEO | [02](02-log-investigation/README.md) |
| 7 | Linux log review (HK SOC loop) | [02](02-log-investigation/README.md), [06](06-linux/README.md) |
| 8 | System-analysis whiteboarding (HK SOC loop) | [10](10-threat-modelling-and-design/README.md) |
| 9 | What makes a password strong, and how would you estimate its complexity? | [04](04-security-fundamentals/README.md) |
| 10 | You're implementing a password manager: how does it store passwords and where does the key come from? | [04](04-security-fundamentals/README.md) |
| 11 | TOTP: how does the secret become a changing code, and how does it cope with clock drift? | [04](04-security-fundamentals/README.md) |
| 12 | Walk through the data flow when a user opens a website and logs in. POST or GET, and why? | [04](04-security-fundamentals/README.md), [05](05-networking/README.md) |
| 13 | "I don't want 2FA. My passwords are unique and I'm good at spotting phishing." | [11](11-judgement-and-hygiene/README.md) |
| 14 | Tabletop exercise and secure system design (onsite rounds) | [09](09-incident-response/README.md), [10](10-threat-modelling-and-design/README.md) |
| 15 | Scenario: a malware incident on a corporate system | [09](09-incident-response/README.md), [15](15-malware-forensics-reversing/README.md) |
| 16 | Name five Linux binaries that are two letters long | [06](06-linux/README.md) |
| 17 | Installing Windows 10 across the company: what do you change on the out-of-the-box OS? | [07](07-windows-and-endpoints/README.md) |
| 18 | "Script up a few things" in a language of your choice (coding call) | [03](03-coding/README.md) |
| 19 | Why Jane Street? Why leave your current role? Questions for us? | [13](13-behavioural/README.md) |
| 20 | Personal security setup, and coaching others on good habits | [11](11-judgement-and-hygiene/README.md) |

---

## Coding ([block 03](03-coding/README.md))

Added November 2026, for the newer round: review a model-written `brute_force_detect` and rank the bugs; why its own tests passed; what LLMs are good and bad at when you write code, and when you stop using one. The Jane Street failed-login build-up (rows 1 to 4 above), plus: Apache failures in a time window; correlate two log formats; impossible travel; sessionise events; extract IOCs; lateral-movement graph (BFS); detect a port scan; join logs in SQL with a timestamp gotcha; and classics (palindrome, two-sum, IPv4 validation, password generator, average a field in a 20-field CSV without pandas, decode a base64+gzip log and count an IP). Also a secure code-review drill, and the tested bash one-liners.

**Know your language ([Python internals](03-coding/know-your-language-python.md)):** what is the GIL and when do threads help; what `yield` does, generators vs lists; what a decorator is and what it's for; context managers; mutable default arguments; `is` vs `==`; how a dict works and the hash-flooding DoS; exceptions you must not swallow; finding a memory leak; `random` vs `secrets`, `subprocess` safely, why `pickle`/`yaml.load`/`eval` are RCE; constant-time comparison; path containment; type hints and dataclasses.

## Log investigation ([block 02](02-log-investigation/README.md))

The six-step method; the ACME/Windows-workstation OA; the CEO briefing; and the verbal drills (what do you look at first, how do you baseline, when do you stop looking).

## Security fundamentals ([block 04](04-security-fundamentals/README.md))

**Auth and identity:** impersonation vs delegation in token exchange; how fast a leaver's sessions die and how Shared Signals pushes it; what goes wrong with a cache in front of an access decision (added November 2026); password storage; rainbow tables and salting; a stolen DB hashed with one round of SHA-256 and a static salt, now what; why scheduled rotation is harmful; account brute-forcing; biometrics vs passwords; cookies vs sessions vs tokens; HTTP statelessness; JWT and what goes wrong; OAuth 2.0 and OIDC, bearer-token theft; SAML and SSO; Kerberos; authn vs authz, ACLs/RBAC/service accounts; comparing MFA methods; U2F/FIDO2 phishing resistance; getting the whole firm onto a password manager.

**Crypto and TLS:** encoding vs encryption vs hashing vs signing vs obfuscation; symmetric vs asymmetric; which key encrypts and which signs; block vs stream ciphers and modes; IV reuse; HMAC; the length-extension attack; timing and other side channels; the TLS handshake; forward secrecy; certificate chains and pinning; notable TLS attacks.

**Back-of-envelope ([maths](04-security-fundamentals/back-of-envelope.md)):** estimate a password's strength and crack time; how much rate limiting buys online; the birthday bound and why 128-bit hashes give 64-bit collision resistance; why GCM nonces have a key budget; a 99%-accurate detector whose alerts are mostly false (base-rate fallacy); events per second and storage for a SIEM; how fast a network scan or ransomware runs; expected value and conditional probability if a brainteaser appears.

**Web and appsec:** same-origin and CORS with preflight; HSTS, certificate transparency, HttpOnly; XSS; CSRF; SQL injection; SSRF; XXE/IDOR/traversal/LFI-RFI/open redirects/deserialisation; API security; secure login on a high-traffic site; securing a LAMP stack; MITM; conducting a secure code review; GitHub webhook abuse and secret scanning.

## Networking ([block 05](05-networking/README.md))

What happens when you type a URL; the TCP handshake and SYN/ACK; TCP vs UDP; every TCP header field; the OSI model; DNS end to end; traceroute at the protocol level and ping's port; the packets-off-the-NIC onion; ARP and ARP spoofing; what a VPN hides and exposes; BGP, local preference and the OPEN message; subnets/masks/router vs switch/broadcast and collision domains; firewalls, IDS vs IPS, filtered vs closed; IPsec phase 1 vs 2; detecting a DDoS, a port scan, network-side brute force; troubleshooting intermittent cross-datacentre latency; reverse-engineering a custom protocol from a pcap.

## Linux ([block 06](06-linux/README.md))

Five two-letter binaries; inodes and hard vs soft links; du vs df; `mv *`, alternatives to `ls`, installing Linux on 100 nodes; RAID 5 vs 6; finding an unauthorised process from logs; IPC security trade-offs; first three steps hardening a server; privilege escalation paths; SELinux/MAC vs DAC and what /proc, /tmp, /etc/shadow expose; the syscall path; buffer overflows and defences; hardening SSH; Linux vs Windows; container isolation (namespaces, cgroups, seccomp); pipe vs redirect, `kill -15` vs `-9`, how you kill a process; high CPU and Python memory leaks; hardening a laptop for Defcon and a home network.

## Windows and endpoints ([block 07](07-windows-and-endpoints/README.md))

Fresh firm-wide Windows build; the login event IDs and 4624/4625; detecting pass-the-hash; Kerberoasting; golden/silver tickets, Mimikatz, DCSync; investigating a PowerShell/scheduled-task/EDR alert; how AD and LDAP relate; running PKI and where AD CS goes wrong (ESC1/ESC8); Entra ID and hybrid-identity attack paths (PRT, OAuth consent, Golden SAML); registry and Group Policy, SMB; EDR vs AV and ASR rules; a user's PC acting weird; macOS essentials; hardware and firmware (Secure Boot vs measured boot, TPM, DMA attacks, BadUSB, firmware inventory, MTE, NUMA and why the order path has no inline inspector).

## Cloud and supply chain ([block 08](08-cloud-and-supply-chain/README.md))

Added November 2026: the 2025 and 2026 supply-chain incidents and their pattern; CI that issues OIDC tokens is an identity provider. Biggest AWS weaknesses; IAM role vs policy and allow/deny evaluation; protecting S3 and auto-remediating a public bucket; securing EC2 and central monitoring; SSRF to the metadata service and IMDSv2; KMS and envelope encryption, serverless; securing containers and Kubernetes RBAC; containers vs VMs vs clusters, escapes, hyperjacking; Spectre/Meltdown and site isolation; a Docker-image pipeline; CI/CD for security; a secret-storage system; supply-chain attack prevention (and the agentic supply chain); service-account lateral movement and BeyondCorp; WAF; infrastructure as code.

## Incident response ([block 09](09-incident-response/README.md))

Malware on a corporate system; failed logins then a success from an odd location; a clicked phishing link; unusual outbound/beaconing; ransomware; an AWS key pushed to a public repo; the impossible-travel/PowerShell/scheduled-task/lateral-movement chain; a suspected insider exfiltrating; the ISP-says-you're-compromised role-play; building a timeline, symptom vs root cause; when to involve legal/users/management and when to stop the attacker; handling an ambiguous alert or your own mistake; tuning a noisy rule, measuring a SOC, detecting credential dumping; an internal AI agent acting out of bounds; a market-hours runaway-order incident.

## Threat modelling and design ([block 10](10-threat-modelling-and-design/README.md))

**Exercise prompts, reported ([interview-questions.md](10-threat-modelling-and-design/interview-questions.md)):** threat-model a Signal-type messaging app (Meta); threat-model a sample application (Meta phone screen); go over a product and threat-model it (Amazon); threat modelling, STRIDE, XSS types (Amazon 2026); presigned-URL object storage, five threats and remediations ranked; a photo app on a cloud bucket for an external vendor; threat-model a new Azure service before launch (Microsoft); threat-model a new feature (Stripe); a read-only report link emailed to a contractor; a support bot that issues refunds via a tool call; a new server motherboard (Google); S3 vs EC2, a smart-home device, an image-upload service, a newly acquired company's infrastructure (Amazon prep guides); system-analysis whiteboarding and secure system design (Jane Street HK); connect a startup to the French exchange (Optiver).

**Worked models to drill cold ([worked-models.md](10-threat-modelling-and-design/worked-models.md)):** Jane Street's permissions service Ontology (from their 2019 talk); a new venue connection; presigned-URL storage; a Signal-type app; a refund-capable support bot; an alternative-data research pipeline.

**Conceptual:** what is threat modelling, which process, how do you perform it; its benefits; threat modelling vs vulnerability scanning vs pentesting; architecture review vs threat modelling; proactive vs reactive; which framework and why, and what is wrong with DREAD; STRIDE per element vs per interaction; give me a repudiation threat; how do you rank (and why not CVSS); how do you involve stakeholders and communicate to non-technical ones; which systems first across a firm; how do you measure whether it is working; what tools; what if you find a serious threat mid-model; the Manifesto's four anti-patterns.

**Trading-firm specific ([hedge-fund-threat-model.md](10-threat-modelling-and-design/hedge-fund-threat-model.md)):** the crown jewels of a trading firm in order; how a prop firm differs from a client-money hedge fund; the actors and their current playbook; the Levitas Capital BEC chain and its lesson; structuring under an alert threshold; why the 2014 "HFT slowed by milliseconds" story should not be cited as fact; adversarial perturbation of market data; the abuse-case library; the one-page reference model.

**Secure-design prompts (in the hub):** name three attack vectors and mitigations for an architecture; design network-anomaly monitoring; design a SIEM; how a botnet works (defensive framing); design a secure supply chain / secret store / CI-CD pipeline; teach threat modelling to a new team; threat-model an LLM agent system; design a firm-wide permissioning system; design a vulnerability-management programme; design the security automation for a SOC.

## Judgement and hygiene ([block 11](11-judgement-and-hygiene/README.md))

The 2FA pushback; your own security setup; is a password notebook ever acceptable; password strength and vault keying; TOTP vs push vs hardware keys and rotation; the goal of information security; threat vs vulnerability vs risk; instant-CISO first priorities; insiders vs outsiders, open vs closed source; why companies don't fix vulns and how to measure a security team; where you get news and who you respect; bug bounties vs testing; what you run at home.

## Security news ([block 12](12-security-news/README.md))

Be able to rank five current stories for a trading firm and name one big-headline-low-relevance item. Refresh the week before.

## Behavioural ([block 13](13-behavioural/README.md))

The 60-second intro; why Jane Street; why leave J.P. Morgan; compensation; questions for them; tell me about yourself; a mistake and what you learned; a high-pressure incident; a time you went above and beyond; how you stay current; defending a technical decision to a senior stakeholder; why a trading firm over big tech; what you would do in your first 90 days.

## Malware, forensics, reversing ([block 15](15-malware-forensics-reversing/README.md))

First move on a suspicious hash; static vs dynamic analysis and static-analysis tools; a web-shell attack diagram and Mimikatz detail; Windows (and some Linux) forensics; analysing a memory image; NTFS artefacts and acquiring evidence; host-based memory forensic tools; translating assembly to normal code; Apple-style silicon/kernel questions; malware families and techniques; how WannaCry/NotPetya spread.

## AI, LLM and agent security ([block 16](16-ai-and-llm-security/README.md))

Added November 2026: building the human-approval step so it survives the attacks Microsoft's red team found most effective; removing the vendor API keys from an internal model broker; what changed in MCP in 2026 and why a gateway cares. The refund-agent walk-through; an indirect injection that exfiltrates an inbox and removing a leg of the trifecta; why a secret system prompt isn't enough; talking the team out of DB write access; getting a toy agent to run a shell command; shipping a 95% classifier; red-teaming an LLM feature; direct vs indirect injection and the OWASP risk; MCP tool poisoning, rug pulls and scoped tokens; the RAG cosine-similarity question; how AI security differs from AppSec; securing an internal AI coding assistant; and describing Jane Street's own agent sandbox back to them.

## Trading domain ([block 17](17-trading-domain/README.md))

Why latency and market hours change the controls; the trade path and a crossed book; FIX for a security engineer; pre-trade risk and kill switches (SEC 15c3-5); time sync as a forensics control; the Knight Capital story; "connect your startup to the French exchange"; insider and alpha protection.

## OCaml literacy ([block 18](18-ocaml-literacy/README.md))

The "why OCaml" answer; reading a variant/pattern-match/option snippet; the ecosystem names; the memory-safety and AI-verification framing; the honest "never shipped it" close.

## Detection engineering and email security ([block 21](21-detection-engineering/README.md))

From "we should detect X" to a rule in production (the lifecycle); write a Sigma rule and get it onto Elastic or ElastAlert (the rule types: frequency, spike, flatline, new_term, cardinality); the pyramid of pain and why behaviours beat indicators; tuning and the economics of false positives; how to measure a detection programme or a SOC; what threat hunting is and how to run a hunt; consuming threat intelligence; Zeek vs Suricata vs NetFlow; detecting a beacon and detecting DNS tunnelling; SPF, DKIM and DMARC and what each actually stops; business email compromise; analyse this suspicious email.

## The Threat Modelling role ([block 22](22-threat-modelling-role/README.md))

Threat-model this system, live; how would you attack this architecture, then how would you fix it; a team has never threat-modelled, how do you start and make it stick; a system owner says the risk is not real; walk me through a security decision you documented (options, trade-off, rationale); how do you prioritise which systems to model first; why this role rather than the generic engineer role.

## AI-assisted threat modelling ([block 23](23-ai-threat-modelling/README.md))

How does an AI-assisted threat-modelling system actually work (the pipeline); what goes wrong with AI-generated threat models (the eleven failure modes); if you built it here, what would you do differently; show me how it runs on a real change (the exchange-connectivity example); how would you scale threat modelling across hundreds of systems with a small team; how do you know the output is any good.

## The application and the screen ([block 25](25-the-application/README.md))

Why are you interested in Jane Street (the free-text box); how did you hear about us; if you have a job now, why are you looking; have you interviewed with Jane Street before, and what has changed since; why this role rather than the other security opening; walk me through your GitHub.

## The recruiter call ([block 26](26-the-recruiter-call/README.md))

Walk me through your background; why Jane Street; why leave an SVP role; you've applied before, what's changed; why the moves; salary expectations; what you earn now; anything you'd leave behind; notice and start date; other processes; three days in the office; the Threat Modelling role as well; right to work; why move you forward.

---

Full timed scripts that chain these into real rounds: [block 19](19-mock-interviews/README.md). Your personal weak-spot plan: [block 20](20-gap-analysis/README.md).
