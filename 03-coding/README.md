# 03 · Coding round

[← Log investigation](../02-log-investigation/README.md) · [Index](../README.md) · [Next: Security fundamentals →](../04-security-fundamentals/README.md)

One hour, your language, one practical problem that grows in parts. A London candidate was rejected in 2018 with a single line of feedback: the coding round wasn't up to scratch ([Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm)). This is the round to over-prepare. Every solution in this block passes its tests (25 tests across the four files), so you can trust it as a reference and rebuild it against a clock.

This block has four files:

| File | Covers |
| --- | --- |
| this page | What Jane Street asks, how to run the hour, the Jane Street problem in four parts, ten more tested tasks, the secure code-review drill, interviewer pushes, the drill sheet |
| [log-analysis-tasks.md](log-analysis-tasks.md) | Apache failures in a window, returning users, correlating two logs, impossible travel, sessions, decoding blobs, IOC extraction and directory walks, lateral-movement BFS, port-scan detection, SQL across log tables |
| [classic-tasks.md](classic-tasks.md) | Valid IPv4, binary to decimal, palindrome with punctuation, substring index, Two Sum, balanced brackets, word frequency, jq-lite, password generator, CSV average without pandas |
| [shell-one-liners.md](shell-one-liners.md) | The "script up a few things" phone round: tested bash, plus the Unix questions HRT and Jane Street ask |
| [know-your-language-python.md](know-your-language-python.md) | "Know your language" is a scored principle: the GIL, generators, decorators, context managers, dict hashing, the gotchas, and the unsafe APIs (`random`, `pickle`, `shell=True`), each with its security angle |

---

## What Jane Street has actually asked

