# 07 · Windows, Active Directory and endpoints

[← Linux](../06-linux/README.md) · [Index](../README.md) · [Next: Cloud and supply chain →](../08-cloud-and-supply-chain/README.md)

Jane Street asked a London cyber security engineer what they'd change on a fresh Windows install rolled out across the firm ([Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm)), and its IT Security postings want AD, Entra ID and PKI depth ([Greenhouse](https://job-boards.greenhouse.io/janestreet/jobs/8155302002)). The Windows event IDs below are the single most repeated topic in SOC and detection interviews ([epicdetect](https://epicdetect.io/blogs/soc-analyst-interview-questions), [rootguard](https://rootguard.gitbook.io/cyberops/resources-hub/blogs/posts-field-notes/soc-prep-interview-questions)). Tags: [JS] Jane Street, [GN] gracenolan, [TW] tadwhitaker, [DM] Miessler, [GD] Glassdoor, [SOC] SOC question banks.

---

## The Jane Street question

<details>
<summary><b>You're rolling Windows out across the company. What do you change on a brand-new, out-of-the-box install?</b> · [JS Cyber Security Engineer, London]</summary>

Say Windows 11 (Windows 10 left support in October 2025), then group it:

**Identity and privilege.** Nobody works as local admin; LAPS rotates the local admin password per machine; tiered admin model so domain admin credentials never touch workstations; Protected Users group for privileged accounts.

**Credentials in memory.** Credential Guard, LSA protection (RunAsPPL), no WDigest, restrict NTLM where you can, Kerberos armouring.

**Disk and boot.** BitLocker with TPM and recovery keys escrowed, Secure Boot, firmware passwords.

**Attack surface.** Remove bloatware and consumer apps, disable LLMNR, NetBIOS over TCP, SMBv1, WPAD, PowerShell v2, Office macros from the internet, unsigned scripts; attack surface reduction rules; AppLocker or WDAC allowlisting where the role allows.

**Visibility.** EDR, Sysmon with a tuned config, PowerShell script block and module logging, process creation (4688) with command line, central log forwarding, Defender tamper protection.

