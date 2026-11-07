# 04b · Cryptography, TLS and PKI

[← Block 04](README.md) · [Auth and identity](auth-and-identity.md) · [Web and AppSec →](web-and-appsec.md)

These are the most repeated fundamentals questions across every source. The Jane Street security loops reported crypto questions directly (the HK May 2023 loop asked what makes a password strong, how a password manager derives its key, and how TOTP works, [block 01](../01-process/README.md)); treat crypto and TLS as core, not adjacent. Tags: [DM] Daniel Miessler, [TW] tadwhitaker collection, [GN] gracenolan notes, [GD] Glassdoor reports.

---

## Core concepts

<details>
<summary><b>Encoding, encryption, hashing, signing, obfuscation: what's the difference?</b> · [DM] [GN] (asked in nearly every security loop)</summary>

Encoding changes representation so data survives a channel (base64, URL encoding); anyone can reverse it, so it is not a security control. Encryption gives confidentiality and reverses only with the key. Hashing is one-way and fixed-length; it gives integrity and lets you store a password without keeping it. Signing is a hash encrypted with a private key: anyone with the public key can check integrity and origin, and the signer can't deny it. Obfuscation makes something hard to read without a key, which buys time, not security. The follow-up they want: a MAC also gives integrity, but both sides share the key, so it can't prove who sent it.
</details>

<details>
<summary><b>Symmetric versus asymmetric: how do they differ and when is each used?</b> · [DM] [TW] [GN]</summary>

Symmetric uses one shared key (AES, ChaCha20): fast, so it carries bulk data, but the problem is getting the key to the other side. Asymmetric uses a key pair (RSA, elliptic curve): slow, but you can publish one key, so it solves key distribution, signing and identity. Real systems combine them: TLS and SSH use asymmetric crypto to agree a symmetric session key, then encrypt traffic symmetrically. The tadwhitaker list calls it "synchronous and asynchronous" encryption; it's the same question.
</details>

<details>
<summary><b>In public-key crypto, which key encrypts and which key signs?</b> · [DM]</summary>

Encrypt with the recipient's public key so only their private key can open it. Sign with your own private key so anyone with your public key can verify. Mixing those up is a common interview slip, so say it slowly.
</details>

<details>
<summary><b>Block versus stream ciphers, and what are cipher modes?</b> · [DM] [GN]</summary>

A block cipher encrypts fixed blocks (AES: 16 bytes) and needs a mode to handle longer data; a stream cipher produces a keystream XORed with the data (ChaCha20) and suits unknown lengths. Modes: ECB encrypts each block independently, so identical plaintext blocks give identical ciphertext (the penguin picture), never use it. CBC chains blocks with an IV and needs a separate MAC, and padding-oracle bugs come from getting that wrong. CTR turns a block cipher into a stream cipher. GCM is CTR plus authentication in one pass, which is why AES-GCM and ChaCha20-Poly1305 are the defaults: you get confidentiality and integrity together (AEAD).
</details>

<details>
<summary><b>What is an IV, and what happens if you reuse one?</b> · [DM]</summary>

A per-message value mixed in so the same plaintext under the same key gives different ciphertext. For CBC it must be unpredictable; for CTR and GCM it's a nonce that must never repeat with the same key. Reusing a GCM nonce leaks the XOR of the two plaintexts and lets an attacker forge authentication tags. That's why long-lived keys get rotated and nonces come from a counter or 96 random bits with a key budget.
</details>

<details>
<summary><b>How does HMAC work, and why is it built that way?</b> · [TW] [GD Google: "what is an HMAC"]</summary>

`HMAC(k, m) = H((k ⊕ opad) ‖ H((k ⊕ ipad) ‖ m))`: hash the message with an inner key, then hash that result with an outer key. The nested structure is there because a plain `H(key ‖ message)` with Merkle-Damgård hashes (MD5, SHA-1, SHA-2) suffers a length-extension attack: from a valid tag you can append data and compute a new valid tag without the key. HMAC also makes the construction provably secure even when the hash has weaknesses. Use it for API request signing, cookie integrity, TOTP.
</details>

<details>
<summary><b>Explain the length-extension attack.</b> · [est] [classic follow-up to HMAC]</summary>

MD5, SHA-1 and SHA-256 are Merkle-Damgård constructions: they process the message in blocks and the final output *is* the internal state after the last block. So if you know `H(secret ‖ message)` and the length of `secret ‖ message`, you can set the hash's internal state to that output and keep hashing, producing a valid `H(secret ‖ message ‖ padding ‖ extra)` without ever knowing the secret. A web app that authenticates `?user=bob&role=user&sig=H(secret‖params)` lets an attacker append `&role=admin` and forge the signature (tools like hash_extender automate it). Fixes: HMAC, or a hash that does not expose its state (SHA-3, BLAKE2, or the truncated SHA-512/256). The interview point: this is why "just hash it with a secret" is not a MAC, and why you name HMAC without being asked.
</details>

