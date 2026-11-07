# 02c · Linux log review

[← Windows walkthrough](windows-walkthrough.md) · [Log investigation](README.md)

"Linux log review" was a named round in the April 2026 Hong Kong loop ([Glassdoor](https://www.glassdoor.co.uk/Interview/Jane-Street-Interview-E255549-RVW103405660.htm)), a Production Engineer was asked to search logs on a VM to find the source of an issue ([Glassdoor](https://www.glassdoor.ie/Interview/Jane-Street-Interview-E255549-RVW82145680.htm)), and Jane Street's servers and tooling are Linux. Expect `auth.log` or `secure`, `syslog` or the journal, a web server log, and the cron and sudo trails. This page is defensive: how to read those logs and spot the signs of an intrusion. Every command was run against sample files first.

---

## Where to look, and what the sign looks like

| Log | Path | The sign you're reading for |
| --- | --- | --- |
| SSH and PAM | `/var/log/auth.log`, `/var/log/secure`, `journalctl -u ssh` | Bursts of failures from one source; a password accept where the fleet uses keys; a login at an odd hour or from a new IP |
| sudo | same auth log | A rights escalation to root that doesn't match the person's normal work |
| User and group changes | auth log, `/var/log/secure` | A new account, or an account added to a privileged group, around the time of other odd activity |
| Scheduled jobs | `/var/log/syslog` CRON lines, `systemctl list-timers`, `/etc/cron.*` | A job running something from a world-writable or hidden directory |
| Services | journal, `systemctl list-units` | A service installed close to the incident window with an unfamiliar name |
| Web server | `/var/log/nginx/access.log`, Apache equivalent | Automated user agents, requests to login or upload paths, requests that try to read files outside the web root |
| Shell history and keys | `~/.bash_history`, `~/.ssh/authorized_keys` | An added key you can't account for; history that was truncated |

The method is the same six steps as the [main page](README.md): orient, baseline, find the rare, pivot on one indicator, build a timeline, write up. Below are the read-only commands for each log.

---

## Auth and sudo

```bash
grep 'Failed password' auth.log | awk '{print $(NF-2)}' | sort | uniq -c | sort -rn      # failures by source IP
grep 'Accepted' auth.log | awk '{print $8, $6}' | sort | uniq -c                          # sessions per user, key vs password
grep 'Accepted password' auth.log                                                         # odd where keys are policy
grep ' sudo: ' syslog | sed -E 's/.*sudo: +([^ ]+) :.*COMMAND=(.*)/\1 \2/'                # who ran what as root
grep -E 'new user|useradd|usermod|group' auth.log syslog                                  # account changes
last; lastb                                                                               # login and bad-login history
```

What you say: "First, what does normal look like? Almost every accept is `publickey` for a named user from the office range. The one `password` accept, for a service account from an external IP, is the thread I pull. Then I check what that session did with sudo."

## Persistence spots

```bash
grep -E 'CRON|systemd' syslog | grep -E '/tmp|/dev/shm|/var/tmp|Started' | cut -c1-100     # jobs and services from odd paths
systemctl list-timers; ls -la /etc/cron.* /etc/systemd/system 2>/dev/null                  # scheduled work
cat ~/.ssh/authorized_keys 2>/dev/null; ls -la /etc/ld.so.preload 2>/dev/null              # added keys, preload hijack
```

A cron job or service pointing at a hidden directory (a name starting with a dot) or a temp path, installed inside the incident window, is persistence. Say you'd capture it before touching it.

## Web server

```bash
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -rn                               # user agents: scripts stand out
grep -E -i 'wp-login|/admin|/uploads/|cmd=|\.\./|/etc/passwd' access.log | cut -d' ' -f1,4,7  # login, upload and traversal attempts
awk '$6=="\"POST" && $9==302 {print $1}' access.log | sort | uniq -c                         # POSTs that redirected: a login that took
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head                              # noisiest clients
```

What you say: "A `python-requests` or `curl` user agent hitting a login path, then a request to a file in an uploads directory with a command in the query string, is the classic get-in-then-run-commands pattern against a web app. I'd pull every request from that source IP in order and see what it reached."

## Running system, read-only

The [Linux block](../06-linux/README.md) has the full process, socket, and integrity kit. The ones you name in a live review:

```bash
ps auxf                                   # process tree with parents
ls -l /proc/<pid>/exe                      # the real binary; a "(deleted)" tag is a strong tell
ss -tanp state established                 # live connections and the owning process
find / -newer /etc/hostname -type f 2>/dev/null | grep -vE '/proc|/sys' | head   # recently changed files
```

Cross-check `/proc` against `ps`: if a connection has no matching visible process, suspect a user-space rootkit and stop trusting the box's own tools.

---

## The two-letter binaries question, here because it's a Linux round

Jane Street asked a London candidate to name five two-letter Linux binaries ([Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm)). Answer: `ls cp mv rm ps df du id wc ln dd tr ip ss nl od vi sh nc`. The trap is `cd`: it's a shell builtin, not a binary, because a child process can't change its parent's working directory. Show it with `type cd` versus `type ls`. Full Linux depth is in [block 06](../06-linux/README.md).

[← Windows walkthrough](windows-walkthrough.md) · [Log investigation](README.md)
