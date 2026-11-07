# 02b · Windows-centric walkthrough, narrated

[← Log investigation](README.md) · [Linux log review →](linux-log-review.md)

The Jane Street online assessment used Windows workstation logs (your own run had `C:\Program Files\Chrome.exe` and `C:\tmp\stage2.exe` in it), a recruiter described the Security Engineer role as mostly Windows-focused ([Blind](https://www.teamblind.com/post/jane-street-security-engineer-krwvr2zb)), and SOC-interview candidates report practising Windows Security event IDs 4624, 4625, 4634, 4688, 1102 plus parent-child process correlation before the live round ([Reddit](https://reddit.sentinel-team.org/posts/1sd96lb/snapshots/2026-04-10T21%3A06%3A06.339702Z)). So the live investigation will most likely be Windows events in a flat file, read with Unix tools. This page is a full narrated run on exactly that, with every command tested against a sample CSV export.

Sample format assumed (an exported Security log with Sysmon-style fields; check `head -2` first, every export differs):

```text
TimeCreated,EventID,Computer,TargetUserName,LogonType,AuthenticationPackageName,IpAddress,ProcessName,CommandLine
```

---

## The event IDs you reach for

| ID | Meaning | Why you grep it |
| --- | --- | --- |
| 4624 | Successful logon | Who was on the box; logon type 2 interactive, 3 network, 10 RDP; NTLM where Kerberos is normal = lateral movement |
| 4625 | Failed logon | Brute force and spray, by target user and source |
| 4688 | Process created (with command line if enabled) | The whole attack chain: parent, child, path, arguments |
| 4698 | Scheduled task created | Persistence |
| 7045 | New service installed (System log) | Persistence and PsExec-style lateral movement (`PSEXESVC`) |
| 4672 | Special privileges assigned | An admin-level logon; on a workstation, a red flag |
| 4648 | Logon with explicit credentials | `runas`, lateral movement with stolen creds |
| 4720 / 4728 / 4732 | Account created / added to privileged group | Persistence through identity |
| 4768 / 4769 / 4771 | Kerberos TGT / service ticket / pre-auth failure | Kerberoasting (4769 with RC4), AS-REP roasting |
| 1102 | Audit log cleared | Anti-forensics; nearly always worth an alert |
| Sysmon 1 / 3 / 10 / 11 / 13 | Process create / network connect / process access / file create / registry set | Richer versions of the above; Sysmon 10 on `lsass.exe` is credential dumping |
| PowerShell 4104 | Script block logged | The decoded script, even when `-enc` hid it |

---

## The narrated run (about 35 minutes)

What you type on the left, what you say on the right. The sample is a phishing chain on WS-09 that moves to a file server.

**Minute 0 to 3, orient.**

```bash
ls -la; wc -l *; head -2 security.csv
cut -d, -f2 security.csv | tail -n +2 | sort | uniq -c | sort -rn
```

> "Before I search for anything I want to know what I've got. One CSV, a header, nine fields. Event ID counts first: mostly 4688 process starts and 4624 logons, which is normal, and then singletons: a 7045 new service, a 4698 scheduled task, a 4672 privileged logon, and a 1102 audit-log clear. The singletons are where I'll start, because an attacker is usually the rare thing."

**Minute 3 to 8, baseline.**

```bash
cut -d, -f3 security.csv | tail -n +2 | sort | uniq -c            # events per host
awk -F, 'NR>1 {if(!($3 in f)) f[$3]=$1; l[$3]=$1} END {for (h in f) print h, f[h], "->", l[h]}' security.csv | sort
awk -F, '$2==4624 {print $5, $6}' security.csv | sort | uniq -c    # logon types and auth packages
```

> "Three hosts. WS-02 is quiet office hours. WS-09 and FILESRV-01 are active around 12:46 to 13:40. Logons are mostly interactive Kerberos, which is what a domain workstation should look like, except two network logons over NTLM to the file server. That's my second lead."

**Minute 8 to 18, find the rare and pivot.**

```bash
awk -F, '$2==4688 && $8 ~ /AppData|\\Temp\\|\\Downloads\\/ {print $1, $3, $4, $8}' security.csv
grep -i -e '-enc' -e 'encodedcommand' security.csv | cut -d, -f1,3,9
echo 'SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQA' | base64 -d | iconv -f UTF-16LE -t UTF-8; echo
awk -F, '$2==7045 || $2==4698 {print $1, $2, $3, $9}' security.csv
awk -F, '$2==4624 && $5==3 && $6=="NTLM" {print $1, $3, $4, $7}' security.csv
```

> "Two binaries ran from user-writable folders on WS-09: `Invoice_Nov.pdf.exe` in Temp, a double extension, forty seconds after Outlook started, then `stage2.exe` in Roaming. Between them, PowerShell with `-nop -w hidden -enc`. The encoded command decodes to `IEX (New-Object`, a download cradle. Then a scheduled task called OneDrive Update that runs stage2 at logon: persistence. Five minutes later, two NTLM network logons to FILESRV-01 from 10.0.0.28, one as jdoe and one as svc_backup, and svc_backup gets special privileges. Then a new service, PSEXESVC. That's lateral movement with a stolen service account. I'd expect 10.0.0.28 to be WS-09; I'll confirm."

**Minute 18 to 25, timeline.**

```bash
awk -F, '$3=="WS-09"' security.csv | sort | cut -d, -f1,2,8
grep -F '10.0.0.28' security.csv | cut -d, -f1,2,3,4
awk -F, '$2==1102' security.csv
```

> "In order: 12:46 jdoe logs in; 12:47 Outlook; 12:47:40 the fake PDF runs; 12:47:43 encoded PowerShell; 12:47:55 stage2; 12:48 persistence; 12:52 lateral movement to the file server as jdoe then as svc_backup; 12:53 admin rights; 12:55 PsExec service; 13:40 the audit log on WS-09 is cleared. The 1102 is the attacker covering tracks, and it tells me anything after 13:40 on WS-09 is missing, not absent. Initial access is a phishing attachment. The thing I most want to know next is how svc_backup's credential was on a workstation."

**Minute 25 to 30, what I didn't have time for, said out loud.**

> "I haven't confirmed 10.0.0.28 is WS-09 (I'd check DHCP or the 4624 source on the workstation). I haven't looked at what svc_backup did on the file server after 12:55: file reads, archives, outbound. I don't know whether stage2 beaconed; I'd want proxy or DNS logs. And I'd check every other host for the same scheduled task name and the same hash."

**Minute 30 to 35, write-up.**

```text
Finding:    Phishing attachment on WS-09 (jdoe) led to malware with persistence, then lateral movement to FILESRV-01 using the svc_backup service account. Critical.
Evidence:   4688 Invoice_Nov.pdf.exe from Temp at 12:47:40 spawned under Outlook; encoded PowerShell decoding to a download cradle; 4698 task "OneDrive Update" running stage2.exe; 4624 type 3 NTLM to FILESRV-01 as svc_backup from 10.0.0.28 at 12:52:30; 4672; 7045 PSEXESVC; 1102 log clear at 13:40.
Scope:      WS-09 confirmed; FILESRV-01 touched with admin rights; svc_backup credential compromised; anything it can reach is in scope until proven otherwise.
Do now:     Isolate WS-09 and FILESRV-01 (keep powered on); disable svc_backup and reset it; preserve logs and memory; block the download domain; search the fleet for the task name and the stage2 hash.
Check next: What svc_backup did after 12:55; outbound connections from both hosts; why a service account could log on to a workstation; the email that carried the attachment and who else got it.
```

And the CEO version in three sentences: "An employee opened a malicious email attachment at 12:47 today. The attacker used it to reach one file server with an administrative account; we've cut both machines off and reset the account. We're now checking what was read on that server and whether anything left the firm, and I'll update you by 16:00."

---

## Other Windows-log tasks that have been asked

| Task | Where | How you'd do it |
| --- | --- | --- |
| Process tree and parent-child correlation from 4688 or Sysmon 1 | SOC interviews ([Reddit](https://reddit.sentinel-team.org/posts/1sd96lb/snapshots/2026-04-10T21%3A06%3A06.339702Z)) | Sysmon 1 carries ParentProcessId and ParentImage; join child to parent on (host, PID) and print the chain. Office or a browser spawning `cmd`, `powershell`, `wscript` or `mshta` is the classic phishing tell. |
| Which Windows event IDs matter for logons, and what do 4624 and 4625 tell you | Most-repeated SOC question ([epicdetect](https://epicdetect.io/blogs/soc-analyst-interview-questions)) | The table above; name logon types 2, 3, 10 and the authentication package. |
| Detect pass-the-hash | Asked directly in detection interviews ([Glassdoor](https://static.glassdoor.com.ar/Interview/information-engineer-interview-questions-SRCH_KO0,20_IP15.htm)) | 4624 type 3 with NTLM where Kerberos is normal, unusual source, often a privileged account; correlate with 4648 on the source and 4776 on the DC. |
| Kerberoasting | same | 4769 with RC4 (0x17) in an AES domain; one account requesting many service tickets quickly. |
| What artefacts can you get from an NTFS partition | Forensics interviews ([Glassdoor](https://static.glassdoor.ch/Interview/digital-forensics-interview-questions-SRCH_KO0,17_IP2.htm)) | `$MFT`, `$LogFile`, `$UsnJrnl`, alternate data streams, Prefetch, Amcache and Shimcache for execution, LNK and Jump Lists, the registry hives (NTUSER.DAT, SYSTEM, SOFTWARE), event logs. Order of volatility: memory, then disk. |
| Describe what attack is being simulated (given data) | SOC interviews ([Glassdoor](https://static-pc.glassdoor.fr/Interview/soc-analyst-interview-questions-SRCH_KO0,11_IP36.htm)) | Name the ATT&CK technique for each step: T1566 phishing, T1059.001 PowerShell, T1053.005 scheduled task, T1021.002 SMB/admin shares, T1070.001 clear event logs. |

Tools you'd name if asked how you'd do this at scale: a SIEM query (Splunk, KQL, Sigma rules as the portable format), EDR timeline export, Sysmon with a tuned config, PowerShell script-block logging turned on, and `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4688}` when you're on the box itself.

[← Log investigation](README.md) · [Linux log review →](linux-log-review.md)