<details>
<summary><b>Timing and other side channels, and why `==` on a secret is a bug.</b> · [est]</summary>

A side channel leaks information through *how* a computation runs rather than its output: time, power, cache behaviour, error messages. The everyday one: comparing a submitted token to the real one with `==` returns as soon as the first byte differs, so response time reveals how many leading bytes were right, and an attacker recovers the token byte by byte over the network. Fix: constant-time comparison (`hmac.compare_digest` in Python, `crypto/subtle` in Go). Others to name: padding-oracle attacks (a different error for bad padding versus bad MAC breaks CBC, which is why you MAC-then-verify before touching the ciphertext and why AEAD modes exist), Spectre and Meltdown as cache side channels across privilege boundaries ([block 08](../08-cloud-and-supply-chain/README.md)), and RSA key recovery from timing in early OpenSSL. Say that cryptographic code is one of the few places where "it returns the right answer" is not the whole specification.
</details>

<details>
<summary><b>What's the difference between a MAC and a digital signature?</b> · [GN]</summary>

Both give integrity. A MAC uses a shared secret, so either side could have produced it: fast, no non-repudiation. A signature uses a private key only the signer holds, so it proves origin to anyone with the public key. Pick a MAC inside one system and a signature across trust boundaries.
</details>

<details>
<summary><b>If you must compress and encrypt, which first?</b> · [DM] [TW]</summary>

Compress first. Good ciphertext looks random and won't compress. Then the twist: compressing attacker-influenced data next to secrets leaks the secret's length (CRIME and BREACH did this to TLS and HTTP), so in a protocol you may choose not to compress at all.
</details>

<details>
<summary><b>How do PRNGs and entropy work, and what breaks them?</b> · [GN]</summary>

A cryptographic PRNG stretches a seed from true entropy (hardware noise, interrupt timing) into an unpredictable stream; `/dev/urandom` and `getrandom()` on Linux, `CryptGenRandom` on Windows. Breakage: seeding with time or a PID, using `random` instead of `secrets` in Python, cloned VMs that boot with identical state, and weak key generation like the eMudhra incident in October 2026, where intermediate CA keys came out of a bad process with values below 256 ([BleepingComputer](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates)).
</details>

## Key exchange and forward secrecy

<details>
<summary><b>How do two parties build a shared secret over a public channel?</b> · [DM]</summary>

Diffie-Hellman. Each side picks a private number, sends `g^a mod p` (or a point multiple on an elliptic curve for ECDH), and both compute the same `g^ab`. An eavesdropper sees `g^a` and `g^b` but can't get `g^ab` without solving the discrete log. Modern TLS uses X25519.
</details>

<details>
<summary><b>What attack is plain Diffie-Hellman vulnerable to?</b> · [DM]</summary>

Man-in-the-middle, because neither side is authenticated: the attacker runs one exchange with each party and relays. TLS fixes it by signing the handshake with the server's certificate key; SSH fixes it with host keys you've seen before.
</details>

<details>
<summary><b>Diffie-Hellman versus RSA?</b> · [DM] [TW]</summary>

DH is a key-agreement protocol, RSA is an encryption and signature algorithm. RSA key exchange (client encrypts a secret to the server's public key) needs the server's key material in advance and gives no forward secrecy; DH needs no prior key but no authentication either. TLS 1.3 dropped RSA key exchange and keeps RSA and ECDSA only for signing.
</details>

<details>
<summary><b>If the server's private key is stolen, can recorded traffic be decrypted? What is forward secrecy?</b> · [DM] [TW] [GN]</summary>

With RSA key exchange, yes: the session secret was encrypted to that key, so a recording plus the key opens everything. With ephemeral DH (DHE, ECDHE) each session uses fresh throwaway values, so the long-term key only signs the handshake and a later theft reveals nothing past. That's forward secrecy; TLS 1.3 makes it mandatory. Signal extends it with a ratchet so even a session key leak covers only a few messages.
</details>

## TLS and PKI

<details>
<summary><b>Describe the TLS handshake.</b> · [DM] [TW] [GN] [GD Google, Amazon, detection loops]</summary>

TLS 1.3: ClientHello carries supported versions, cipher suites, SNI and the client's ECDHE key share. ServerHello picks the suite and sends its key share; both derive the handshake keys, so everything after this is already encrypted. The server sends its certificate chain, a CertificateVerify (a signature over the transcript with the certificate's private key, which is the authentication step) and Finished (an HMAC over the transcript). The client validates the chain to a trusted root, checks the hostname, verifies the signature, sends its Finished, and application data flows under keys derived from the shared secret. One round trip; 0-RTT resumption exists at the cost of replay. TLS 1.2 did the same in two round trips with the key exchange after the certificate.
</details>

