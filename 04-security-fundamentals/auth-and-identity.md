# 04a · Authentication and identity

[← Block 04](README.md) · [Crypto and TLS →](crypto-and-tls.md) · [Web and AppSec →](web-and-appsec.md)

Where each question was asked is in the summary line. Sources: [JS] Jane Street reports on Glassdoor, [DM] Daniel Miessler's interview questions, [TW] the tadwhitaker Glassdoor collection, [GN] gracenolan's study notes, [GD] other Glassdoor reports. Links are in [SOURCES.md](../SOURCES.md).

---

## Passwords and hashing

<details>
<summary><b>How should a site store passwords?</b> · [JS] [DM] [TW]</summary>

Never the password, never a fast hash. Store `argon2id(salt, password)` or bcrypt, with a unique random salt per user and a work factor tuned so a check takes tens of milliseconds. Compare in constant time. Add a server-side "pepper" kept outside the database so a dumped table alone isn't crackable. Check new passwords against breach lists. Rotate the work factor upwards over time by re-hashing on login.
</details>

<details>
<summary><b>What are rainbow tables, and what is salting?</b> · [DM]</summary>

A rainbow table is a precomputed map from hashes back to passwords, compressed with hash chains so a terabyte covers a huge space. It works when the same password always gives the same hash. A salt is a random per-user value mixed into the hash, so the attacker has to recompute everything per user and precomputation is useless. Salts aren't secret; the slowness of the KDF is what buys time.
</details>

<details>
<summary><b>Our database was stolen. Passwords were one round of SHA-256 with a static salt. What now?</b> · [TW]</summary>

Treat every password as compromised. A static salt means one rainbow table or one GPU run covers everyone, and SHA-256 runs at billions of guesses a second. Now: force resets, invalidate sessions and tokens, tell users to change the same password elsewhere, watch for credential stuffing against you and partners. Change: argon2id with per-user salts, a pepper in a secrets store, MFA, breach-list checks, and rate limits on login. Then the harder question: how was the database stolen, and what else was in it.
</details>

<details>
<summary><b>Why is scheduled password rotation considered harmful?</b> · [GN]</summary>

Forced rotation produces predictable sequences (`Winter25!` to `Spring26!`), written-down passwords and help-desk load, without stopping the attack that matters, which is use of a stolen password within hours. Rotate on evidence of compromise, screen against breach lists, and use MFA. NIST 800-63B says the same.
</details>

<details>
<summary><b>How do you handle account brute-forcing?</b> · [DM]</summary>

Layered: slow hashing so offline cracking is expensive; rate limits per account and per source; progressive delays; lockout with care, because lockout is a denial-of-service lever against your own users; CAPTCHA after a few failures; alerts on success-after-failures and on password sprays (one source, many users); MFA so a guessed password isn't enough. Say where fail2ban sits: noise reduction at the edge, not the control.
</details>

<details>
<summary><b>Why can biometrics be a problem compared with passwords?</b> · [GN]</summary>

You can't rotate a fingerprint. Biometrics are identifiers, not secrets, and they're probabilistic, so there's a false-accept rate. They work well as a local unlock that releases a key held in secure hardware (phone, passkey), badly as a secret sent to a server.
</details>

## Sessions, cookies and tokens

<details>
<summary><b>How do cookies and sessions work, and how do they differ from tokens?</b> · [TW] [GN]</summary>

A session is server-side state keyed by a long random ID; the browser holds only the ID in a cookie and sends it automatically to that domain. Revocation is trivial (delete the record). A token (JWT) is self-contained and signed, so any service can verify it without a lookup, which suits APIs and microservices, but it can't be revoked before expiry without a denylist, so keep access tokens short-lived with a refresh token. Cookie flags: `HttpOnly` (JavaScript can't read it), `Secure` (HTTPS only), `SameSite` (CSRF), scoped `Domain` and `Path`. Regenerate the session ID on login to kill session fixation.
</details>

<details>
<summary><b>How does HTTP handle state?</b> · [DM]</summary>

It doesn't. Each request is independent; cookies, bearer tokens and URL parameters are the workarounds that carry identity between requests. That's why session management is where most authentication bugs live.
</details>

<details>
<summary><b>Explain how JWT works and what goes wrong with it.</b> · [TW]</summary>