| Task as reported | Build-up | Source |
| --- | --- | --- |
| Fill `def brute_force_detect(entries): pass`, where `entries` is an array of log-like strings | Detect brute force attacks | [Glassdoor HK](https://www.glassdoor.com.hk/Interview/Jane-Street-Interview-Questions-E255549.htm) |
| Given a log, track consecutive login attempts for one specific user | Part 2: now for all users (Cybersecurity Analyst, Dec 2025; rejected after this round) | [Glassdoor](https://static.glassdoor.ch/Interview/Jane-Street-Interview-E255549-RVW99892167.htm) |
| Get the list of events with failed logins, then decide whether to alert | Then add users, then timestamps: alert on N failures per user inside a window. Done live in a shared document, discussed rather than run (Security Engineer, Hong Kong, 2023) | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm) |
| Event-log parsing about failed logins, with two engineers in the room | DevOps Engineer, Oct 2025: the same problem family given to an adjacent team | [Glassdoor](https://www.glassdoor.ca/Interview/Jane-Street-Interview-E255549-RVW101486068.htm) |
| Two coding rounds in one loop (video "coding" and onsite "more coding"), alongside a Linux log review | SOC Analyst, Hong Kong, Apr 2026 | [Glassdoor](https://www.glassdoor.co.uk/Interview/Jane-Street-Interview-E255549-RVW103405660.htm) |
| "Script up a few things" in a language of your choice (phone round) | Cyber Security Engineer, London, 2018 | [Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm) |
| Search logs on a VM to find the source of an issue; a LeetCode-like problem with a twist after the first answer | Production Engineer, NY (the neighbouring team) | [Glassdoor](https://www.glassdoor.ie/Interview/Jane-Street-Interview-E255549-RVW82145680.htm) |

Five independent reports, one theme: failed logins in a log, growing from one user to all users to a time window to an alert decision. Expect it to be run twice (a screen and an onsite), and expect at least one version where you can't execute the code, so you desk-check it out loud. Jane Street's own guidance: real code in a real language, know your language's basic data structures, no bonus points for OCaml, open-ended problems with several reasonable answers, and the journey matters more than the snapshot ([Jane Street](https://www.janestreet.com/preparing-for-a-software-engineering-interview/)).

## What the same round looks like elsewhere

| Task | Where | Source |
| --- | --- | --- |
| Regex a log file for dates and IPs; walk a directory tree and check file flags; implement a Caesar cipher. "Sec engineer coding bar is LeetCode easy, focus on string manipulation." | Security engineer loops | [Blind](https://www.teamblind.com/post/Coding-interviews-for-security-engineering-roles-R7opjq8H) |
| Return true if a string is a valid RFC 1918 address. The interviewer wanted the IP treated as a 32-bit integer, not if/else on octets. | Security engineer | [Blind](https://www.teamblind.com/post/interview-question-seeking-alternate-answers-lna8pq03) |
| Parse a log of system events and flag malicious IPs by access frequency; a rate limiter keyed by user ID and IP | Snap Security Engineer | [Dataford](https://dataford.io/interview-guides/snap/security-engineer) |
| Millions of log lines: find the top 10 IPs causing errors; scan an AWS account for public S3 buckets | Instacart Security Engineer | [Dataford](https://dataford.io/interview-guides/instacart/security-engineer) |
| IP range to CIDR (LeetCode 751); a first-match ALLOW/DENY firewall over IPs and CIDRs | Databricks | [Hello Interview](https://www.hellointerview.com/community/questions/ip-cidr-conversion/cm5eguhac02l9838ou1owbtje) |
| Build the `cd` command (path normalisation); two LeetCode easy-to-medium questions in 15 minutes each; one coding plus one threat-modelling screen | Meta Security Engineer | [Glassdoor](https://www.glassdoor.com.au/Interview/Meta-Interview-E40772-RVW68446978.htm), [Glassdoor](https://fr.glassdoor.ch/Entretien/Meta-Entretien-E40772-RVW56815335.htm) |
| Two Sum; "how would you create a botnet"; core security questions plus one coding problem in the phone screen | Google Security Engineer | [Glassdoor](https://www.glassdoor.com.au/Interview/Google-Interview-E9079-RVW36658699.htm) |
| Parsing log files for patterns, basic ciphers, SQL queries | Cyber Security Engineer | [Glassdoor](https://static.glassdoor.de/Interview/cyber-security-engineer-interview-questions-SRCH_KO0,23_IP283.htm) |
| Merge intervals, then design and implement a rate limiter | Patreon phone screen | [Blind](https://www.teamblind.com/post/patreoncom-phone-screen-wtf-ru7qwjbw) |
| Secure code review: find the vulnerabilities in this code, why they exist, how to fix them (Amazon does this instead of coding) | Amazon Security Engineer | [Glassdoor](https://www.glassdoor.ca/Interview/Amazon-Interview-E6036-RVW92367615.htm) |
| Security-themed practice list: ciphers, parse arbitrary logs, web scraper, port scanner, password brute-forcer, PDF metadata, recover deleted items, YARA-style signatures | Ex-Google security engineer's study notes | [gracenolan](https://github.com/gracenolan/Notes/blob/master/interview-study-notes-for-security-engineering.md) |
| From Apache access logs, extract IPs that failed 10 times in a 5-minute window; about half of Google's detection-and-response loop is coding, usually one coding question mixed with an open-ended scenario per round | Google Detection and Response | [Blind](https://www.teamblind.com/post/Security-Engineer-Interview-at-Google-a37HOvBx) |
| Parse timestamps and IPs from Apache logs; correlate two logs with a hashmap of lists; count word frequency; make API calls with `requests` and write tests; Caesar cipher; LC-easy string manipulation; LC-medium string problems with a stack | 8.5 security on-sites across Google, Meta, Amazon, Datadog, Snap, Uber | [Blind](https://www.teamblind.com/post/i-did-85-security-engineer-on-sites-with-top-tech-companiesa-prep-guide-lyanpve6) |
| Screening round is log parsing and string splitting; palindrome where "Taco, cat" counts; expected themes: parse logs and build logic on top, defang IPs, enrich a dataset; two LC-easy in 45 minutes each at onsite; BFS/DFS fair game, unit tests may matter | Meta Security Engineer (detection team) | [Blind](https://www.teamblind.com/post/security-engineer-interview-meta-mei6qc2r), [Blind](https://www.teamblind.com/post/facebook-meta-security-engineer-phone-screen-coming-up-1tjiynwi) |
| Decode a base64-encoded, compressed log and count an IP; build a password generator; average a field over 30 years from a 20-field CSV without pandas | Security engineer loops reported on Reddit | [r/cybersecurity](https://reddit.sentinel-team.org/posts/1re3tmb/snapshots/2026-02-25T23%3A00%3A22.500174Z), [r/cybersecurity](https://reddit.sentinel-team.org/posts/1quuw5n/snapshots/2026-02-03T22%3A40%3A54.044863Z) |
| Find returning users from access logs; evaluate IP access rules; most frequent user in a log; parse and summarise HTTP access logs | Amazon and Apple (candidate-reported via prep site) | [prachub](https://prachub.com/resources/amazon-security-engineer-new-grad-interview-security-scenarios-and-preparation) |
| Regex IOCs out of a log, then recurse over a directory tree or download the logs first; parse JSON and extract elements jq-style; three scripting rounds at Google, scripting not LeetCode | Blind megathread | [Blind](https://www.teamblind.com/post/security-engineer-interview-resources-megathread-fp1izvtl) |
| Code screens: parse logs, call APIs, automate, scale ingestion, SQL joins across log tables; graded on completeness, edge cases, error handling and tests | Detection-engineering loops | [Julie Sparks](https://medium.com/@julieasparks/acing-your-detection-engineering-interview-9ed95f70ffd7) |
| Validate an IP address format; binary to decimal; graph traversal; "easy mediums" on strings, file handling, dicts and sets | Facebook AppSec and Netskope infra security | [Blind](https://www.teamblind.com/post/facebook-application-security-engineer-interview-rcbrbqfj) |

---

## How to run the hour

1. **Ask five questions before you type** (about 60 seconds): What does a line look like, can I see three? Is it sorted by time? Can lines be malformed? What counts as brute force: N in a row, N in a window, or both? What do I return?
2. **Say the plan in two sentences.** "I'll parse each line into a small tuple, then keep a counter per user. Parse errors get skipped and counted."
3. **Write the simplest version that works**, run it on three lines you make up, then grow it.
4. **Test out loud** before you say done: the happy path, a malformed line, a success between failures. Write the test cases as comments if you can't run code.
5. **Name the complexity and the next step** without being asked.
6. **If it's a shared document with no interpreter** (the 2023 Hong Kong loop), desk-check: pick three lines, walk the state variables through them by hand, out loud. That's what they're watching for.

The grading criteria one detection-engineering interviewer published: completeness, edge cases, error handling, tests ([Julie Sparks](https://medium.com/@julieasparks/acing-your-detection-engineering-interview-9ed95f70ffd7)). Jane Street adds clarity and the questions you ask.

---

## The Jane Street problem, built up in four parts

Assumed line format (say your assumption, then confirm it):

```text
2026-11-02 02:01:09 FAIL user=svc_backup ip=185.220.101.47
2026-11-02 02:11:56 OK   user=svc_backup ip=185.220.101.47
```

```python
from collections import defaultdict, deque
from datetime import datetime, timedelta


def parse(line):
    """(ts, ok, user, ip) or None for a line we can't read. Never crash on bad input."""
    parts = line.split()
    if len(parts) < 4:
        return None
    date, time, result, *rest = parts
    try:
        ts = datetime.fromisoformat(f"{date} {time}")
    except ValueError:
        return None
    fields = dict(p.split("=", 1) for p in rest if "=" in p)
    if result not in ("OK", "FAIL") or "user" not in fields:
        return None
    return ts, result == "OK", fields["user"], fields.get("ip")


# Part 1: longest run of consecutive failures for one user
def max_consecutive_failures(entries, user):
    best = run = 0
    for line in entries:
        e = parse(line)
        if e is None or e[2] != user:
            continue
        run = 0 if e[1] else run + 1
        best = max(best, run)
    return best


# Part 2: every user, alert once per streak, plus the alert that matters most
def brute_force_detect(entries, threshold=5):
    run = defaultdict(int)
    alerts = []
    for line in entries:
        e = parse(line)
        if e is None:
            continue
        ts, ok, user, ip = e
        if ok:
            if run[user] >= threshold:
                alerts.append(("SUCCESS_AFTER_FAILURES", user, ip, ts, run[user]))
            run[user] = 0
        else:
            run[user] += 1
            if run[user] == threshold:          # once per streak, not on every line
                alerts.append(("BRUTE_FORCE", user, ip, ts, run[user]))
    return alerts


# Part 3: N failures inside a time window. Beats an attacker who lets the real user log in between.
def window_detect(entries, threshold=5, window=timedelta(minutes=5)):
    recent, flagged = defaultdict(deque), {}
    for line in entries:
        e = parse(line)
        if e is None or e[1]:
            continue
        ts, _, user, _ = e
        q = recent[user]
        q.append(ts)
        while q and ts - q[0] > window:
            q.popleft()
        if len(q) >= threshold and user not in flagged:
            flagged[user] = ts
    return flagged


# Part 4: password spray. One source, many different users, few tries each.
def spray_detect(entries, min_users=10, window=timedelta(minutes=10)):
    recent, flagged = defaultdict(deque), {}
    for line in entries:
        e = parse(line)
        if e is None or e[1] or e[3] is None:
            continue
        ts, _, user, ip = e
        q = recent[ip]
        q.append((ts, user))
        while q and ts - q[0][0] > window:
            q.popleft()
        users = {u for _, u in q}
        if len(users) >= min_users and ip not in flagged:
            flagged[ip] = (ts, len(users))
    return flagged
```

Follow-ups they add, with the one-line answer:

| Push | Answer |
| --- | --- |
| Complexity? | Part 2 is O(n) time, O(users) memory. Part 3 can cap memory with `deque(maxlen=threshold)` and check `ts - q[0] <= window` once it's full. |
| Lines out of order? | Sort first if it's a file. If it's a stream, buffer a few seconds of lateness and process in timestamp order. |
| Attacker spreads across many IPs? | Count per user regardless of source. That's why Part 2 keys on user, and Part 4 keys on IP for the opposite case. |
| False positives? | A service account with a stale password fails forever. Allowlist known noisy pairs, but still alert on success after failures. |
| Scale to millions of lines? | Make it a generator over a stream, evict users idle longer than the window, never hold the file in memory. |
| Username variants? | Lowercase, strip `DOMAIN\`, treat "invalid user" lines as failures. |

---

## Other tasks, with tested solutions

**Top N IPs causing errors in a huge access log.** One pass, a `Counter`, never load the file.

```python
import re
from collections import Counter

LOG_RE = re.compile(r'^(?P<ip>\d{1,3}(?:\.\d{1,3}){3}) .*?"(?P<method>[A-Z]+) (?P<path>\S+)[^"]*" (?P<status>\d{3})')

def top_failed_ips(lines, n=10):
    counts = Counter()
    for line in lines:                       # lines can be an open file: streams, no memory blow-up
        m = LOG_RE.match(line)
        if m and m.group("status") in ("401", "403"):
            counts[m.group("ip")] += 1
    return counts.most_common(n)
```

Push: "the file doesn't fit on one machine" → shard by IP hash, count per shard, merge the top-N lists. "Top N exactly vs approximately" → exact needs all keys; a count-min sketch gives approximate at fixed memory.

**IP as a 32-bit integer.** The Blind interviewer wanted exactly this.

```python
import ipaddress

def ip_to_int(ip):
    octets = ip.split(".")
    if len(octets) != 4:
        raise ValueError(ip)
    value = 0
    for o in octets:
        n = int(o)
        if not 0 <= n <= 255:
            raise ValueError(ip)
        value = (value << 8) | n
    return value

PRIVATE = [("10.0.0.0", 8), ("172.16.0.0", 12), ("192.168.0.0", 16)]

def in_cidr(ip, network, prefix):
    mask = (0xFFFFFFFF << (32 - prefix)) & 0xFFFFFFFF
    return ip_to_int(ip) & mask == ip_to_int(network) & mask

def is_rfc1918(ip):
    try:
        return any(in_cidr(ip, net, p) for net, p in PRIVATE)
    except ValueError:
        return False

def is_rfc1918_stdlib(ip):                   # say this exists, then show you can do it by hand
    try:
        return ipaddress.ip_address(ip).is_private
    except ValueError:
        return False
```

**IP range to CIDR blocks** (LeetCode 751). Greedy: the biggest aligned block that fits.

```python
def int_to_ip(n):
    return ".".join(str((n >> shift) & 255) for shift in (24, 16, 8, 0))

def ip_to_cidr(start, n):
    cur, out = ip_to_int(start), []
    while n > 0:
        low_bit = cur & -cur or 1 << 32      # alignment of cur: largest block that can start here
        size = 1
        while size * 2 <= n and size * 2 <= low_bit:
            size *= 2
        out.append(f"{int_to_ip(cur)}/{32 - size.bit_length() + 1}")
        cur += size
        n -= size
    return out
# ip_to_cidr("255.0.0.7", 10) -> ["255.0.0.7/32", "255.0.0.8/29", "255.0.0.16/32"]
```

**First-match firewall.** Ordered rules, first match wins, default deny.

```python
def firewall_allows(rules, ip):
    target = ip_to_int(ip)
    for action, spec in rules:               # [("DENY", "10.0.0.5"), ("ALLOW", "10.0.0.0/8"), ...]
        net, _, prefix = spec.partition("/")
        prefix = int(prefix) if prefix else 32
        mask = (0xFFFFFFFF << (32 - prefix)) & 0xFFFFFFFF
        if target & mask == ip_to_int(net) & mask:
            return action == "ALLOW"
    return False
```

Push: "thousands of rules, millions of lookups" → a radix trie keyed on prefix bits, or precompute non-overlapping ranges and binary-search.

**Sliding-window rate limiter keyed by user and IP** (Snap, Patreon).

```python
from collections import defaultdict, deque

class RateLimiter:
    def __init__(self, limit, window_seconds):
        self.limit, self.window = limit, window_seconds
        self.hits = defaultdict(deque)

    def allow(self, user, ip, now):
        keys = (("user", user), ("ip", ip))
        for key in keys:                      # check both before counting the hit
            q = self.hits[key]
            while q and now - q[0] >= self.window:
                q.popleft()
            if len(q) >= self.limit:
                return False
        for key in keys:
            self.hits[key].append(now)
        return True
```

Push: "many API servers" → shared state in Redis, atomic `INCR` plus `EXPIRE` or a Lua script; token bucket if you want bursts. "Memory" → evict idle keys; a fixed-window counter uses O(1) per key at the cost of edge bursts.

**Build `cd`** (Meta). It's path normalisation, and it's the same logic that stops directory traversal.

```python
def normalise_path(cwd, target):
    path = target if target.startswith("/") else f"{cwd}/{target}"
    out = []
    for part in path.split("/"):
        if part in ("", "."):
            continue
        if part == "..":
            if out:
                out.pop()
        else:
            out.append(part)
    return "/" + "/".join(out)
# normalise_path("/home/oleh", "../etc/./passwd") -> "/home/etc/passwd"
```

Push: "now make a safe file download" → normalise, then require the result to start with the allowed base directory, and resolve symlinks with `os.path.realpath` before the check.

**Caesar cipher and extraction regexes** (Blind, Glassdoor).

```python
import re

def caesar(text, shift):
    out = []
    for ch in text:
        if ch.isalpha():
            base = ord("A") if ch.isupper() else ord("a")
            out.append(chr((ord(ch) - base + shift) % 26 + base))
        else:
            out.append(ch)
    return "".join(out)

IPV4_RE = re.compile(r"\b(?:(?:25[0-5]|2[0-4]\d|1?\d?\d)\.){3}(?:25[0-5]|2[0-4]\d|1?\d?\d)\b")
DATE_RE = re.compile(r"\b\d{4}-\d{2}-\d{2}(?:[ T]\d{2}:\d{2}:\d{2})?\b")

def extract(lines):
    ips, dates = set(), set()
    for line in lines:
        ips.update(IPV4_RE.findall(line))
        dates.update(DATE_RE.findall(line))
    return sorted(ips), sorted(dates)
```

Say it: a cipher question is about clean loops and modular arithmetic, not cryptography. If they ask "is this secure?", the answer is "no, 26 keys, and letter frequency breaks it in seconds."

**Walk a directory tree and flag risky files** (Blind).

```python
import os, stat

def risky_files(root):
    found = []
    for dirpath, _, files in os.walk(root):
        for name in files:
            p = os.path.join(dirpath, name)
            try:
                st = os.lstat(p)                 # lstat: don't follow symlinks
            except OSError:
                continue
            if stat.S_ISLNK(st.st_mode):
                continue
            if st.st_mode & stat.S_IWOTH:
                found.append(("world-writable", p))
            if st.st_mode & stat.S_ISUID:
                found.append(("setuid", p))
    return found
```

**Beaconing in proxy logs.** Group by (user, host), look at the gaps.

```python
from collections import defaultdict
from datetime import datetime
from statistics import mean, pstdev

def find_beacons(lines, min_hits=10, max_jitter=0.1):
    seen = defaultdict(list)                 # 'ts ts user src METHOD host out in'
    for line in lines:
        parts = line.split()
        if len(parts) < 6:
            continue
        try:
            ts = datetime.fromisoformat(f"{parts[0]} {parts[1]}")
        except ValueError:
            continue
        seen[(parts[2], parts[5])].append(ts)
    hits = []
    for key, times in seen.items():
        if len(times) < min_hits:
            continue
        times.sort()
        gaps = [(b - a).total_seconds() for a, b in zip(times, times[1:])]
        avg = mean(gaps)
        if avg > 0 and pstdev(gaps) / avg <= max_jitter:
            hits.append((key, len(times), round(avg)))
    return hits
```

Push: "real malware adds jitter" → raise the tolerance, or test whether gaps cluster around a few values; also rank by low bytes and high regularity together.

**Merge intervals** (Patreon). Sort, then extend or append.

```python
def merge_intervals(intervals):
    out = []
    for start, end in sorted(intervals):
        if out and start <= out[-1][1]:
            out[-1][1] = max(out[-1][1], end)
        else:
            out.append([start, end])
    return out
```

---

## Secure code review drill (Amazon style, also useful if Jane Street hands you code)

Find every problem in this snippet in five minutes, then say the fix for each.

```python
import hashlib, os, pickle, sqlite3
from flask import Flask, request, send_file

app = Flask(__name__)
SECRET = "sk_live_9f8e7d6c"
db = sqlite3.connect("users.db", check_same_thread=False)

@app.route("/login", methods=["POST"])
def login():
    u, p = request.form["user"], request.form["password"]
    h = hashlib.md5(p.encode()).hexdigest()
    row = db.execute(f"SELECT id FROM users WHERE name='{u}' AND pw='{h}'").fetchone()
    return "ok" if row else ("bad password for " + u, 401)

@app.route("/export")
def export():
    name = request.args.get("file")
    return send_file("/srv/exports/" + name)

@app.route("/ping")
def ping():
    host = request.args.get("host")
    return os.popen("ping -c 1 " + host).read()

@app.route("/import", methods=["POST"])
def load():
    return str(pickle.loads(request.data))

@app.route("/admin/users")
def users():
    return str(db.execute("SELECT name, pw FROM users").fetchall())

app.run(debug=True, host="0.0.0.0")
```

<details>
<summary>The nine problems and fixes</summary>

1. **SQL injection** in `login`: string-built query. Fix: parameterised query `execute("... WHERE name=? AND pw=?", (u, h))`.
2. **Unsalted MD5 for passwords.** Fix: argon2id or bcrypt with per-user salt; compare in constant time.
3. **Username reflected in the error** and different messages for bad user versus bad password: enumeration. Fix: one generic message, same timing.
4. **Path traversal** in `export`: `file=../../etc/passwd`. Fix: normalise, `realpath`, require the result to start with `/srv/exports/`, or map an ID to a filename.
5. **Command injection** in `ping`: `host=;id`. Fix: `subprocess.run(["ping","-c","1",host])` with no shell, after validating `host` as an IP or hostname.
6. **Insecure deserialisation**: `pickle.loads` on request data is remote code execution. Fix: JSON with a schema.
7. **No authentication or authorisation** on `/admin/users`, and it returns password hashes. Fix: auth middleware, role check, never return hashes.
8. **Hardcoded secret** in source. Fix: environment or a secrets manager, rotate the leaked one.
9. **`debug=True` on `0.0.0.0`**: the Werkzeug debugger gives a remote shell. Fix: never in production; bind to localhost behind a proxy.

Bonus: no rate limiting on login, no CSRF protection, no TLS enforced, a shared SQLite connection across threads.
</details>

How to talk through a review: start with the entry points (every `request.*`), follow each input to where it's used (query, shell, filesystem, deserialiser, response), then check auth on every route, then secrets and config. Say the severity as you go.

---

## Know these cold in Python

`Counter`, `defaultdict`, `deque`, `datetime.fromisoformat` and `strptime` with `%z`, `timedelta`, `re` with named groups, `json.loads` per line, `csv.DictReader`, `sorted(key=...)`, generators for streaming files, set operations, `ipaddress`, `os.walk`, `base64`, `gzip`, `sqlite3`, `secrets`, `subprocess.run` without `shell=True`, `hashlib`, `hmac.compare_digest`. For BFS: `deque` and a `seen` set. For "write tests": a `test_` function with three asserts is enough; say you'd use `pytest`.

## What interviewers push on, and the one-line answer

| Push | Answer |
| --- | --- |
| "Can you do it in one pass?" | Yes: stream the lines, keep per-key state in a dict, never store the file. |
| "What's the complexity?" | O(n) time, O(keys) memory; say which structure holds the memory. |
| "The input is a stream, not a file." | Generator over the stream; evict idle keys; buffer a few seconds for late lines. |
| "Lines are out of order." | Sort if it fits; otherwise a bounded reorder buffer keyed on timestamp. |
| "A line is malformed." | Skip it, count it, log a sample; never crash the whole run on one line. |
| "Usernames vary in case or have a domain prefix." | Normalise before counting: lowercase, strip `DOMAIN\`. |
| "The attacker spreads across IPs" / "across users." | Key on the thing that's constant: user for brute force, IP for spray. |
| "False positives?" | Allowlist known noisy pairs; keep the success-after-failures alert even for them. |
| "How would you test this?" | Three cases on the spot: happy path, malformed line, the boundary (exactly N, exactly the window). |
| "Make it faster." | Compile the regex once; avoid per-line `datetime` parsing when a string prefix compare will do; batch I/O. |
| "Now do it in SQL." | `GROUP BY user, window` with `HAVING COUNT(*) >= N`; normalise timestamps first. |
| "What would you log or alert?" | The alert record with user, IP, first and last timestamp, count, and the evidence lines. |

## Timed drill sheet

Six sessions, 60 to 90 minutes each. Blank file, timer running, talking out loud the whole time. Tick when you finish inside the time with tests passing.

- [ ] **Drill 1 (45 min):** the Jane Street problem, all four parts, from memory. Then desk-check Part 3 by hand on five lines without running it.
- [ ] **Drill 2 (3 × 20 min):** `parse_apache` and `failed_ips_in_window`; `correlate` and `impossible_travel`; `decode_b64_gzip` and `count_ips`.
- [ ] **Drill 3 (3 × 20 min):** `extract_iocs` then `walk_and_extract`; `reachable_hosts`; `port_scan_sources`.
- [ ] **Drill 4 (3 × 15 min):** `ip_to_int` and `is_rfc1918`; `ip_to_cidr`; `firewall_allows` then `RateLimiter`.
- [ ] **Drill 5 (6 × 8 min):** `is_valid_ipv4`, `is_palindrome`, `balanced`, `two_sum`, `jget`, `generate_password`.
- [ ] **Drill 6 (45 min):** the secure code-review drill, all nine findings with fixes in 5 minutes; then `users_with_failures_then_success` in SQL; then 15 minutes of bash one-liners against sample logs from memory.

Then invent one new variant for each family (a different log format, a different window rule, a different key) and solve it cold. The interview question will be a variant, not a copy.

[← Log investigation](../02-log-investigation/README.md) · [Index](../README.md) · [Next: Security fundamentals →](../04-security-fundamentals/README.md)
