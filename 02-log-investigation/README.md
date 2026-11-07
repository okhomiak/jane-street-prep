# 02 · Log investigation (online assessment and live round)

[← Process](../01-process/README.md) · [Index](../README.md) · [Next: Coding →](../03-coding/README.md)

Two of Jane Street's rounds are the same exercise: a shell, a pile of logs from a small fictional firm, and the question "what happened?". The online version is timed and ends with a written briefing. The live version is an hour on Zoom with an engineer watching and a harder scenario, still only basic bash ([Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Cybersecurity-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,33.htm)). What they score is your method and how you explain it, not typing speed.

---

## What candidates report

| What | Detail | Source |
| --- | --- | --- |
| Online assessment | A shell with access to logs from several workstations. Work out the attack. | [Glassdoor HK](https://www.glassdoor.com.hk/Interview/Jane-Street-Interview-Questions-E255549.htm) |
| Online assessment, Dec 2025 | A simulated incident to investigate, then write a briefing for the CEO. | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Cybersecurity-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,33.htm) |
| Live round | The same format in front of an engineer while you explain your thinking. The second scenario is "a bit more involved" but needs only basic bash. | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Cybersecurity-Analyst-Interview-Questions-EI_IE255549.0,11_KO12,33.htm) |
| Your own run (May 2026) | 45-minute timer, one question at a time with an answer box and a Notes panel. Part 1 counts and locates things; Part 2 asks for the story. No `file`, no Ctrl-C, `/tmp` writable but not persistent. Wiki and file-share logs aren't syslog, so read them before grepping. | Your assessment |

Questions from other firms that test the same skill:

| Question | Where | Source |
| --- | --- | --- |
| Given suspicious Linux logs, how would you decide whether an unauthorised process is running? | Jane Street prep guide | [Dataford](https://dataford.io/interview-guides/jane-street/security-engineer) |
| Walk me through investigating a potential breach in production. | Jane Street prep guide | [Dataford](https://dataford.io/interview-guides/jane-street/security-engineer) |
| What would you do if you discovered an infected host? | Security engineer loops, 2016 collection | [tadwhitaker](https://github.com/tadwhitaker/Security_Engineer_Interview_Questions) |
| You have a memory dump from a possibly compromised machine. How do you approach it? | Same collection | [tadwhitaker](https://github.com/tadwhitaker/Security_Engineer_Interview_Questions) |
| Here is a packet capture. Identify the protocol, the traffic, and whether it looks malicious. | Same collection | [tadwhitaker](https://github.com/tadwhitaker/Security_Engineer_Interview_Questions) |
| Walk through analysing a suspicious email: headers, content, attachments, static and dynamic analysis, verdict, signature. | Security engineer interview | [Glassdoor](https://static.glassdoor.es/Interview/security-engineer-interview-questions-SRCH_KO0,17_IP445.htm) |
| Optiver's security OA: a scripting task plus "analyse this file". The candidate failed the file part for not saying what actions they'd take. | Optiver Security Engineer | [Glassdoor](https://www.glassdoor.com/Interview/Optiver-Security-Engineer-Interview-Questions-EI_IE243355.0,7_KO8,25.htm) |

That Optiver note is the whole lesson: always finish with what you'd do next.

---

## The six-step loop

Say each step out loud before you run anything.

| Step | Minutes | What you do | What you say |
| --- | --- | --- | --- |
| 1. Orient | 2 | `ls -la`, `wc -l`, `head -3` on every file | "Before I search for anything, I want to know what each log is and how it's shaped." |
| 2. Baseline | 5 | Count event types, users, hosts, days | "This tells me what normal looks like, so the odd things stand out." |
| 3. Find the rare | 10 | Sort by count and read the bottom of the list | "Attackers are usually the rare thing, not the common thing." |
| 4. Pivot | 10 | Take one indicator (IP, user, host) across every log | "I'll follow this IP through everything we have." |
| 5. Timeline | 5 | Merge the suspect's events into one sorted view | "What happened first? That's usually initial access." |
| 6. Write up | 5 | Finding, evidence, scope, impact, do now, check next | "Here's what I'm confident about and what I'm not." |

Habits that score:

- Say what you expect a command to show before you run it, then say whether it did.
- Keep a running notes file: append with `| tee -a /tmp/notes.txt`.
- Ask early: "Are all timestamps UTC?", "Is it fine to use awk?", "Can I write to /tmp?"
- Prefer short commands you can type from memory. `grep -e A -e B` beats a long regex.
- When you hit a dead end, say so and pick the next pivot. Silence reads as being lost.

---

## Command kit

Field numbers depend on the log. Run `head` first and adjust. Every command here was run against sample logs in these formats before it went in.

**Orient and baseline**

```bash
ls -la; wc -l *
head -3 auth.log proxy.log wiki.log
awk '{print $5, $6}' auth.log | sort | uniq -c | sort -rn          # event types, e.g. "Accepted publickey"
cut -d, -f2 wiki.log | sort | uniq -c | sort -rn | head               # users in a CSV log
```

**Auth logs: brute force, success after failures, spray, junk lines**

```bash
grep 'Failed password' auth.log | awk '{print $(NF-2)}' | sort | uniq -c | sort -rn | head   # failures by source IP
grep '185.220.101.47' auth.log | grep Accepted                                              # did the noisy IP get in?
grep 'Failed password' auth.log | awk '{print $(NF-2), $(NF-4)}' | sort -u | awk '{print $1}' | uniq -c | sort -rn | head -3   # distinct users per IP = spray
grep 'Accepted password' auth.log                      # odd when everyone else uses keys
grep -v -e 'Failed password' -e 'Accepted' auth.log    # lines that match no known pattern
grep '185.220.101.47' auth.log | cut -c1-16 | uniq -c  # attempts per minute
```

**Proxy logs: exfiltration and beaconing**

```bash
awk '{out[$3" "$6]+=$7} END {for (k in out) print out[k], k}' proxy.log | sort -rn | head -5   # bytes out by user+host
awk '$5=="POST"' proxy.log | head                                                            # uploads only
awk '{print $6, $3}' proxy.log | sort -u | awk '{print $1}' | uniq -c | awk '$1==1'          # hosts only one user ever visits
grep 'cdn-update-check.net' proxy.log | awk '{split($2,t,":"); s=t[1]*3600+t[2]*60+t[3]; if (p) print s-p; p=s}' | sort | uniq -c   # same gap every time = beacon
```

**Endpoint process logs (one JSON object per line)**

```bash
grep -o '"image": "[^"]*"' process.jsonl | sort | uniq -c | sort -n | head   # rarest binaries
grep -i -e 'AppData' -e '\\Temp\\' process.jsonl                            # run from user-writable folders
grep '"host": "ws-09"' process.jsonl | grep -v '"ppid": 1000,'                # unusual parents on one host
echo 'SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQA' | base64 -d | tr -d '\0'       # PowerShell -enc is UTF-16LE base64
jq -r 'select(.host=="ws-09") | [.ts,.pid,.ppid,.cmdline] | @tsv' process.jsonl   # if jq exists
```

**One timeline for one user across logs**

```bash
{ grep -h ' jdoe ' proxy.log auth.log; grep -h '"user": "jdoe"' process.jsonl | sed 's/^{"ts": "\([^"]*\)", /\1 /'; } | sort | less
```

**Walk a process tree upwards by hand** (when there's no jq)

```bash
p=7004; while [ -n "$p" ] && [ "$p" != 1000 ]; do l=$(grep "\"pid\": $p," process.jsonl); echo "$l" | grep -o '"pid": [0-9]*, "ppid": [0-9]*, "image": "[^"]*"'; p=$(echo "$l" | grep -o '"ppid": [0-9]*' | grep -o '[0-9]*'); done
```

---

## What "the attack" usually is

The ACME scenario and the public reports point at a small set of plots. Know the tell for each.

| Plot | The tell | Where it shows |
| --- | --- | --- |
| Phishing to malware | Outlook or a browser spawns something from Temp or AppData, then PowerShell with `-enc`, then a new binary, then persistence (`schtasks`, Run key, a service) | Process log |
| Command and control | One host hitting one odd domain at a fixed interval, tiny bytes each way | Proxy or DNS log |
| Brute force or spray | Many failures then one success; or one IP, many users, one try each | Auth log |
| Insider data theft | Bulk reads of sensitive pages out of hours, then a large upload to a paste or file-sharing site | Wiki or file-share log, then proxy |
| Lateral movement | One account logging into many hosts in minutes; network logons using NTLM where Kerberos is normal | Auth and Windows security logs |
| Credential theft | `lsass` access, Mimikatz-style command lines, DCSync replication from a non-DC | Process and Windows logs |

Red herrings they plant: a legitimate user reading a few sensitive pages during work hours, a handful of fat-finger login failures from internal IPs, a malformed log line, a scanner from an internal security host.

---

## Write-up and CEO briefing

Part 2 of the online assessment and the end of the live round both want a write-up. The Dec 2025 candidate was asked for a briefing for the CEO, so practise both the technical note and the executive one.

**Technical note** (five lines, lead with the conclusion):

```text
Finding:    <one line, with severity>
Evidence:   <the two or three log lines or counts that prove it>
Scope:      <hosts, users, time window, data touched>
Do now:     <contain, preserve evidence, who to tell>
Check next: <what you didn't have time to confirm>
```

**CEO briefing** (no jargon, under 120 words):

```text
What happened:   On <date>, an attacker got into <what> through <how, in plain words>.
What they got:   <data or access, and what we don't yet know>.
What we did:     <contained X, reset Y, preserved evidence>.
Risk now:        <low / medium / high, and why>.
Next 48 hours:   <three actions, with owners>.
What we need:    <a decision, budget, or a call to a regulator or client>.
```

Say the uncertainty out loud. "I'm confident about A. I think B but couldn't prove it in the time. C is a guess." That line is worth more than a fourth finding.

---

## Verbal drills

For each, give your first three moves in under a minute.

| Scenario | First three moves |
| --- | --- |
| EDR flags malware on a trader's laptop at 14:00, markets open | Isolate from the network but keep it powered on so memory survives. Get the trader a clean machine so trading isn't blocked. Pull the process tree and every outbound connection from that host. |
| One host calls the same unknown domain every 60 seconds | Check how many hosts talk to it (one is worse than many). Find the process making the call. Look at what ran on that host just before the first call. |
| A developer pushed an AWS key to a public repo 20 minutes ago | Revoke the key first, investigate second; bots find these in minutes. Check CloudTrail for any use of that key. Scrub history and add a pre-commit secret scanner. |
| Someone read 30 strategy pages at 22:00 then uploaded 48 MB to a paste site | Preserve the logs before anyone touches anything. Check whether this matches their role and hours. Bring in HR and legal before you contact the person. |
| A memory dump from a possibly compromised box | Confirm the profile and OS build. List processes and compare with the host's normal set. Look for network connections, injected code, and anything with a deleted or unsigned backing file. |
| Here's a pcap | Protocol hierarchy first. Then conversations by bytes. Then the odd ones: DNS with long labels, TLS to an IP with no SNI, HTTP to a raw IP, a port that doesn't match its protocol. |
| A user forwards a suspicious email | Headers: From versus Reply-To versus Return-Path, SPF, DKIM, DMARC. Links: where they really go, domain age. Attachment: type, hash lookup, detonate in a sandbox. Verdict, then block and write a signature. |

[← Process](../01-process/README.md) · [Index](../README.md) · [Next: Coding →](../03-coding/README.md)