**Updates and config.** Automatic updates with a ring model, a hardened baseline (CIS or Microsoft's) applied by Group Policy or Intune, firewall on with inbound denied, RDP off unless needed and then NLA plus MFA.

Then the judgement the posting asks for: a trader's desktop can be locked to an allowlist; a developer's machine needs room, so you go heavier on detection there. Say you'd pilot on one team, measure the help-desk load, then roll out.
</details>

## Event IDs and detection

<details>
<summary><b>Which Windows event IDs matter for login monitoring, and what do 4624 and 4625 tell you?</b> · [SOC, the most repeated]</summary>

4624 successful logon, 4625 failed logon, 4634/4647 logoff, 4648 logon with explicit credentials (runas, lateral movement), 4672 special privileges assigned (admin logon), 4720 account created, 4728/4732 added to a privileged group, 4740 lockout, 4768 Kerberos TGT request, 4769 Kerberos service ticket request, 4771 Kerberos pre-auth failed, 4776 NTLM validation, 1102 audit log cleared, 4688 process creation, 7045 new service installed, 4698 scheduled task created, 4104 PowerShell script block.

Inside 4624 the fields that matter: logon type (2 interactive, 3 network, 4 batch, 5 service, 7 unlock, 8 cleartext network, 9 new credentials, 10 remote interactive, 11 cached), authentication package (Kerberos versus NTLM), source IP and workstation, target user, logon ID for correlating later events, elevated token.
</details>

<details>
<summary><b>How do you detect pass-the-hash?</b> · [GD: "what is pass the hash"] [SOC]</summary>

4624 with logon type 3 and authentication package NTLM, in an environment where Kerberos is normal, from a source that doesn't normally talk to that host, often as a privileged account. Correlate with 4648 on the source and 4688 command lines containing `sekurlsa::pth` or similar on the source. On domain controllers, a spike of 4776 from an unusual source. Prevent: Credential Guard, restrict NTLM, tiered admin, LAPS, Protected Users, and no admin logons to workstations.
</details>

<details>
<summary><b>What is Kerberoasting, how is it used, and how do you detect it?</b> · [GD: asked directly] [GN]</summary>

Any domain user can request a service ticket for any account with a service principal name; the ticket is encrypted with that account's password hash, so the attacker takes it offline and cracks it. Detection: 4769 with RC4 encryption (type 0x17) in a mostly AES domain, one account requesting tickets for many services in a short time, requests for honeypot SPNs. Prevent: long random passwords for service accounts or group managed service accounts, AES-only, remove unnecessary SPNs. Related: AS-REP roasting hits accounts with pre-authentication disabled (4768 with pre-auth type 0).
</details>

<details>
<summary><b>Golden and silver tickets, Mimikatz, DCSync.</b> · [GN]</summary>

A golden ticket is a forged TGT signed with the `krbtgt` account hash: domain-wide access that survives password resets until `krbtgt` is rotated twice. A silver ticket is a forged service ticket using one service account's hash: narrower, and it never touches the DC, so it's quieter. Mimikatz is the tool that dumps credentials from LSASS and forges tickets. DCSync abuses replication rights to pull hashes from a DC without touching LSASS (detect with 4662 showing replication GUIDs from a non-DC account). Defence: protect LSASS, limit who has replication rights, monitor 4672 for domain admins logging into odd systems.
</details>

<details>
<summary><b>How would you investigate a PowerShell alert, a scheduled-task alert, or an EDR malware alert?</b> · [SOC] [JS scenario round: malware on a corporate system]</summary>

PowerShell: get the full command (4104 script block, 4688 with command line), decode `-enc` base64 (UTF-16LE), look for `IEX`, `DownloadString`, `-nop -w hidden`, then the parent process (Office or a browser spawning PowerShell is the classic phishing chain) and any outbound connection. Scheduled task: 4698 shows the creator and the action; check what binary it runs, from where, and whether the creating process makes sense. EDR malware: isolate the host, keep it powered on, pull the process tree and network connections, hash the file and check it, scope by searching the hash and the C2 domain across the fleet, then recover the host from a known-good image. Full playbooks in [block 09](../09-incident-response/README.md).
</details>

## Active Directory and platform

<details>
<summary><b>How does Active Directory work, and how does LDAP relate?</b> · [TW] [GN]</summary>

AD is a directory of users, computers, groups and policy, replicated across domain controllers, with Kerberos for authentication and LDAP as the query protocol. Group Policy pushes configuration to machines and users by OU. LDAP is the wire protocol; AD is one implementation with Windows-specific schema and Kerberos on top. Attack surface: over-privileged groups, stale accounts, unconstrained delegation, ACL misconfigurations that BloodHound maps, LDAP signing and channel binding off, certificate services misconfigurations (ESC1 and friends).
</details>

<details>
<summary><b>How do the registry and Group Policy relate? What is SMB and why does it matter?</b> · [GN]</summary>

Group Policy is mostly registry settings delivered centrally; persistence and hardening both end up as registry keys, which is why Run keys, services and policies are hunt locations. SMB is the file and printer sharing protocol; it carries lateral movement (PsExec, admin shares), credential relay when signing is off, and historic worms (EternalBlue on SMBv1). Enforce SMB signing, kill SMBv1, block 445 between workstations.
</details>

<details>
<summary><b>EDR versus antivirus, and what's an attack surface reduction rule?</b> · [SOC]</summary>

Antivirus matches files to signatures at write or execute time. EDR records behaviour continuously (process trees, network, registry, memory) and lets you query, isolate and respond across the fleet; it catches living-off-the-land activity that has no malicious file. ASR rules are Defender policies that block whole behaviours: Office spawning child processes, scripts launching downloaded executables, credential stealing from LSASS.
</details>

<details>
<summary><b>A user brings you a PC that's acting weird and you suspect malware. What do you do?</b> · [TW]</summary>

Don't reboot or "clean" it yet. Take it off the network but keep it on, note what the user saw and when, capture volatile data (processes, connections, logged-on users), then disk and memory images if it looks real. Triage: autoruns and scheduled tasks, recent files, browser downloads, Office documents with macros, new services, the EDR timeline. Decide: false alarm, commodity adware, or something targeted. If targeted, it becomes an incident; if not, reimage anyway, because cleaning in place is how you miss persistence. Tell the user what happened in plain words.
</details>

<details>
<summary><b>macOS: what should I know?</b> · [GN]</summary>

Gatekeeper and notarisation, System Integrity Protection, TCC prompts for camera and files, FileVault, the Secure Enclave for keys and Touch ID, MDM for configuration, Endpoint Security framework for EDR. Historic lesson: `goto fail` (2014), a duplicated line that skipped signature verification in TLS, the argument for tests and code review of crypto paths. Persistence lives in LaunchAgents and LaunchDaemons, login items and configuration profiles.
</details>

## Rapid-fire

| Question | One-line answer |
| --- | --- |
| Logon type 10 versus 3? | 10 is RDP (remote interactive); 3 is a network logon such as SMB or WinRM. |
| What is 1102? | Security log cleared: an attacker cleaning up, nearly always worth an alert. |
| What is LAPS? | Unique, rotated local admin passwords per machine stored in AD or Entra, so one stolen local admin hash doesn't open the fleet. |
| Why disable LLMNR and NetBIOS? | They let Responder poison name lookups and capture NTLM hashes on the LAN. |
| What's NTLM relay? | Forwarding a captured NTLM authentication to another service in real time; SMB and LDAP signing stop it. |
| Constrained versus unconstrained delegation? | Unconstrained lets a server impersonate anyone who connects to it to any service; constrained limits the targets. |
| What is WDAC versus AppLocker? | Both allowlist executables; WDAC is kernel-enforced and harder to bypass, AppLocker is simpler to deploy. |
| Sysmon event 1, 3, 10, 11? | Process create, network connection, process access (LSASS reads), file create. |
| What is device attestation? | TPM-signed proof of boot state and configuration, used by conditional access to trust a device. |
| Entra ID conditional access? | Policy engine that gates sign-in on user, device health, location and risk, where MFA and device trust get enforced. |

[← Linux](../06-linux/README.md) · [Index](../README.md) · [Next: Cloud and supply chain →](../08-cloud-and-supply-chain/README.md)
