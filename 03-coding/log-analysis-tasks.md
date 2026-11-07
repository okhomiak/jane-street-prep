# 03b · Log-analysis coding tasks

[← Coding overview](README.md) · [Classic tasks](classic-tasks.md) · [Shell one-liners](shell-one-liners.md) · [Code review](README.md#secure-code-review-drill-amazon-style-also-useful-if-jane-street-hands-you-code)

Every task here was reported by a candidate, and every function passes its tests (`11 passed` in the stage 2 suite). Practise each from a blank file against a 20-minute timer, then invent a variant.

Sources: [Google D&R thread on Blind](https://www.teamblind.com/post/Security-Engineer-Interview-at-Google-a37HOvBx) (Apache failures in a window), [8.5 security on-sites write-up](https://www.teamblind.com/post/i-did-85-security-engineer-on-sites-with-top-tech-companiesa-prep-guide-lyanpve6) (parse timestamps and IPs from Apache logs, correlate two logs with a hashmap of lists), [prachub Amazon report](https://prachub.com/resources/amazon-security-engineer-new-grad-interview-security-scenarios-and-preparation) (returning users, most frequent user, IP access rules), [r/cybersecurity](https://reddit.sentinel-team.org/posts/1re3tmb/snapshots/2026-02-25T23%3A00%3A22.500174Z) (decode a base64 gzip log and count an IP), [Blind megathread](https://www.teamblind.com/post/security-engineer-interview-resources-megathread-fp1izvtl) (IOC regex then recurse over a directory), [Meta detection thread](https://www.teamblind.com/post/security-engineer-interview-meta-mei6qc2r) (defang IPs, enrich a dataset), [Julie Sparks on detection-engineering interviews](https://medium.com/@julieasparks/acing-your-detection-engineering-interview-9ed95f70ffd7) (SQL joins across log tables, graded on completeness, edge cases, error handling, tests), [gracenolan](https://github.com/gracenolan/Notes/blob/master/interview-study-notes-for-security-engineering.md) (detect port scanning).

---

## Apache access logs: parse, then find IPs with 10 failures in 5 minutes

A Google detection-and-response engineer gave this as a representative coding question. It's the Jane Street brute-force problem wearing a web-log costume.

```python
import re
from collections import Counter, defaultdict, deque
from datetime import datetime, timedelta

APACHE_RE = re.compile(
    r'^(?P<ip>\S+) \S+ (?P<user>\S+) \[(?P<ts>[^\]]+)\] "(?P<method>[A-Z]+) (?P<path>\S+)[^"]*" '
    r'(?P<status>\d{3}) (?P<bytes>\d+|-)'
)

def parse_apache(line):
    """Return a dict or None. Timestamp like 02/Nov/2026:10:00:00 +0000."""
    m = APACHE_RE.match(line)
    if not m:
        return None
    d = m.groupdict()
    try:
        d["ts"] = datetime.strptime(d["ts"], "%d/%b/%Y:%H:%M:%S %z")
    except ValueError:
        return None
    d["status"] = int(d["status"])
    d["bytes"] = 0 if d["bytes"] == "-" else int(d["bytes"])
    return d

def failed_ips_in_window(lines, threshold=10, window=timedelta(minutes=5), fail_statuses=(401, 403)):
    """IPs that produced >= threshold failures inside any window. One pass, a deque per IP."""
    recent, flagged = defaultdict(deque), {}
    for line in lines:
        e = parse_apache(line)
        if e is None or e["status"] not in fail_statuses:
            continue
        q = recent[e["ip"]]
        q.append(e["ts"])
        while q and e["ts"] - q[0] > window:
            q.popleft()
        if len(q) >= threshold and e["ip"] not in flagged:
            flagged[e["ip"]] = e["ts"]
    return flagged

def returning_users(lines):
    """Users seen on more than one distinct day, plus the most frequent user."""
    days = defaultdict(set)
    counts = Counter()
    for line in lines:
        e = parse_apache(line)
        if e is None or e["user"] == "-":
            continue
        days[e["user"]].add(e["ts"].date())
        counts[e["user"]] += 1
    returning = sorted(u for u, d in days.items() if len(d) > 1)
    most_frequent = counts.most_common(1)[0][0] if counts else None
    return returning, most_frequent
```

Say while you write: the regex is anchored and uses named groups so a malformed line returns `None` instead of crashing; `strptime` with `%z` keeps the offset; `lines` can be an open file so it streams. Pushes: "what counts as a failure" (status codes, or `/login` POSTs that got a 200 with an error body), "distributed attacker" (key on user or path instead of IP), "the log is 50 GB" (shard by IP hash, merge).

## Correlate two logs

The 8.5-onsites author was asked to join two logs; the expected answer is a hashmap of lists for O(1) lookup, not a nested loop.

```python
def correlate(log_a, log_b, key):
    """Join two lists of dicts on key with a hashmap of lists: O(n+m), not O(n*m)."""
    index = defaultdict(list)
    for rec in log_a:
        index[rec[key]].append(rec)
    joined = {}
    for rec in log_b:
        if rec[key] in index:
            joined.setdefault(rec[key], ([], []))
            joined[rec[key]][1].append(rec)
    for k in joined:
        joined[k] = (index[k], joined[k][1])
    return joined

def impossible_travel(logins, min_gap=timedelta(hours=2)):
    """logins: dicts with user, ts, country. Flag a user seen in two countries closer than min_gap."""
    by_user = defaultdict(list)
    for e in logins:
        by_user[e["user"]].append((e["ts"], e["country"]))
    flagged = []
    for user, events in by_user.items():
        events.sort()
        for (t1, c1), (t2, c2) in zip(events, events[1:]):
            if c1 != c2 and t2 - t1 < min_gap:
                flagged.append((user, c1, c2, t2))
    return flagged

def sessionise(events, gap=timedelta(minutes=30)):
    """Split per-user events into sessions when the gap exceeds `gap`. events: (user, ts)."""
    by_user = defaultdict(list)
    for user, ts in events:
        by_user[user].append(ts)
    sessions = {}
    for user, times in by_user.items():
        times.sort()
        out, cur = [], [times[0]]
        for t in times[1:]:
            if t - cur[-1] > gap:
                out.append((cur[0], cur[-1], len(cur)))
                cur = [t]
            else:
                cur.append(t)
        out.append((cur[0], cur[-1], len(cur)))
        sessions[user] = out
    return sessions
```

Pushes: VPNs and mobile carriers make "impossible travel" noisy, so say you'd compare against the user's usual countries first; sessionisation is how you turn raw events into "a login session from Singapore at 03:00".

## Decode an encoded blob, then count

A Reddit candidate was handed a base64-encoded, gzip-compressed log and asked to count an IP's occurrences. The PowerShell `-EncodedCommand` decode is the same idea and comes up in the live round.

```python
import base64, gzip

def decode_b64_gzip(blob):
    return gzip.decompress(base64.b64decode(blob)).decode("utf-8")

def count_ips(text):
    ipv4 = re.compile(r"\b(?:(?:25[0-5]|2[0-4]\d|1?\d?\d)\.){3}(?:25[0-5]|2[0-4]\d|1?\d?\d)\b")
    return Counter(ipv4.findall(text))

def decode_powershell_enc(b64):
    """PowerShell -EncodedCommand is UTF-16LE text, then base64."""
    return base64.b64decode(b64).decode("utf-16-le")
```

Shell equivalents: `base64 -d blob | gunzip | grep -o '10\.0\.0\.1' | wc -l` and `echo '<b64>' | base64 -d | iconv -f UTF-16LE -t UTF-8`.

## IOCs: extract, defang, walk a tree

Blind and Google reports: regex IOCs out of a log, then "now do it recursively over a directory" or "download the logs first". Meta: defang IPs and enrich a dataset.

```python
import os

def defang(s):
    return s.replace("http", "hxxp").replace(".", "[.]")

def refang(s):
    return s.replace("hxxp", "http").replace("[.]", ".")

IOC_PATTERNS = {
    "ipv4": re.compile(r"\b(?:(?:25[0-5]|2[0-4]\d|1?\d?\d)\.){3}(?:25[0-5]|2[0-4]\d|1?\d?\d)\b"),
    "sha256": re.compile(r"\b[a-fA-F0-9]{64}\b"),
    "sha1": re.compile(r"\b[a-fA-F0-9]{40}\b"),
    "md5": re.compile(r"\b[a-fA-F0-9]{32}\b"),
    "url": re.compile(r"\bhttps?://[^\s\"'<>]+"),
    "domain": re.compile(r"\b(?:[a-z0-9-]+\.)+(?:com|net|org|io|ru|cn|info|biz|xyz|top|uk|de)\b", re.I),
}

def extract_iocs(text):
    found = {k: set(p.findall(text)) for k, p in IOC_PATTERNS.items()}
    found["domain"] -= {u.split("/")[2] for u in found["url"] if u.count("/") >= 2}   # a URL's host isn't a second finding
    return {k: sorted(v) for k, v in found.items() if v}

def walk_and_extract(root, exts=(".log", ".txt")):
    merged = defaultdict(set)
    for dirpath, _, files in os.walk(root):
        for name in files:
            if name.endswith(exts):
                with open(os.path.join(dirpath, name), errors="replace") as fh:
                    for k, v in extract_iocs(fh.read()).items():
                        merged[k].update(v)
    return {k: sorted(v) for k, v in merged.items()}
```

Pushes: the TLD list is a shortcut, say so and offer the public suffix list; `errors="replace"` keeps a binary file from crashing the run; for "download first" use `requests.get(url, timeout=10, stream=True)` and write to disk before parsing. Enrichment means joining your IOC set against a reputation feed, which is `correlate` again.

## Lateral movement as a graph

Meta says BFS and DFS are fair game. The security version: from a connection log, what could an attacker on one host reach?

```python
def reachable_hosts(connections, start):
    """connections: (src, dst) pairs. BFS over the directed graph."""
    graph = defaultdict(set)
    for s, d in connections:
        graph[s].add(d)
    seen, queue = {start}, deque([start])
    while queue:
        cur = queue.popleft()
        for nxt in graph[cur]:
            if nxt not in seen:
                seen.add(nxt)
                queue.append(nxt)
    seen.discard(start)
    return seen
```

Push: "weighted by how hard each hop is" is Dijkstra, which Meta says you won't need; say it and move on. "Which single host, if isolated, cuts the attacker off from the domain controller" is an articulation-point question; answer it by removing candidates and re-running BFS.

## Port-scan detection from flow records

```python
def port_scan_sources(flows, min_ports=20, window=timedelta(minutes=1)):
    """flows: (ts, src, dst, dport). A source touching >= min_ports distinct ports on one host inside the window."""
    recent, flagged = defaultdict(deque), set()
    for ts, src, dst, dport in sorted(flows):
        q = recent[(src, dst)]
        q.append((ts, dport))
        while q and ts - q[0][0] > window:
            q.popleft()
        if len({p for _, p in q}) >= min_ports:
            flagged.add((src, dst))
    return flagged
```

Push: a horizontal scan (one port, many hosts) needs the key flipped to `(src, dport)`; slow scans need a longer window and lower threshold, which is where false positives come from.

## SQL across log tables

Detection-engineering screens include SQL joins across log tables. Build it in SQLite so you can run it.

```python
import sqlite3
from datetime import timezone

def users_with_failures_then_success(rows):
    """rows: (user, ts, ok). Who failed >= 3 times in the 10 minutes before a success."""
    con = sqlite3.connect(":memory:")
    con.execute("create table auth(user text, ts text, ok integer)")
    # Normalise to 'YYYY-MM-DD HH:MM:SS' UTC. ISO strings with 'T' or '+00:00' won't compare with datetime() output.
    con.executemany("insert into auth values (?,?,?)",
                    [(u, t.astimezone(timezone.utc).strftime("%Y-%m-%d %H:%M:%S"), int(ok)) for u, t, ok in rows])
    sql = """
    select s.user
    from auth s
    join auth f on f.user = s.user and f.ok = 0
         and f.ts between datetime(s.ts, '-10 minutes') and s.ts
    where s.ok = 1
    group by s.user, s.ts
    having count(f.ts) >= 3
    """
    return sorted({r[0] for r in con.execute(sql)})
```

The gotcha this test caught: timestamps stored as ISO text (`2026-11-02T10:00:00+00:00`) don't compare correctly with SQLite's `datetime()` output (`2026-11-02 10:50:00`), so an hour-old failure matched. Normalise on the way in. Say that in the interview and you've shown exactly the care they're scoring.

## Beacon detection

The proxy-log version is in the [overview](README.md#other-tasks-with-tested-solutions). The pattern: group by (host, destination), sort timestamps, take the gaps, flag low jitter with enough hits.

[← Coding overview](README.md) · [Classic tasks](classic-tasks.md) · [Shell one-liners](shell-one-liners.md)
