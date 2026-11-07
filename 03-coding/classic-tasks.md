# 03c · Classic tasks that keep getting reported

[← Coding overview](README.md) · [Log-analysis tasks](log-analysis-tasks.md) · [Shell one-liners](shell-one-liners.md)

Small, fast, and reported often enough that you should never lose a minute on them. Every function passes its tests.

Sources: [Meta detection thread](https://www.teamblind.com/post/security-engineer-interview-meta-mei6qc2r) (palindrome where "Taco, cat" counts), [Blind megathread](https://www.teamblind.com/post/security-engineer-interview-resources-megathread-fp1izvtl) (palindrome, substring index, Caesar cipher, jq-style JSON extraction), [Facebook AppSec thread](https://www.teamblind.com/post/facebook-application-security-engineer-interview-rcbrbqfj) (validate an IP address format, binary to decimal, graph traversal), [Google report](https://www.glassdoor.com.au/Interview/Google-Interview-E9079-RVW36658699.htm) (Two Sum), [8.5 on-sites write-up](https://www.teamblind.com/post/i-did-85-security-engineer-on-sites-with-top-tech-companiesa-prep-guide-lyanpve6) (word frequency, LC-medium string problems with a stack), [r/cybersecurity](https://reddit.sentinel-team.org/posts/1quuw5n/snapshots/2026-02-03T22%3A40%3A54.044863Z) (password generator; CSV average without pandas).

```python
import csv, io, re, secrets, string
from collections import Counter

def is_valid_ipv4(s):
    parts = s.split(".")
    return len(parts) == 4 and all(p.isdigit() and 0 <= int(p) <= 255 and (p == "0" or not p.startswith("0")) for p in parts)

def binary_to_decimal(s):
    if not s or any(c not in "01" for c in s):
        raise ValueError(s)
    value = 0
    for c in s:
        value = value * 2 + (c == "1")
    return value

def is_palindrome(s):
    """'Taco, cat' counts: compare only alphanumerics, case-insensitive."""
    cleaned = [c.lower() for c in s if c.isalnum()]
    return cleaned == cleaned[::-1]

def substring_index(hay, needle):
    """First index of needle in hay without str.find; -1 if absent."""
    if needle == "":
        return 0
    for i in range(len(hay) - len(needle) + 1):
        if hay[i:i + len(needle)] == needle:
            return i
    return -1

def two_sum(nums, target):
    seen = {}
    for i, n in enumerate(nums):
        if target - n in seen:
            return seen[target - n], i
        seen[n] = i
    return None

def balanced(s):
    """Stack-based bracket matching, the usual 'LC medium with a stack' shape."""
    pairs, stack = {")": "(", "]": "[", "}": "{"}, []
    for ch in s:
        if ch in "([{":
            stack.append(ch)
        elif ch in pairs:
            if not stack or stack.pop() != pairs[ch]:
                return False
    return not stack

def word_frequency(text, n=5):
    words = re.findall(r"[a-z0-9']+", text.lower())
    return Counter(words).most_common(n)

def jget(obj, path):
    """jq-lite: 'a.b[0].c' -> value or None."""
    for part in re.findall(r"[^.\[\]]+|\[\d+\]", path):
        if part.startswith("["):
            idx = int(part[1:-1])
            if not isinstance(obj, list) or idx >= len(obj):
                return None
            obj = obj[idx]
        else:
            if not isinstance(obj, dict) or part not in obj:
                return None
            obj = obj[part]
    return obj

def generate_password(length=20, classes=(string.ascii_lowercase, string.ascii_uppercase, string.digits, "!@#$%^&*")):
    """One char from every class guaranteed, the rest random, then shuffled. `secrets`, never `random`."""
    if length < len(classes):
        raise ValueError("too short for the policy")
    chars = [secrets.choice(c) for c in classes]
    alphabet = "".join(classes)
    chars += [secrets.choice(alphabet) for _ in range(length - len(chars))]
    for i in range(len(chars) - 1, 0, -1):           # Fisher-Yates with secrets
        j = secrets.randbelow(i + 1)
        chars[i], chars[j] = chars[j], chars[i]
    return "".join(chars)

def average_by_year(csv_text, year_field, value_field):
    """CSV average per year without pandas. Skips blank or non-numeric values."""
    from collections import defaultdict
    totals, counts = defaultdict(float), Counter()
    for row in csv.DictReader(io.StringIO(csv_text)):
        try:
            v = float(row[value_field])
        except (ValueError, KeyError, TypeError):
            continue
        totals[row[year_field]] += v
        counts[row[year_field]] += 1
    return {y: totals[y] / counts[y] for y in totals}
```

What each one is really testing, and the line to say:

| Task | What they listen for |
| --- | --- |
| Valid IPv4 | Edge cases: leading zeros, empty octets, 256, three parts. Then "and `ipaddress.ip_address` does this in production". |
| Binary to decimal | Rejecting bad input, and `int(s, 2)` as the real answer once you've shown the loop. |
| Palindrome with punctuation | Normalise first, then compare; O(n) time, O(n) space, and the two-pointer O(1)-space version if pushed. |
| Substring index | Naive is O(n·m); name KMP or `str.find` without implementing them. |
| Two Sum | One pass with a dict; return indices, handle no answer. |
| Balanced brackets | A stack; the empty-stack pop is the bug they watch for. |
| Word frequency | Tokenising is the hard part (case, apostrophes); `Counter.most_common`. |
| jq-lite | Path parsing with a regex; return `None` instead of raising on a missing key. |
| Password generator | `secrets` not `random`; guarantee each class; shuffle with a CSPRNG; say you'd prefer a passphrase for humans. |
| CSV average | `csv.DictReader`, skip bad rows and count them, never crash on one blank cell. |

Optimisation pushes a Blind candidate hit on these: "can you do it in one pass", "what's the memory", "what if the input is a stream". Have the answer ready before they ask.

[← Coding overview](README.md) · [Log-analysis tasks](log-analysis-tasks.md) · [Shell one-liners](shell-one-liners.md)
