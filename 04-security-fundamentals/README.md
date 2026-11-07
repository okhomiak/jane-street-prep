# 04 · Security fundamentals

[← Coding](../03-coding/README.md) · [Index](../README.md) · [Next: Networking →](../05-networking/README.md)

The fundamentals interviews are where Jane Street asks what one candidate called "undergraduate security course" questions, then pushes until you reach the edge of what you know ([Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,29.htm)). Answer in three beats: the mechanism, the attack it stops or enables, the trade-off.

This block has five files:

| File | Covers |
| --- | --- |
| this page | The five questions Jane Street is known to ask, answered in full |
| [auth-and-identity.md](auth-and-identity.md) | Passwords, hashing, sessions, cookies, OAuth, OIDC, SAML, JWT, Kerberos, MFA, passkeys, SSO |
| [crypto-and-tls.md](crypto-and-tls.md) | Symmetric and asymmetric, modes, IVs, HMAC, signatures, DH, forward secrecy, TLS handshake and attacks, PKI |
| [web-and-appsec.md](web-and-appsec.md) | Same-origin, CORS, XSS, CSRF, SQLi, SSRF, XXE, IDOR, traversal, HSTS, API security, secure login design |
| [reported-questions.md](reported-questions.md) | The drill list: every reported fundamentals question grouped by topic, each with where it was asked, answers in the pages above |

Each question shows where it was asked. Click a question to see the answer, and try to say yours first.

---

## The five Jane Street questions

<details>
<summary><b>Q1. What makes a password strong, and how would you estimate its complexity?</b> · Jane Street Security Engineer, Hong Kong</summary>

Strength is the size of the space an attacker has to search, measured in bits of entropy, divided by how fast they can search it. Length beats character mixing: every extra character multiplies the space, a symbol only adds a little. A 4-word passphrase from a 7,776-word Diceware list is about 52 bits (12.9 bits a word); 5 words is about 64 bits; a 16-character random string from a manager is over 100 bits.

Estimate it by asking how the password was made, not what it looks like. `Summer2026!` looks mixed but comes from a tiny human pattern space, so a cracker with rules finds it in seconds. A random 20-character manager password is effectively uncrackable offline.

The other half is the hash. A GPU rig tries tens of billions of MD5 guesses a second but only thousands of bcrypt or Argon2 guesses, so a weak hash turns a decent password into a cracked one. Say the modern guidance: long, unique per site, generated and stored by a manager, checked against breach lists, MFA on anything that matters, no forced rotation (NIST 800-63B dropped it because it produces `Password1`, `Password2`).
</details>

<details>
<summary><b>Q2. You're implementing a password manager. How should it store passwords, and where does the encryption key come from?</b> · Jane Street Security Engineer, Hong Kong</summary>

Store the vault as ciphertext only, encrypted with an authenticated cipher (AES-256-GCM or XChaCha20-Poly1305) so tampering is detected. The server never sees the master password or the key: that's the zero-knowledge model.

The key comes from the master password through a slow, salted key derivation function: Argon2id (or PBKDF2 with a high iteration count, scrypt). Derive it on every unlock, keep it only in memory, never write it to disk. Use a key hierarchy: master password → KDF → master key → unwraps a random vault key that actually encrypts the items. Then changing the master password re-wraps one key instead of re-encrypting every item, and you can add a second unlock path (biometric via TPM or Secure Enclave, a recovery key) by wrapping the same vault key again.

