# 05 · Networking

[← Security fundamentals](../04-security-fundamentals/README.md) · [Index](../README.md) · [Next: Linux →](../06-linux/README.md)

Jane Street's network engineers get asked to list every TCP header field and explain BGP local preference ([Glassdoor](https://www.glassdoor.sg/Interview/Jane-Street-Interview-E255549-RVW81660298.htm)); its support engineers get openssl, certificates and the TCP handshake ([Glassdoor](https://static.glassdoor.fr/Interview/technical-support-engineer-interview-questions-SRCH_KO0,26_IP939.htm)). A security candidate won't get CCNP depth, but the same interviewers will go one layer below the obvious. Tags: [JS] Jane Street, [DM] Miessler, [TW] tadwhitaker, [GN] gracenolan, [GD] Glassdoor.

---

## The questions that come up everywhere

<details>
<summary><b>What happens when you type a URL and press enter?</b> · [DM] [GD Meta] [GD cloud security loops]</summary>

Give it as a chain and name one attack and one control at each hop. Browser checks HSTS and cache. DNS: stub resolver, OS cache, recursive resolver, root, TLD, authoritative (attack: spoofing and cache poisoning; control: DNSSEC, DoH). ARP for the gateway's MAC (attack: ARP spoofing; control: dynamic ARP inspection). TCP three-way handshake (attack: SYN flood; control: SYN cookies). TLS handshake (attack: stripping, rogue CA; control: HSTS, CT, pinning). HTTP request with cookies (attack: CSRF, XSS; control: SameSite, CSP). Server side: load balancer, app, database. Response rendered. If they want depth, go down to Ethernet framing, MTU and the routing decision at each hop.
</details>

<details>
<summary><b>TCP three-way handshake, and what is SYN/ACK?</b> · [TW] [GD Google] [JS support loop]</summary>