<details>
<summary><b>Does TLS use symmetric or asymmetric encryption?</b> · [DM]</summary>

Both. Asymmetric to authenticate the server and agree the key, symmetric (AES-GCM or ChaCha20-Poly1305) for the data because it's a thousand times faster.
</details>

<details>
<summary><b>How do certificates work, and how is the padlock produced?</b> · [TW] [GN]</summary>

A certificate binds a public key to a name, signed by a CA. The browser checks the chain of signatures up to a root in its trust store, the validity dates, the hostname against the SAN list, and revocation (OCSP stapling, CRLSets). The padlock means "this connection is to a party holding the private key for a certificate a CA vouched for", nothing about whether the site is honest. Depth: Certificate Transparency logs let domain owners spot misissuance; CAA records say which CAs may issue; DigiNotar (2011) and the eMudhra weak keys (2026) are the examples of what a bad CA costs.
</details>

<details>
<summary><b>How has TLS been attacked?</b> · [DM] [GN]</summary>

Heartbleed (an OpenSSL bounds bug leaking memory, including keys), BEAST (CBC IV chaining in TLS 1.0), POODLE (SSL 3.0 padding), CRIME and BREACH (compression leaking secrets), Logjam and FREAK (export-grade DH and RSA downgrades), ROBOT (Bleichenbacher padding oracle against RSA key exchange), downgrade attacks in general. The lesson they want: most were fixed by removing options, which is why TLS 1.3 has five cipher suites and no RSA key exchange, no compression, no renegotiation.
</details>

<details>
<summary><b>SSL, TLS or HTTPS: which is most secure?</b> · [DM] (a trick)</summary>

They aren't comparable. SSL is the dead predecessor of TLS, TLS is the protocol, HTTPS is HTTP carried over TLS. The honest answer is "TLS 1.3, and the question is a category error". Keep a straight face.
</details>

<details>
<summary><b>Should you encrypt all data at rest? How do you authenticate that a message came from me?</b> · [TW]</summary>

At rest: full-disk or volume encryption is cheap and defends against lost hardware and disposal, so yes by default, but it does nothing against an attacker who logs in, so field-level encryption with separate keys is the control for the data that matters, and key management is the real work (KMS, envelope encryption, rotation, who can call decrypt). Message origin: sign it with a key only you hold, or use a MAC with a key only we share, and include a nonce or timestamp so it can't be replayed.
</details>

<details>
<summary><b>How does SSH use asymmetric crypto to set up a symmetric session?</b> · [GN] [GD Amazon: "how does ssh work"]</summary>

Both sides run an ECDH exchange to agree a session key; the server signs the exchange with its host key, which the client checks against `known_hosts` (trust on first use, or an SSH CA). The user then authenticates inside the encrypted channel, ideally by signing a challenge with their private key, and the session runs under symmetric encryption with per-direction keys and MACs. Hardening is in [block 06](../06-linux/README.md).
</details>

## Rapid-fire

| Question | Answer in one line |
| --- | --- |
| Examples of symmetric algorithms? | AES, ChaCha20; historical: DES, 3DES, RC4, Blowfish. |
| Asymmetric? | RSA, ECDSA, Ed25519, Diffie-Hellman, X25519, ElGamal. |
| Hash functions you'd use today? | SHA-256, SHA-3, BLAKE2/3 for integrity; argon2id, bcrypt, scrypt for passwords; never MD5 or SHA-1 for security. |
| Why is MD5 broken? | Practical collisions since 2004; chosen-prefix collisions mean two different files with the same hash, so no good for signatures or certificates. |
| What is a padding oracle? | A server that reveals whether decrypted padding was valid lets an attacker decrypt CBC ciphertext byte by byte. |
| What is envelope encryption? | Encrypt data with a random data key, encrypt that key with a master key in a KMS; rotate the master without re-encrypting the data. |
| What is a TPM / Secure Enclave for? | Hardware that holds keys and performs operations so the key never enters main memory; disk encryption, device attestation, passkeys. |
| What does certificate pinning do? | Trust only a specific key or CA for a host, defeating rogue CAs at the cost of operational pain on rotation. |
| Post-quantum? | Shor's algorithm breaks RSA and ECC; ML-KEM (Kyber) is already in TLS hybrids; symmetric crypto just needs longer keys. |
| What is nonce misuse resistance? | AEADs like AES-GCM-SIV or XChaCha20 that degrade gracefully if a nonce repeats. |

[← Block 04](README.md) · [Auth and identity](auth-and-identity.md) · [Web and AppSec →](web-and-appsec.md)
