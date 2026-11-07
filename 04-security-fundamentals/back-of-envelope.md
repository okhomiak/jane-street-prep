# Back-of-envelope security maths

[← Fundamentals](README.md) · [Index](../README.md)

Jane Street asked a security candidate to estimate password complexity ([block 01](../01-process/README.md)); that is an estimation question, and estimation is the firm's native register. Probability brainteasers show up in their adjacent ops loops, and even if none has been reported in a security loop, an interviewer who hears "roughly 2 to the 40, so hours not years" will lean in. Be able to do these on a whiteboard, out loud, with the approximations named. Tags: [JS] Jane Street, [est] established.

The two numbers to carry: **2^10 ≈ 10^3** (so 2^20 ≈ a million, 2^30 ≈ a billion, 2^40 ≈ a trillion) and **a year ≈ 3 × 10^7 seconds**.

---

## Passwords and brute force

<details>
<summary><b>Estimate the strength of a password, and how long it takes to crack.</b> · [JS Hong Kong]</summary>

Entropy in bits = length × log2(alphabet size), *if the characters are random*. Lowercase only: log2(26) ≈ 4.7 bits per character; lowercase plus digits: 5.2; full printable ASCII (95): 6.6. So eight random lowercase letters ≈ 37 bits; twelve random mixed characters ≈ 79 bits; a four-word random passphrase from a 7,776-word list (Diceware) ≈ 4 × 12.9 ≈ 52 bits.

Crack time = 2^entropy ÷ guesses per second. The guess rate depends entirely on the hash: offline against MD5 or NTLM a single GPU does on the order of 10^10 to 10^11 guesses a second, against bcrypt at a sane cost maybe 10^4 to 10^5, and online against a rate-limited login a few per second. So 37 bits (≈ 10^11 guesses) against NTLM is seconds; against bcrypt it is days to weeks; and 79 bits (≈ 10^24) is beyond any attacker against anything.

The judgement point Jane Street was after: a human-chosen `Summer2026!` has nowhere near 6.6 bits per character, because it is drawn from a tiny space of patterns a cracker's rules enumerate first, so you estimate from *how it was generated*, not how it looks. Length and randomness beat character classes, and the hash function matters as much as the password.
</details>

<details>
<summary><b>How much does rate limiting buy you online?</b> · [est]</summary>

At 10 guesses a second with no lockout, an attacker makes about 10^6 guesses a day. A 1-in-10^6 password (a common top-million list) falls in a day per account; spread across 10,000 accounts in a password spray, the attacker tries one guess per account per day and stays under every per-account lockout while still covering a million account-guesses a month. That is why you rate-limit per source and per account, alert on many-accounts-one-password patterns, and why MFA is the control that makes the arithmetic irrelevant.
</details>

---

## Collisions and the birthday bound

<details>
<summary><b>Why is a 128-bit hash "only" 64 bits against collisions? Why do 96-bit GCM nonces have a key budget?</b> · [est]</summary>

The birthday bound: with N possible values you expect a collision after about √N samples. So a 128-bit hash gives ~2^64 collision resistance (fine), MD5's 128 bits were fine on paper until the maths was broken, and a 64-bit identifier collides after only ~2^32 ≈ 4 billion values, which a busy system reaches. For AES-GCM with a random 96-bit nonce, the probability of a repeat approaches dangerous levels around 2^32 messages under one key, so you rotate keys well before that or use a counter; a repeated GCM nonce is catastrophic ([crypto](crypto-and-tls.md)). The one-liner: halve the bits for collisions, and ask how many samples the system will ever produce.
</details>

---

## Alerts and the base-rate fallacy

<details>
<summary><b>A detector is 99% accurate. Most of its alerts are false. Explain.</b> · [est] [the single most useful piece of maths in detection]</summary>

Bayes, with round numbers. Suppose the true event happens in 1 of every 10,000 sessions. In a million sessions there are 100 real events and 999,900 benign ones. A 99%-accurate detector (99% true-positive rate, 1% false-positive rate) catches 99 of the 100 real ones, and also flags 1% of 999,900 ≈ 9,999 benign ones. So roughly 10,000 alerts, of which 99 are real: under 1% precision. Nearly every alert is a false positive, and the analysts drown.

Three consequences you can state: precision is driven by the base rate, so rare events need very low false-positive rates, not just high detection rates; the fix is to raise the base rate before the expensive step (enrich and correlate so the detector only runs on suspicious context) or to accept the detector as a *filter* that feeds a cheaper review, not as the alert; and this is why "the classifier catches 95% in eval, do you ship it" in [block 16](../16-ai-and-llm-security/README.md) gets the answer "as one layer".
</details>

---

## Sizing and scale

<details>
<summary><b>How many events per second will the SIEM see? How much storage?</b> · [est]</summary>

Multiply and round. 5,000 endpoints × ~20 security events a second each (EDR plus Windows audit at a sane policy) ≈ 100,000 events a second. At ~500 bytes an event that is ~50 MB/s, ~4 TB a day raw, ~1.5 PB a year before compression and tiering; compress 5 to 10×, keep 30 to 90 days hot and the rest cold. The interview value is less the number than showing you know the levers: audit policy verbosity, which sources matter (authentication, process creation, DNS, proxy), hot versus cold retention, and that the order path's packet captures ([block 17](../17-trading-domain/README.md)) are a separate, much larger stream you sample or keep short.
</details>

<details>
<summary><b>How long to scan or brute-force the internal network? How fast does ransomware spread?</b> · [est]</summary>

A /16 is 65,536 addresses; at 1,000 probes a second a full port sweep of the common ports is minutes, which is why port-scan detection is about rate and fan-out, not individual probes. Encryption speed is disk-bound: at a few hundred MB/s a workstation's useful data is gone in minutes, so a ransomware response is measured in minutes to isolate, not hours, and the real control is stopping the spread (SMB between workstations, credential hygiene), not out-running the encryption.
</details>

---

## Probability they might throw in

Ops-engineer and support loops at Jane Street have included probability brainteasers ([block 01](../01-process/README.md)). None reported in a security loop, but if one appears, do not freeze: state the model, compute, sanity-check. Two shapes to have practised:

- **Expected value.** "Expected number of coin flips to get two heads in a row" (asked in a 2026 SWE loop): set up states, E = 6. The method matters more than the number.
- **Conditional probability.** The base-rate problem above is the same muscle. Draw the 2×2 table (real/benign × flagged/not) and read the answer off it.

Say the approximation you are making ("call a year 3 × 10^7 seconds"), keep powers of two and ten straight, and give a range rather than a false-precision number.

[← Fundamentals](README.md) · [Index](../README.md)