Client sends SYN with a random initial sequence number; server replies SYN-ACK (its own sequence number, acknowledging the client's plus one); client sends ACK. Both sides now have agreed sequence numbers and window sizes, and options like MSS, SACK and window scaling. Security angles: SYN floods exhaust the half-open table, fixed by SYN cookies; predictable sequence numbers allowed blind spoofing in the 1990s; a half-open scan (`nmap -sS`) sends SYN and never completes, so it's quieter than a full connect (`-sT`), which is a question in the tadwhitaker list.
</details>

<details>
<summary><b>TCP versus UDP. Which is more secure?</b> · [GD Google] [TW] [GN] [Citadel: "in the context of HFT"]</summary>

TCP is connection-oriented, ordered, reliable, with flow and congestion control; UDP is a datagram with none of that, so it's faster and simpler. Neither is "secure"; security comes from the layer above (TLS, QUIC, DTLS). What differs is abuse: UDP is spoofable because there's no handshake, so it carries amplification DDoS (DNS, NTP, memcached). TCP's handshake ties a source to a connection, which is why most scanners and firewalls reason about state. In trading, multicast UDP carries market data because latency beats reliability and the feed can be re-requested; order entry uses TCP because you can't lose an order.
</details>

<details>
<summary><b>List the TCP header fields and what each is for.</b> · [JS Network Engineer]</summary>

Source port, destination port (16 bits each); sequence number; acknowledgement number; data offset (header length); reserved bits; flags (CWR, ECE, URG, ACK, PSH, RST, SYN, FIN); window size (flow control); checksum; urgent pointer; options (MSS, window scale, SACK permitted, SACK blocks, timestamps, which also give PAWS). Security hooks: RST injection to kill connections, flags combinations used by scanners (XMAS, NULL, FIN scans), timestamps leaking uptime, window size fingerprinting the OS.
</details>

<details>
<summary><b>The OSI model: what gets added at each layer?</b> · [TW] [GN] [JS Network Engineer]</summary>

7 application (HTTP, DNS data), 6 presentation (TLS sits around here in practice), 5 session, 4 transport (TCP/UDP header: ports, sequence numbers), 3 network (IP header: addresses, TTL), 2 data link (Ethernet frame: MACs, VLAN tag, FCS), 1 physical (bits on the wire). Encapsulation: each layer wraps the one above. The interviewer's follow-up is usually "which layer is X" (ARP: between 2 and 3; ICMP: 3; BGP: application over TCP 179; a switch: 2; a router: 3).
</details>

<details>
<summary><b>How does DNS work, and why do security teams care so much?</b> · [GN] [DM] [GD]</summary>

Stub resolver asks a recursive resolver; it walks root, TLD and authoritative servers and caches by TTL. Reverse lookups use `in-addr.arpa` with the octets reversed. Security cares because almost every attack touches DNS: malware resolves C2 domains, DGA and fast-flux rotate them, DNS tunnelling exfiltrates data in long subdomain labels at a volume no HTTP log sees, and newly registered domains are a strong phishing signal. Controls: log every query, a resolver with blocking (RPZ), DNSSEC for integrity, DoH/DoT for privacy with the trade-off that it blinds your own monitoring unless you force clients to your resolver.
</details>

<details>
<summary><b>How does traceroute work at the protocol level? What port does ping use?</b> · [DM] [TW] [GN]</summary>

Traceroute sends probes to the final destination with TTL 1, 2, 3 and so on; each router that decrements TTL to zero returns an ICMP Time Exceeded, which reveals the hop. Linux uses UDP probes to high ports by default, Windows uses ICMP Echo, and `tcptraceroute` uses TCP SYN to get through firewalls. Ping has no port: ICMP is a layer 3 protocol. Say both of those unprompted; Miessler calls them trick questions that catch people who've only read about networks.
</details>

<details>
<summary><b>Miessler's onion: how many packets leave the NIC to complete a traceroute to twitter.com after you plug in the cable?</b> · [DM]</summary>

Walk it: DHCP (discover, offer, request, ack), ARP for the gateway, DNS for the resolver (possibly ARP first), DNS query and response, then the traceroute probes themselves: typically three per TTL value for however many hops, each triggering an ICMP reply. The number isn't the point; naming every layer in order is. Add: if DoH is on, the DNS step is a TLS handshake too.
</details>

<details>
<summary><b>ARP: TCP or UDP? How does ARP spoofing work?</b> · [TW] [GN]</summary>

Neither. ARP is its own layer 2 protocol carried directly in Ethernet frames, mapping IPs to MACs with a broadcast "who has". Spoofing: send unsolicited replies claiming the gateway's IP maps to your MAC, and the LAN sends you its traffic. Defences: dynamic ARP inspection with DHCP snooping on switches, static entries for critical hosts, 802.1X so strangers can't join, and encryption above it so interception yields little.
</details>

<details>
<summary><b>What does a VPN hide and what does it expose? Why doesn't chaining seven proxies help?</b> · [GN]</summary>

A VPN hides traffic contents and destinations from the local network and your source IP from the destination; it exposes everything to the VPN provider, and DNS or IPv6 leaks, timing, browser fingerprints and account logins still identify you. Chaining proxies adds hops but every hop is a party that can log, and correlation of timing and volume at the ends defeats it, which is why Tor's design (layered encryption, no single hop knowing both ends) is the serious version and even Tor is visible on a corporate network by its entry nodes and traffic shape.
</details>

## Infrastructure questions

<details>
<summary><b>What is BGP, what is local preference, and why does it have an open message?</b> · [JS Network Engineer] [GD Meta] [GN]</summary>

BGP is the path-vector protocol between autonomous systems; it decides how the internet routes. Local preference is the attribute an AS sets on routes learned from neighbours to say which exit it prefers for outbound traffic; higher wins, it's compared before AS path length, and it only matters inside your AS. The OPEN message exists because the TCP handshake only proves a socket works; OPEN carries the BGP version, AS number, hold time and capabilities, and the peer must agree before routes flow. Security angle: BGP hijacks (announcing someone else's prefix) and route leaks; RPKI with route origin validation, prefix filters, max-prefix limits, and TCP-AO or MD5 on sessions.
</details>

<details>
<summary><b>Subnets, masks, router versus switch, broadcast and collision domains.</b> · [TW] [GN]</summary>

A subnet mask splits an address into network and host bits; `/24` means 256 addresses, 254 usable. Subnets limit broadcast domains and let you put firewalls between zones, which is the security use: trading, corporate, guest and management networks shouldn't share a broadcast domain. A switch forwards frames by MAC (layer 2, one broadcast domain per VLAN); a router forwards packets by IP between networks (layer 3) and is where ACLs live. Collision domains are a hub-era idea: a full-duplex switch port is its own. CAM table overflow floods a switch with fake MACs until it fails open and broadcasts everything, so port security and MAC limits matter.
</details>

<details>
<summary><b>Firewalls, IDS versus IPS, and filtered versus closed ports.</b> · [TW] [DM]</summary>

