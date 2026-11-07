# 04d · Reported fundamentals questions, with sources

[← Block 04](README.md) · [Auth](auth-and-identity.md) · [Crypto/TLS](crypto-and-tls.md) · [Web/AppSec](web-and-appsec.md)

Every fundamentals-style question below was reported by a candidate or published by a hiring manager. Answers live in the teaching pages ([auth](auth-and-identity.md), [crypto](crypto-and-tls.md), [web](web-and-appsec.md), [networking](../05-networking/README.md), [Linux](../06-linux/README.md), [Windows](../07-windows-and-endpoints/README.md), [cloud](../08-cloud-and-supply-chain/README.md)); this page is the drill list, so cover the answer and recite before you check. Tags: [JS] Jane Street, company names as reported, [DM] Daniel Miessler, [GN] gracenolan, [TW] tadwhitaker.

Use it two ways: read a row, say the answer out loud in under a minute, then open the teaching page to check. Mark the ones you stumble on and drill those.

---

## Cryptography and TLS

| Question | Where reported | Source |
| --- | --- | --- |
| What makes a password strong, and how would you estimate its complexity? | [JS] Security Engineer HK 2023 | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm) |
| How does a password manager derive its encryption key, and how should the vault be stored? | [JS] Security Engineer HK 2023 | same |
| How does TOTP turn a secret into a changing code, and cope with clock drift? | [JS] Security Engineer HK 2023 | same |
| How do two devices produce the same TOTP code? | Hiring-manager list | [Logical Cybersecurity](https://james-owens-477n.squarespace.com/career-advice/2019/5/25/interview-questions-to-ask-infosec-candidates) |
| Symmetric vs asymmetric encryption, and when each is used | Datadog, Okta, Amazon intern, Apple | [Blind](https://www.teamblind.com/post/i-did-85-security-engineer-on-sites-with-top-tech-companiesa-prep-guide-lyanpve6), [Glassdoor Okta](https://www.glassdoor.co.uk/Interview/Okta-Interview-E444756-RVW97098835.htm), [Glassdoor Apple](https://www.glassdoor.ca/Interview/Apple-Interview-E1138-RVW94270550.htm) |
| Difference between encoding, encryption and hashing | Datadog, Google D&R, detection loops | [Blind](https://www.teamblind.com/post/Security-Engineer-Interview-at-Google-a37HOvBx), [Medium](https://medium.com/@julieasparks/acing-your-detection-engineering-interview-9ed95f70ffd7) |
| Explain the TLS/SSL handshake and certificate provisioning | Amazon, Google, detection loops | [Blind](https://www.teamblind.com/post/i-did-85-security-engineer-on-sites-with-top-tech-companiesa-prep-guide-lyanpve6) |
| Diffie-Hellman vs RSA; DSA vs RSA | [DM], curated banks | [Miessler](https://newsletter.danielmiessler.com/p/infosec-interview-questions) |
| What is an IV, a block cipher mode, ECB vs CBC | [DM] | same |
| Max length you can encrypt with RSA; how do you encrypt a large file | Hiring-manager list | [Logical Cybersecurity](https://james-owens-477n.squarespace.com/career-advice/2019/5/25/interview-questions-to-ask-infosec-candidates) |
| What is HMAC and why is it built that way | Google | [Glassdoor](https://www.glassdoor.fr/Entretien/Google-Entretien-E9079-RVW252630.htm) |
| How do certificate authorities store their root keys | curated bank | [adonaiohesed wiki](https://adonaiohesed.github.io) |
| Compress then encrypt, or encrypt then compress | [DM], [TW] | [Miessler](https://newsletter.danielmiessler.com/p/infosec-interview-questions) |
| What is forward secrecy; if the server key leaks, can past traffic be read | [DM] | same |
| Rainbow tables and salting | [DM] | same |
| How much do you know about different encryption schemes | SpaceX | [Glassdoor](https://www.glassdoor.com/Interview/SpaceX-Interview-E40371-RVW50019581.htm) |
| AES-256 vs AES-128; MD5 vs Base64 | Hiring-manager list | [Logical Cybersecurity](https://james-owens-477n.squarespace.com/career-advice/2019/5/25/interview-questions-to-ask-infosec-candidates) |
| Is all traffic on port 443 secure | curated bank | [Logical Cybersecurity](https://james-owens-477n.squarespace.com/career-advice/2019/5/25/interview-questions-to-ask-infosec-candidates) |

## Authentication and identity

| Question | Where reported | Source |
| --- | --- | --- |
| Would you decline 2FA if your passwords were unique and you spot phishing? | [JS] Security Engineer NY 2019 | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm) |
| Open-ended password-hygiene questions probing your view on "best" practice | [JS] Security Engineer 2018 | same |
| The data flow when a user logs into a website; POST or GET and why | [JS] Security Engineer HK 2023 | same |
| OAuth vs OIDC; protocols behind SSO | Okta, Google (api security) | [Glassdoor Okta](https://www.glassdoor.co.uk/Interview/Okta-Interview-E444756-RVW97098835.htm) |
| How SSH works, and why keys beat passwords | Amazon AppSec, gracenolan | [Glassdoor](https://www.glassdoor.ca/Interview/Amazon-Interview-E6036-RVW32507244.htm) |
| OAuth bearer-token theft; how PKCE helps | [GN], techinterview LLM loop | [gracenolan](https://github.com/gracenolan/Notes/blob/master/interview-study-notes-for-security-engineering.md) |
| Should MFA apply everywhere, always (MFA fatigue) | hiring-manager thread | [Reddit](https://reddit.sentinel-team.org/posts/1pilmec/snapshots/2025-12-10T22%3A21%3A26.018438Z) |
| 2FA know vs have factors; when is SMS acceptable | [DM] Medium consultant | [Medium](https://medium.com/swlh/the-first-question-i-ask-when-interviewing-someone-for-a-security-role-aaa19ee4f00d) |
| Kerberos, golden ticket, pass-the-hash | curated banks | [adonaiohesed wiki](https://adonaiohesed.github.io) |

## Web and application security

| Question | Where reported | Source |
| --- | --- | --- |
| XSS and its types; XSS vs CSRF | Amazon (STRIDE/XSS loop), Google, [TW] | [Glassdoor](https://www.glassdoor.sg/Interview/Amazon-Interview-E6036-RVW92531625.htm) |
| Find the vulnerabilities in this code; why they exist and how to fix | Amazon AppSec London 2025, T-Mobile | [Glassdoor](https://www.glassdoor.ca/Interview/Amazon-Interview-E6036-RVW92367615.htm) |
| Explain SSRF; SSRF controls in the cloud | Microsoft, Google, [GN] | [Glassdoor](https://www.glassdoor.co.uk/Interview/Microsoft-Interview-E1651-RVW80750670.htm) |
| Mitigate SQL injection; can you take SQLi to RCE | Okta, curated banks | [Glassdoor Okta](https://www.glassdoor.co.uk/Interview/Okta-Interview-E444756-RVW97098835.htm) |
| Keep one risk, business-logic flaw or XSS, which and why | Okta | same |
| All the ways to achieve RCE | Okta | same |
| Same-origin policy, CORS, CSP, cookie flags, a race condition | curated AppSec banks | [adonaiohesed wiki](https://adonaiohesed.github.io) |
| OWASP Top 10, remediation, recent vuln trends | Salesforce, Microsoft Security Response | [Glassdoor](https://www.glassdoor.com/Interview/Salesforce-Interview-E11159-RVW65813660.htm) |
| Deep AppSec: request smuggling, DOM XSS, JWT attacks, does JWT prevent CSRF, zip slip, SSTI, SSRF via DNS rebinding | staff AppSec list | [Blind](https://www.teamblind.com/post/security-engineer-interview-megathread-x7gnmvg2) |
| How is the padlock in the browser generated | [TW] | [tadwhitaker](https://github.com/tadwhitaker/Security_Engineer_Interview_Questions) |
| A specific software supply-chain vulnerability type | CrowdStrike ProdSec | [Glassdoor](https://www.glassdoor.com/Interview/CrowdStrike-Interview-E795976-RVW88299659.htm) |
| How would you secure a LAMP stack | Google | [Glassdoor](https://www.glassdoor.com/Interview/Google-Interview-E9079-RVW84199979.htm) |
| Secure a cloud-based web application; principles of securing APIs | Amazon | [Glassdoor](https://www.glassdoor.sg/Interview/Amazon-Interview-E6036-RVW92531625.htm) |

## Networking and the "what happens when" chain

| Question | Where reported | Source |
| --- | --- | --- |
| What happens when you visit google.com / type a URL | Google, Amazon intern, Cloud SecEng, [DM] | [Blind](https://www.teamblind.com/post/Security-Engineer-Interview-at-Google-a37HOvBx), [Glassdoor](https://www.glassdoor.com/Interview/Amazon-Interview-E6036-RVW71919545.htm) |
| TCP vs UDP; explain each header field | Google Chicago 2025, [JS] Network Engineer | [Glassdoor](https://www.glassdoor.co.uk/Interview/Google-Interview-E9079-RVW95737365.htm), [Glassdoor](https://www.glassdoor.sg/Interview/Jane-Street-Interview-E255549-RVW81660298.htm) |
| Explain BGP; BGP local preference; BGP weaknesses | [JS] Network Engineer, curated | [Glassdoor](https://www.glassdoor.sg/Interview/Jane-Street-Interview-E255549-RVW81660298.htm) |
| What port does ping use (trick); filtered vs closed ports | [DM] | [Miessler](https://newsletter.danielmiessler.com/p/infosec-interview-questions) |
| How does traceroute work; how many packets leave the NIC after plugging in | [DM] | same |
| ARP spoofing on a slow network and its mitigation | curated bank | [adonaiohesed wiki](https://adonaiohesed.github.io) |
| Router vs switch vs access point vs firewall | hiring-manager thread | [Reddit](https://reddit.sentinel-team.org/posts/1pilmec/snapshots/2025-12-10T22%3A21%3A26.018438Z) |
| Describe how a botnet works | Google Dublin 2025, Google 2020 | [Glassdoor](https://www.glassdoor.co.uk/Interview/Google-Interview-E9079-RVW95737365.htm) |
| How does DNS work; DNS covert channels; why DNS monitoring matters | Google Zurich, [DM] | [Glassdoor](https://www.glassdoor.sg/Interview/Google-Interview-E9079-RVW61050703.htm) |
| IDS vs IPS; HIDS vs NIDS; MITM defence; DMZ | curated SOC lists | [adonaiohesed wiki](https://adonaiohesed.github.io) |

## OS, Linux and Windows internals

| Question | Where reported | Source |
| --- | --- | --- |
| Name five two-letter Linux binaries | [JS] Cyber Security Engineer London 2018 | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm) |
| What would you change on a fresh firm-wide Windows build | [JS] same | same |
| Inode; hard vs soft link; du vs df; what mv * does; install Linux on 100 nodes | HRT Systems Engineer | [Glassdoor](https://www.glassdoor.com/Interview/Hudson-River-Trading-Systems-Engineer-Interview-Questions-EI_IE470937.0,20_KO21,37.htm) |
| Pipe vs redirect; kill -15 vs -9; Python yield, generators, decorators | HRT SRE | [Glassdoor](https://www.glassdoor.ca/Interview/Hudson-River-Trading-Interview-E470937-RVW98611866.htm) |
| Where are passwords stored; /etc/shadow hash algorithms; effective vs real UID; SUID vs sticky bit; drop privileges | deep Linux list | [adonaiohesed wiki](https://adonaiohesed.github.io) |
| How does a buffer overflow work and how do you defend against it | Google, [DM] | [Glassdoor](https://www.glassdoor.fr/Entretien/Google-Entretien-E9079-RVW252630.htm) |
| Default Windows logs; 4624/4625; the logon-type field; RDP event ID | curated SOC list | [adonaiohesed wiki](https://adonaiohesed.github.io) |
| Where on a disk would you look for a remote-login trace and for installed apps | Amazon intern | [Glassdoor](https://www.glassdoor.com/Interview/Amazon-Interview-E6036-RVW71919545.htm) |
| View a file without vi, emacs, cat, less or more | hiring-manager list | [Logical Cybersecurity](https://james-owens-477n.squarespace.com/career-advice/2019/5/25/interview-questions-to-ask-infosec-candidates) |
| Memory tagging from the silicon and kernel layer; XNU, Mach ports, sandbox profiles | Apple Offensive Security | [Glassdoor](https://www.glassdoor.com/Interview/Apple-Interview-E1138-RVW104627847.htm) |

## Cloud, design and program-building (overlap with [08](../08-cloud-and-supply-chain/README.md) and [10](../10-threat-modelling-and-design/README.md))

| Question | Where reported | Source |
| --- | --- | --- |
| Build a security program for a small company | Google Zurich | [Glassdoor](https://www.glassdoor.sg/Interview/Google-Interview-E9079-RVW61050703.htm) |
| Scalable security controls in CI/CD without slowing developers | Amazon | [Glassdoor](https://clear.glassdoor.nl/Interview/Amazon-Interview-E6036-RVW98613827.htm) |
| Design a secure system for remote work; a secrets service like Vault; a symmetric key-management system | Meta ProdSec, Blind onsites, Block | [Glassdoor](https://www.glassdoor.com/Interview/Meta-Product-Security-Engineer-Interview-Questions-EI_IE40772.0,4_KO5,30.htm), [Glassdoor](https://www.glassdoor.com/Interview/Block-Interview-E422050-RVW72068498.htm) |
| Public S3; segment workloads; secure Kubernetes in prod; CI/CD supply chain; IAM for employees plus partners | curated cloud list | [adonaiohesed wiki](https://adonaiohesed.github.io) |
| Traffic to 169.254.169.254, what do you suspect | Amazon | [Blind](https://www.teamblind.com/post/i-did-85-security-engineer-on-sites-with-top-tech-companiesa-prep-guide-lyanpve6) |
| How do you implement an SBOM | Apple | [Glassdoor](https://www.glassdoor.ca/Interview/Apple-Interview-E1138-RVW94270550.htm) |

## Risk and judgement (overlap with [11](../11-judgement-and-hygiene/README.md))

| Question | Where reported | Source |
| --- | --- | --- |
| Threat vs vulnerability vs risk, and how they relate | Uber, curated | [Glassdoor](https://static.glassdoor.nl/Interview/Uber-Security-Engineer-Interview-Questions-EI_IE575263.0,4_KO5,22.htm) |
| Which CIA element matters most in this industry, pick one and argue | hiring-manager thread | [Reddit](https://reddit.sentinel-team.org/posts/1pilmec/snapshots/2025-12-10T22%3A21%3A26.018438Z) |
| A CEO wants a blocked site opened; a firewall exception for an executive | [DM], hiring-manager list | [Logical Cybersecurity](https://james-owens-477n.squarespace.com/career-advice/2019/5/25/interview-questions-to-ask-infosec-candidates) |
| Open-source vs proprietary: more secure? Insiders vs outsiders: more dangerous? | [DM] | [Miessler](https://newsletter.danielmiessler.com/p/infosec-interview-questions) |
| What is the goal of security in an organisation | [DM] | same |
| Explain a critical vulnerability to five different audiences, ending with a non-technical relative | hiring-manager thread | [Reddit](https://reddit.sentinel-team.org/posts/1pilmec/snapshots/2025-12-10T22%3A21%3A26.018438Z) |

If you can answer every row here out loud in under a minute, the fundamentals interviews hold no surprises. The ones Jane Street is most likely to push on, based on its own reports, are the five in the [block 04 overview](README.md) plus TCP, DNS, the login data flow, buffer overflows, and "what happens when you type a URL".

[← Block 04](README.md) · [Auth](auth-and-identity.md) · [Crypto/TLS](crypto-and-tls.md) · [Web/AppSec](web-and-appsec.md)
