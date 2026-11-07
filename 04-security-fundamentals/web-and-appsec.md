# 04c · Web and application security

[← Block 04](README.md) · [Auth and identity](auth-and-identity.md) · [Crypto and TLS](crypto-and-tls.md)

Google's and DoorDash's loops included a round where they show you a running app and ask you to find the XSS, SSRF and IDOR ([Glassdoor](https://static.glassdoor.com.ar/Interview/security-engineer-interview-questions-SRCH_KO0,17_IP372.htm)); Amazon hands you code and asks why the bugs exist ([Glassdoor](https://www.glassdoor.ca/Interview/Amazon-Interview-E6036-RVW92367615.htm)). Jane Street's threat-modelling posting wants an offensive background. So know each bug three ways: how to find it, how to exploit it, how to fix it. Tags: [DM] Miessler, [TW] tadwhitaker, [GN] gracenolan, [GD] Glassdoor.

---

## Browser model

<details>
<summary><b>Explain the same-origin policy and CORS, including preflight.</b> · [TW] [GN]</summary>

Origin is scheme + host + port. A page can send requests anywhere, but by default script can only read responses from its own origin, which is what stops `evil.com` reading your bank page. CORS is the server opting out: `Access-Control-Allow-Origin` tells the browser which origins may read. Requests that aren't "simple" (custom headers, JSON content type, PUT/DELETE) trigger an `OPTIONS` preflight first. The classic bug is reflecting the request's `Origin` header back with `Allow-Credentials: true`, which gives every site read access to the logged-in user's data. `null` origin and regex mistakes (`bank.com.evil.com`) are the other two.
</details>

<details>
<summary><b>What do HSTS and certificate transparency do? What is HttpOnly for?</b> · [GN]</summary>

HSTS tells the browser to use HTTPS for this host for the next N seconds and refuse to click through certificate errors, which kills SSL stripping; preloading covers the first visit. Certificate transparency puts every issued certificate in public logs so a domain owner can see misissuance. `HttpOnly` stops script reading a cookie, so XSS can't steal the session, though it can still make requests as the user while the page is open.
</details>

## The big four

<details>
<summary><b>XSS: types, defences, and explain it to a 10-year-old.</b> · [DM] [TW] [GD Amazon: "XSS and its types"]</summary>

Someone sneaks their own instructions into a page so your browser runs them thinking they came from the site. Reflected: the payload is in the request and echoed back (a crafted link). Stored: it's saved on the server and served to everyone (a comment). DOM-based: client-side script writes attacker-controlled data into the page without the server being involved. Impact: steal sessions, act as the user, keylog, deface, pivot into internal apps. Defences in order: output encoding for the context you're writing into (HTML, attribute, JavaScript, URL), templating engines that escape by default, a Content Security Policy that blocks inline script, `HttpOnly` cookies to limit the blast, input validation as a second layer. Miessler's note: emphasise output encoding over input validation.
</details>

<details>
<summary><b>CSRF: what is it, how do you defend, and how would you spot it as an admin?</b> · [DM] [TW]</summary>

The browser sends cookies automatically, so a page on `evil.com` can make the user's browser `POST /transfer` to the bank and the bank sees a valid session. Defences: a per-session or per-request token the attacker can't read (synchroniser token), `SameSite=Lax` or `Strict` cookies, checking `Origin` or `Sec-Fetch-Site` headers, re-authentication for sensitive actions. Detecting attacks: requests that change state with a missing or wrong token, a foreign `Origin` or `Referer`, bursts of identical state-changing requests. Miessler's trap: "check the referrer header" alone is a weak answer because it's spoofable from non-browser clients and often stripped. Difference from XSS: CSRF rides the user's trust in the site; XSS rides the site's trust in the user.
</details>

<details>
<summary><b>SQL injection: variants, finding it, fixing it.</b> · [GN] [TW]</summary>

Untrusted input concatenated into a query changes the query. Error-based (database errors leak data), union-based (append your own `SELECT`), blind boolean (true/false pages), time-based (`SLEEP(5)`), second-order (stored now, triggered later). Find it with single quotes, `' OR 1=1--`, timing, and sqlmap in a test environment. Fix: parameterised queries everywhere, least-privilege database accounts, an ORM used properly, and input validation as a backstop, not the control. Say that stored procedures and escaping are not substitutes for parameters.
</details>

<details>
<summary><b>SSRF: what it is and why cloud makes it worse.</b> · [TW] [GN] [GD Google loop: ssrf]</summary>

The server fetches a URL the attacker controls, so the attacker reaches things only the server can: internal admin panels, `localhost` services, and the cloud metadata endpoint `169.254.169.254`, which hands out IAM credentials. That's how Capital One lost 100 million records in 2019. Fixes: allowlist destinations, resolve DNS then check the IP against private ranges (and re-check at connect time to beat DNS rebinding), block link-local, use IMDSv2 which needs a PUT with a token and a hop limit, and give the instance role the least privilege possible so a stolen credential buys little.
</details>

## The rest of the list

<details>
<summary><b>XXE, IDOR, directory traversal, LFI/RFI, open redirects, deserialisation.</b> · [TW] [GN] [GD Google loop: idor]</summary>

XXE: an XML parser resolving external entities reads local files or hits internal URLs; disable DTDs and external entities. IDOR: the app trusts an ID in the request (`/invoice/1234`) without checking ownership; fix with object-level authorisation on every access, and use unguessable IDs only as a second layer. Directory traversal: `../` in a filename escapes the intended folder; normalise and `realpath`, then check the prefix (see the `cd` solution in [block 03](../03-coding/README.md)). LFI/RFI: including a file by user-controlled name, which becomes code execution with PHP wrappers or log poisoning; never build include paths from input. Open redirect: `?next=https://evil.com` after login, used to phish and to steal OAuth codes; allowlist relative paths. Insecure deserialisation: `pickle`, Java serialisation, PHP `unserialize` on untrusted bytes is remote code execution; use JSON with schemas.
</details>

<details>
<summary><b>API security: microservices, bearer tokens, what to check.</b> · [GD Google: "api security, how to secure microservices"]</summary>

Authenticate every call (OAuth 2.0 with short-lived tokens, mTLS between services), authorise per object and per action, validate schemas, rate limit per client, don't leak internals in errors, log with request IDs, version APIs, and never trust the gateway alone. Between services: a service mesh or mTLS so a compromised pod can't call anything it likes, and workload identity instead of long-lived secrets. Reference the OWASP API Top 10: broken object-level authorisation is number one for a reason.
</details>

<details>
<summary><b>How would you implement a secure login on a high-traffic site?</b> · [DM]</summary>

TLS everywhere with HSTS, a proven framework rather than custom auth code, argon2id hashing, rate limits that scale (shared counters), MFA with passkeys offered first, session IDs regenerated on login, `HttpOnly`/`Secure`/`SameSite` cookies, CSRF tokens, breach-list checks, monitoring for stuffing. Performance point: hashing is the expensive step by design, so size the auth tier for it and consider a dedicated identity service.
</details>

<details>
<summary><b>How would you secure a LAMP stack?</b> · [GD Google]</summary>

Linux: patching, minimal packages, firewall, SSH keys only, fail2ban at the edge, separate service accounts, AppArmor or SELinux. Apache: latest version, remove default pages and `ServerTokens`, TLS with modern config, `mod_security` or a WAF in front, directory listing off. MySQL: bind to localhost, no root over network, per-app accounts with least privilege, encrypted backups. PHP: `display_errors` off, `allow_url_include` off, `open_basedir`, parameterised queries, dependency updates, and this month's PHP2Shell patches ([block 12](../12-security-news/README.md)). Then logging and monitoring across all four.
</details>

<details>
<summary><b>Man-in-the-middle attacks: how, and how do you stop them?</b> · [TW]</summary>

Get between two parties and relay: ARP spoofing on a LAN, rogue Wi-Fi, DNS poisoning, BGP hijacks, compromised CAs, or malware installing a root certificate on the victim. Defences: TLS with proper validation, HSTS and preloading, certificate pinning for your own apps, DNSSEC and DoH, 802.1X and dynamic ARP inspection on the network, and user education for the "click through the warning" case.
</details>

<details>
<summary><b>How would you conduct a security code review? What are the first things you do with a repo?</b> · [TW] [GD Amazon]</summary>

First: understand what the app does and where trust boundaries are. Then grep for the dangerous sinks (`exec`, `system`, `eval`, raw SQL, `pickle`, file opens, `innerHTML`), trace each back to its sources (`request.*`, headers, files, queues), and check auth on every route. Then secrets in the repo and history (trufflehog, gitleaks), dependency versions, and the build pipeline. Prioritise by reachability and impact, write each finding with a reproduction and a fix. Run the automated tools (semgrep, CodeQL) but say that they find classes, not logic bugs. The drill in [block 03](../03-coding/README.md) is the practice version.
</details>

<details>
<summary><b>How can GitHub webhooks be abused? How would you search our repos for secrets?</b> · [TW]</summary>

A webhook is an outbound POST to a URL on events. Abuse: pointing it at an internal service (SSRF from GitHub's network, or from a self-hosted runner), unsigned payloads that an attacker forges to trigger deploys, or a compromised receiver harvesting payloads with code and user data. Verify the HMAC signature, allowlist receivers, don't put secrets in payloads. Secrets search: run gitleaks or trufflehog over every repo including history and branches, add a pre-receive or pre-commit hook, enable push protection, and rotate anything found because a secret in history is public to anyone who ever cloned.
</details>

## Rapid-fire

| Question | One-line answer |
| --- | --- |
| Difference between HTTP and HTML? | Protocol versus markup; a calm answer is the point. |
| What does CSP do? | Tells the browser which script and resource sources are allowed; `nonce` or `hash` for inline, kills most XSS impact. |
| Secure cookie attributes? | `Secure`, `HttpOnly`, `SameSite`, tight `Path` and `Domain`, `__Host-` prefix. |
| What's clickjacking? | Framing your site invisibly over a decoy; `frame-ancestors` in CSP or `X-Frame-Options`. |
| Mass assignment? | Binding request fields straight onto a model so `is_admin=true` sticks; allowlist fields. |
| Race condition in a web app? | Two requests redeem one coupon; use database constraints and atomic operations, not check-then-act. |
| How does a WAF help and where does it fail? | Blocks known patterns at the edge, buys time; fails on logic bugs, encoded payloads, and false positives that teams then disable. |
| What's a bug bounty's advantage over a pentest? | Many testers with varied skills, paid on results, continuous; the pentest gives scoped coverage and a report. |
| What is SAST vs DAST vs SCA? | Static code analysis, running-app testing, dependency vulnerability scanning; you need all three plus humans. |
| Explain a prototype pollution or a request smuggling bug in one line. | Pollution: writing to `__proto__` changes every object; smuggling: front and back ends disagree on where a request ends. |

[← Block 04](README.md) · [Auth and identity](auth-and-identity.md) · [Crypto and TLS](crypto-and-tls.md)