A firewall enforces policy on packets or connections (stateless ACL, stateful, next-gen with application awareness; in cloud, security groups and NACLs). An IDS watches and alerts; an IPS sits inline and blocks, which means it can also cause outages. Filtered means the firewall dropped the probe silently; closed means the host answered RST. Miessler's question: do you prefer filtered or closed? Filtered hides what exists and slows scans; closed is honest and makes your own troubleshooting easier. A mature answer says drop by default on the internet edge, reject inside the network where you want fast failure, and that obscurity isn't a control, only a delay.
</details>

<details>
<summary><b>IPsec phase 1 versus phase 2.</b> · [TW]</summary>

Phase 1 (IKE SA) authenticates the peers (PSK or certificates) and sets up a secure channel for negotiating; phase 2 (IPsec SAs, child SAs) negotiates the actual tunnel keys and traffic selectors, one per direction, re-keyed often. Say that WireGuard replaced most of this for new deployments with a fixed modern cipher set.
</details>

<details>
<summary><b>How would you detect a DDoS? A port scan? Brute force from the network side?</b> · [TW] [GN]</summary>

DDoS: traffic volume and packet rate far above baseline, many sources, one destination or one URL, SYN backlog filling, upstream telling you first; mitigate with scrubbing or an anycast CDN, SYN cookies, rate limits. Port scan: one source touching many ports or many hosts in a short window, lots of SYNs without ACKs, RSTs back; netflow and firewall logs show it. Brute force: many authentication failures per source or per account per minute; the detection logic is in [block 03](../03-coding/README.md).
</details>

<details>
<summary><b>Draw a network, then find where an issue happened. How would you troubleshoot intermittent latency between two data centres?</b> · [TW] [Citadel]</summary>

Draw it with trust zones and the path a packet takes. Then bisect: is it one host or many, one direction or both, one time of day or always. Tools along the path: `ping` and `mtr` for loss per hop, `ss -ti` for retransmits and RTT on the socket, interface counters for errors and drops, QoS and buffer stats on switches, and a packet capture at both ends to see where the gap is. In the security version of the question the "issue" is a bad firewall rule or an asymmetric route breaking a stateful device.
</details>

<details>
<summary><b>How would you reverse-engineer a custom protocol from a packet capture?</b> · [TW]</summary>

Separate conversations, look at the first bytes of each message for magic numbers and length fields, compare messages that differ by one known action, watch for lengths that match the payload, look for structure (type, length, value), check whether it's encrypted (entropy), and use the client binary if you have it. Write a dissector as you go and confirm by replaying.
</details>

## Rapid-fire

| Question | One-line answer |
| --- | --- |
| Port ranges? | 0-1023 well known, 1024-49151 registered, 49152-65535 dynamic. |
| Ports worth knowing and their risks? | 22 SSH (brute force, keys), 23 Telnet (plaintext, kill it), 25/587 SMTP (relay, spoofing), 53 DNS (tunnelling, amplification), 80/443 web, 88 Kerberos, 135/139/445 RPC/SMB (lateral movement, EternalBlue), 389/636 LDAP, 3389 RDP (never on the internet), 3306/5432 databases (bind to localhost). |
| DNS port? | 53 UDP for queries, TCP for zone transfers and large responses; 853 for DoT, 443 for DoH. |
| What's in an HTTP request? | Method, path, version, headers (Host, Cookie, Authorization, User-Agent, Origin, Referer), body. Security-relevant response headers: CSP, HSTS, Set-Cookie flags, X-Content-Type-Options. |
| DHCP handshake? | Discover, Offer, Request, Acknowledge; rogue DHCP hands out a malicious gateway or resolver. |
| How does TCP react to packet loss? | Retransmit on timeout or three duplicate ACKs, halve the congestion window, grow back slowly. |
| How does a packet travel between two hosts on the same subnet? | ARP for the destination MAC, frame straight to it through the switch, no router involved. |
| What is 802.1X? | Port-based network access control: authenticate the device before the switch port carries traffic. |
| Why is IRC relevant to botnets? | Early C2 channel: cheap, broadcast to all bots; modern C2 hides in HTTPS, DNS, and cloud services. |
| What's an amplification attack? | Small spoofed requests to UDP services that reply with large responses to the victim; fix with source validation (BCP 38) and rate limits. |

[← Security fundamentals](../04-security-fundamentals/README.md) · [Index](../README.md) · [Next: Linux →](../06-linux/README.md)