Header, payload and signature, base64url-joined. The signature (HMAC with a shared secret, or RSA/ECDSA with a private key) lets any holder of the key or public key verify the claims without a database. Classic failures: accepting `alg: none`, letting the token choose the algorithm so an RSA public key gets used as an HMAC secret, no expiry, secrets in the payload (it's only encoded, not encrypted), long-lived tokens in localStorage where XSS can read them, and no revocation. Fix: pin the algorithm server-side, short expiry, `kid`-based key lookup, store in `HttpOnly` cookies when the client is a browser.
</details>

<details>
<summary><b>Explain OAuth 2.0 and OIDC. How do bearer tokens get stolen?</b> · [TW] [GN] [GD Google: api security, bearer tokens, oauth, oidc]</summary>

OAuth is delegation: a user lets app A access their data at service B without giving A their password. Authorization code flow: A redirects the user to B, B authenticates them and returns a short code to A's registered redirect URI, A swaps the code for an access token at B's token endpoint (with PKCE so an intercepted code is useless). OIDC adds an ID token (a JWT about who the user is) on top, which is what "log in with Google" uses. Bearer tokens are stolen through XSS, logs, referrer leaks, open redirects on the redirect URI, malicious browser extensions and leaked CI secrets; whoever holds the token is the user. Mitigate with short lifetimes, sender-constrained tokens (DPoP or mTLS), exact redirect URI matching, and scopes that are as narrow as possible.
</details>

<details>
<summary><b>How does SAML work, and how does SSO work?</b> · [TW] [GN]</summary>

SAML: the service provider redirects the user to the identity provider; the IdP authenticates them and returns a signed XML assertion through the browser; the SP validates the signature against the IdP certificate and creates a local session. SSO is one IdP (Okta, Entra, Google) issuing assertions or OIDC tokens to many apps, so one strong login with MFA covers everything and offboarding is one switch. Risks: signature wrapping and XML parsing bugs in old libraries, an IdP compromise is a compromise of everything, and apps that skip signature validation.
</details>

<details>
<summary><b>How does Kerberos work?</b> · [TW] [GN]</summary>

Tickets instead of passwords on the wire. The client proves its identity to the KDC's authentication service (using a key derived from the password, with pre-authentication) and gets a ticket-granting ticket. To reach a service it presents the TGT to the ticket-granting service and gets a service ticket encrypted with that service's key, plus a session key. The service decrypts the ticket and trusts it. Clocks must be within about five minutes. Attacks live in the details: Kerberoasting (request service tickets for accounts with SPNs and crack them offline, so service accounts need long random passwords), AS-REP roasting (accounts without pre-auth), pass-the-ticket, golden tickets (forged TGTs using the krbtgt hash, so rotate it twice after a domain compromise), silver tickets (forged service tickets with a service account hash). More in [block 07](../07-windows-and-endpoints/README.md).
</details>

<details>
<summary><b>Authentication versus authorisation? ACLs, RBAC, service accounts?</b> · [TW] [GN]</summary>

Authentication is proving who you are; authorisation is what you're allowed to do once that's known. ACLs attach permissions to resources; RBAC attaches permissions to roles and roles to people, which scales and audits better. Service accounts are identities for software: they can't do MFA, their credentials get copied into config and CI, and they're usually over-privileged, so give them short-lived credentials (workload identity, IAM roles, OIDC federation from CI) and scope each to one job. The question behind the question is least privilege.
</details>

## Multi-factor and passkeys

<details>
<summary><b>Compare MFA methods.</b> · [JS] [GN]</summary>

SMS: cheap, stops credential stuffing, loses to SIM swap and relay phishing. TOTP: no network needed, secret must be protected, still relayable by a phishing proxy. Push: convenient, loses to push fatigue unless number matching is on. FIDO2 keys and passkeys: the response is bound to the site's origin, so a lookalike site gets nothing; they're the only option that stops phishing. Order of rollout: admins and money-moving roles first, then everyone with a passkey. Measured view: match the method to the account's value; don't demand a hardware key for a lunch-ordering app.
</details>

<details>
<summary><b>How do U2F and FIDO2 stop phishing?</b> · [GN]</summary>

The browser passes the origin into the signed challenge. A key registered for `bank.com` signs a challenge that includes `bank.com`; a phishing page at `bank-login.com` gets a signature for its own origin, which the real bank rejects. Nothing the user can be tricked into typing changes that. The private key never leaves the authenticator.
</details>

<details>
<summary><b>How would you get the whole firm onto a password manager?</b> · Jane Street theme</summary>

Make it the easiest path: SSO into the manager, browser extension that autofills, import from the browser, a 10-minute enrolment session, shared vaults for teams so people stop pasting secrets in chat. Measure adoption and chase the stragglers one by one. The pitch is "you'll never type a password again", not "policy says".
</details>

## Delegation, revocation, and the decision point's own bugs

<details>
<summary><b>A service has to call another service on behalf of a user. Impersonation or delegation?</b> [curated] [RFC 8693]</summary>

OAuth token exchange ([RFC 8693](https://www.rfc-editor.org/rfc/rfc8693)) gives you both. **Impersonation**: the new token's subject is the user and nothing records that a service is acting; the downstream can't tell the difference. **Delegation**: the subject is still the user, and an `act` claim names the service acting for them, so the downstream can authorise the pair and the log shows who really made the call. For agents, delegation is the right default: the agent's own identity sits in `act`, its rights are the intersection of its own and the user's, and the token is short-lived and scoped to one audience. Keycloak 26.8 shipped exactly this, in preview, described as for agents ([Keycloak](https://www.keycloak.org/docs/latest/release_notes/index.html)). Say why impersonation is dangerous: an audit trail that can't separate a person from a program acting as them.
</details>

<details>
<summary><b>Someone leaves at 10:00. How long until every session and token they hold is dead?</b> [curated]</summary>

Depends on the slowest path, so list them: the identity provider disables the account (seconds); refresh tokens stop working (next refresh); access tokens already issued live until expiry, so keep them short (minutes); sessions in applications that never re-check live until they time out. Push beats poll: the Shared Signals Framework, CAEP and RISC have been final OpenID standards since September 2025, so an identity provider can send a session-revoked event and a receiver can kill the session immediately ([OpenID](https://openid.net/press-release-openid-foundation-finalizes-global-standards-for-real-time-identity-security/)). Then the long-lived things nobody thinks of: SSH keys, personal access tokens, API keys, OAuth grants to third-party apps. Measure it as a number, time to revoke, and test it.
</details>

<details>
<summary><b>You put a cache in front of an authorisation decision for speed. What can go wrong?</b> [curated, current]</summary>

Two bug families showed up again and again in 2026 in the systems that make access decisions. **The cache answers the wrong question.** OpenFGA twice had cache-key collisions where one check got another request's cached answer ([CVE-2026-41131](https://bugzilla.redhat.com/show_bug.cgi?id=2460447), [CVE-2026-48096](https://exploit-intel.com/vuln/CVE-2026-48096)); Vault's policy cache could resolve policies across namespaces and, through a key collision, resolve an alias to root (fixed in 2.1.2 and 2.2.1, [release notes](https://developer.hashicorp.com/vault/docs/updates/release-notes)); Istio's JWKS resolver cached key responses on its background refresh without the checks the normal path applies ([bulletin](https://istio.io/latest/news/security/istio-security-2026-009/)). **The rule matches input its author didn't mean.** A slash in a value substituted into a Vault templated policy path escaped the intended path ([CVE-2026-5006](https://www.sentinelone.com/vulnerability-database/cve-2026-5006/)); Istio compared host headers case-sensitively, and at some proxies source-IP rules matched the proxy's own address, so a DENY failed open ([bulletin](https://istio.io/latest/news/security/istio-security-2026-009/)). So: build the cache key from every input the decision reads, including policy and data versions; keep the lifetime shorter than the fastest revocation, or evict on revoke; put inputs in one canonical form before matching; give every rule a test for the request it must refuse; and patch the decision point as fast as the secrets manager.
</details>

## Rapid-fire

| Question | Answer in one line |
| --- | --- |
| Difference between a session cookie and a persistent cookie? | No expiry means it dies with the browser; `Expires` or `Max-Age` keeps it, so logins survive restarts but so does theft. |
| What is session fixation? | Attacker sets the victim's session ID before login; fix by issuing a new ID at login. |
| What is credential stuffing? | Replaying leaked username/password pairs across sites; unique passwords and MFA stop it. |
| What's a pepper? | A secret added to every hash and stored outside the database, so a dumped table alone can't be cracked. |
| What is a magic link? | Emailed one-time login URL; security rests on the mailbox, so it needs short expiry and single use. |
| Why constant-time comparison? | A byte-by-byte compare that stops at the first mismatch leaks how many bytes matched through timing. |
| What is step-up authentication? | Ask for a stronger factor only for sensitive actions (payments, admin), keeping daily friction low. |
| Device trust? | Certificates or MDM attestation that the device is managed and healthy, used as a factor in zero-trust access. |
| Impersonation vs delegation? | Same subject; delegation adds an `act` claim naming who is acting, so the downstream can tell. |
| Fastest way to kill a session elsewhere? | A pushed Shared Signals session-revoked event, plus short-lived access tokens. |
| Cache in front of an access decision? | Key on every input, expire faster than revocation, test with the cache on. |

[← Block 04](README.md) · [Crypto and TLS →](crypto-and-tls.md) · [Web and AppSec →](web-and-appsec.md)