Details that show depth: a unique salt per user stored next to the ciphertext, a per-item nonce never reused with the same key, an HMAC or the AEAD tag to catch modification, autofill that only fires on the exact origin (that's the anti-phishing feature), clipboard clearing, memory zeroing after use, and 2FA on the account to protect the download of the encrypted blob, which is defence in depth rather than part of the crypto.

Trade-off to mention: the KDF cost is a dial. Too low and offline cracking of a stolen vault gets cheap; too high and unlock on a phone takes seconds.
</details>

<details>
<summary><b>Q3. For TOTP, how does the secret become a code that keeps changing, and how does it cope with the client and server clocks disagreeing?</b> · Jane Street Security Engineer, Hong Kong</summary>

At enrolment the server generates a random secret (shown as a base32 string or QR code) and both sides keep it. Every 30 seconds the client computes `HMAC-SHA1(secret, floor(unix_time / 30))`, takes 4 bytes from the HMAC at an offset chosen by its last nibble (dynamic truncation), and reduces that number mod 10^6 to get 6 digits. That's RFC 6238 on top of HOTP (RFC 4226). The code changes because the counter input changes; the secret never leaves either side.

Clock drift: the server computes the code for the current step and usually one step either side (±30 s) and accepts any match. Good implementations also remember each user's observed drift and shift the window, and they reject a code that has already been used in that step to stop replay.

Then the limits, unprompted: TOTP proves possession of the secret, not the identity of the site, so a phishing proxy can relay a live code. Passkeys and FIDO2 keys bind the response to the origin, which is why they're the answer for anything high value. And the server must store the secret encrypted at rest, because unlike a password hash it's a symmetric secret that reproduces the codes.
</details>

<details>
<summary><b>Q4. Walk through the data flow when a user opens a website and logs in. POST or GET, and why?</b> · Jane Street Security Engineer, Hong Kong</summary>

DNS resolves the name, TCP connects, TLS negotiates keys and proves the server's identity through its certificate. The browser sends `GET /login`; the server returns the form. Submitting sends `POST /login` with the credentials in the body over the encrypted connection. The server looks up the user, hashes the submitted password with the stored salt using the same slow KDF, compares in constant time, then creates a session: a long random ID stored server-side, returned in a `Set-Cookie` header marked `HttpOnly`, `Secure` and `SameSite`. Every later request carries the cookie; logout deletes the server-side record so the cookie is dead even if someone kept a copy. A stateless alternative is a signed token (JWT) in the response, with the trade-off that you can't revoke it before expiry without a denylist.

POST, for three reasons: GET puts parameters in the URL, which lands in browser history, proxy and server logs, and the `Referer` header of the next page; GET is meant to be safe and repeatable, and a login changes state; and crawlers and prefetchers follow GETs. TLS encrypts the URL in transit either way, so the difference is where it gets written down, not the wire.

Follow-ups to be ready for: where CSRF fits (cookie-based sessions need a token or `SameSite`), why the session ID must be regenerated on login (session fixation), and what the password manager autofill checks before filling (exact origin).
</details>

<details>
<summary><b>Q5. "I don't want 2FA. All my passwords are unique and I'm good at spotting phishing."</b> · Jane Street Security Engineer, New York</summary>

Agree with the true part first: unique passwords kill credential stuffing, which is the most common account takeover. Then the gap: a site can leak your password through its own breach, an infostealer on your machine grabs it regardless of your judgement, and a well-built proxy phish fools careful people on a tired day. 2FA covers the cases where your password is already gone.

Then measure it, because the posting says not everyone should lock down their whole life. A forum account can skip it. Email, the password manager, banking and anything that can reset other accounts can't, because those are the keys to everything else. Pick the cheapest 2FA that fits: a passkey or hardware key costs seconds per login and can't be phished; TOTP is fine for medium-value accounts; SMS is better than nothing.

Close with the trade you'd offer: "Put a key on your email and your manager, and I won't ask about the rest." The full version of this, with the other judgement questions, is in [block 11](../11-judgement-and-hygiene/README.md).
</details>

---

## How Jane Street pushes

From the reports: they ask a basic question, then follow the thread. "How does TOTP work" becomes "what about clock drift" becomes "what if the secret leaks". Expect four or five follow-ups on one topic. Two things keep you safe: know one layer deeper than the question, and say "I'm not sure, here's my reasoning" the moment you reach the edge. That second one is a hiring principle, not a weakness.

Next: [auth-and-identity.md](auth-and-identity.md)

[← Coding](../03-coding/README.md) · [Index](../README.md) · [Next: Networking →](../05-networking/README.md)
