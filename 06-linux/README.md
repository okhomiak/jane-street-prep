# 06 · Linux and systems

[← Networking](../05-networking/README.md) · [Index](../README.md) · [Next: Windows and endpoints →](../07-windows-and-endpoints/README.md)

Jane Street runs on Linux and asks Linux questions across every technical loop: "name five two-letter binaries" to a cyber security engineer, "list some two-letter commands" to a developer, FizzBuzz-level Unix questions (basic commands, reading files, using man pages) to a DevOps candidate, and "tell me about your Linux side projects" to a Linux engineer ([Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm), [Glassdoor](https://www.glassdoor.co.nz/Interview/Jane-Street-Interview-E255549-RVW4711912.htm), [Glassdoor](https://www.glassdoor.co.in/Interview/Jane-Street-Interview-E255549-RVW23314043.htm)). Hudson River Trading's systems loop is the same flavour: inodes, hard versus soft links, `du` versus `df`, RAID 5 versus 6 ([Glassdoor](https://www.glassdoor.com/Interview/Hudson-River-Trading-Systems-Engineer-Interview-Questions-EI_IE470937.0,20_KO21,37.htm)). Tags: [JS] Jane Street, [HRT] Hudson River Trading, [DM] Miessler, [TW] tadwhitaker, [GN] gracenolan, [DF] Dataford's Jane Street guide, [GD] Glassdoor.

---

## The Jane Street and HRT questions

<details>
<summary><b>Name five Linux binaries that are two letters long.</b> · [JS Cyber Security Engineer, London]</summary>

`ls cp mv rm ps df du id wc ln dd tr su ip ss nl od vi sh nc bc ar as cc ld nm xz`. The trap: `cd` is a shell builtin, not a binary; show it with `type cd` versus `type ls`. If they keep going, explain why `cd` has to be a builtin: a child process can't change its parent's working directory.
</details>

<details>
<summary><b>What is an inode? Hard versus soft link?</b> · [HRT]</summary>

An inode is the filesystem record for a file: owner, mode, timestamps, size, block pointers, link count. The name lives in the directory entry, not the inode. A hard link is a second directory entry pointing at the same inode: same file, can't cross filesystems, can't point at directories, and the data survives until the last link is removed. A soft link is a file whose content is a path: can cross filesystems, dangles if the target moves. Security angle: symlink races (TOCTOU) in `/tmp`, which is why `fs.protected_symlinks` exists; and a deleted-but-open file (`(deleted)` in `/proc/<pid>/exe`) is a classic malware tell.
</details>

<details>
<summary><b>Why might du and df disagree?</b> · [HRT]</summary>

`df` asks the filesystem how many blocks are used; `du` walks the tree and sums file sizes. They differ when a process holds a deleted file open (space used, no path to count; find it with `lsof +L1`), when a mount hides files underneath a mount point, with sparse files, with reserved blocks for root, and when `du` can't read some directories.
</details>

<details>
<summary><b>What does `mv *` do? Alternatives to `ls`? Install Linux on 100 nodes?</b> · [HRT]</summary>

`mv *` expands the glob, so the last match becomes the destination: everything is moved into the last directory in the list, or it errors if the last item is a file. Alternatives to `ls`: `find . -maxdepth 1`, `echo *`, `stat`, `tree`, `printf '%s\n' *`, or `python -c 'import os;print(os.listdir())'` when `ls` is trojaned, which is why you'd want one. 100 nodes: PXE boot with a kickstart or preseed answer file from a provisioning server, or an image pushed by a tool like Foreman, Cobbler, MAAS, then configuration management (Ansible, Salt, Puppet) to converge them. Say the security part: signed images, a known-good baseline, no hand-built snowflakes.
</details>

<details>
<summary><b>RAID 5 versus RAID 6?</b> · [HRT]</summary>

RAID 5 stripes with one parity block and survives one disk failure; RAID 6 keeps two parity blocks and survives two. With large disks, a RAID 5 rebuild takes long enough that a second failure or an unrecoverable read error during the rebuild is likely, so RAID 6 or mirroring is the safer choice. Neither is a backup.
</details>

<details>
<summary><b>Given suspicious logs, how do you determine whether an unauthorised process is running?</b> · [DF, Jane Street guide] [TW: "discovered an infected host"]</summary>

Cross-check views that an attacker would have to fake all at once. Process tree with parents (`ps auxf`, `pstree -p`); the real binary behind each process (`ls -l /proc/<pid>/exe`, a `(deleted)` tag or a path in `/tmp` or `/dev/shm` is bad); the command line (`tr '\0' ' ' < /proc/<pid>/cmdline`); listening and established sockets per process (`ss -tulpn`, `ss -tanp`); open files (`lsof -p`); persistence (`crontab -l`, `/etc/cron.*`, `systemctl list-timers`, `systemctl list-units --type=service`, `~/.bashrc`, `~/.ssh/authorized_keys`, `/etc/ld.so.preload`); package integrity (`rpm -Va`, `debsums`). A user-space rootkit can lie to `ps`, so compare `/proc` entries with `ps` output and look at network logs from outside the box. If it's a kernel rootkit, trust nothing on the host and image it.
</details>

<details>
<summary><b>Explain the security trade-offs of different IPC mechanisms on Linux.</b> · [DF, Jane Street guide] [Citadel: thread versus process isolation]</summary>

Unix domain sockets: filesystem permissions and `SO_PEERCRED` tell you who connected; the default choice for local services. Localhost TCP: any local user can connect, so you need application-level auth. Pipes: inherited from the parent, simple and scoped. Shared memory: fastest, but no per-message access control and a corrupt writer corrupts the reader. D-Bus: policy-driven but a large attack surface. Signals: tiny channel, easy to abuse for denial of service. Threads share an address space so one bug compromises all; processes give the kernel a boundary (separate memory, separate credentials, seccomp, namespaces) at the cost of copying data. Say what you'd pick for a trading component: processes with a Unix socket and a strict seccomp profile.
</details>

## Hardening and privilege

<details>
<summary><b>What are your first three steps when securing a Linux server?</b> · [DM] [TW: "how do you harden a system"]</summary>

Miessler says the list matters less than staying calm. A good list: patch and enable automatic security updates; reduce exposure (firewall default deny, remove services you don't need, SSH keys only with no root login); and get visibility (central logging, auditd or an EDR agent, file integrity monitoring). Then least privilege for accounts and services, a MAC system (SELinux or AppArmor) in enforcing mode, and a baseline you can compare against.
</details>

<details>
<summary><b>How do you elevate privileges on Linux? How do you find the paths?</b> · [TW] [GN]</summary>

Attacker view: `sudo -l` (sudo on `vim`, `less`, `find` or `python` is a root shell), SUID binaries (`find / -perm -4000`), file capabilities (`getcap -r /`), writable scripts or binaries that root runs from cron or a service, writable `PATH` directories, kernel exploits on unpatched boxes, credentials in config files and shell history, docker group membership (equals root), and misconfigured NFS or `ld.so.preload`. Defender view is the same list run as an audit, plus not giving service accounts shells.
</details>

<details>
<summary><b>SELinux, MAC versus DAC, and what /proc, /tmp and /etc/shadow expose.</b> · [GN]</summary>

DAC is the owner deciding (mode bits, ACLs); MAC is policy the owner can't override (SELinux labels and type enforcement, AppArmor profiles), so a compromised web server still can't read `/etc/shadow` even as that user. `/proc` exposes every process's command line, environment, maps and open files to users who can read them, which is why `hidepid=2` exists. `/tmp` is world-writable and shared, so it's where symlink races and dropped payloads live; mount it `noexec,nosuid`. `/etc/shadow` holds the password hashes and is root-readable only; a world-readable copy is an instant audit finding.
</details>

<details>
<summary><b>How does the kernel know which function to call for a user request?</b> · [TW]</summary>

A system call: user code puts a syscall number and arguments in registers and executes `syscall`; the CPU switches to ring 0 and jumps to the kernel entry point, which indexes the syscall table and runs the handler. Security relevance: seccomp filters which syscalls a process may make, which is how containers and sandboxes shrink the kernel's attack surface; `strace` shows you the calls, and a process doing `ptrace` on others is a hunt lead.
</details>

<details>
<summary><b>How does a buffer overflow work and how do you defend against it?</b> · [DM] [GD Google] [GN]</summary>

Writing past the end of a stack buffer overwrites the saved return address, so the function returns into attacker-chosen code or, with non-executable stacks, into a chain of existing code fragments (ROP). Defences, layered: memory-safe languages where you can, compiler canaries and `_FORTIFY_SOURCE`, NX/DEP so data pages can't execute, ASLR so addresses aren't predictable, PIE and RELRO, control-flow integrity, and fuzzing to find the bugs before shipping. Jane Street's OCaml is memory-safe by default; say that it removes the class rather than the instance.
</details>

<details>
<summary><b>How would you harden SSH?</b> · [GN] [TW]</summary>

Keys only, no root login, no agent forwarding by default (`ProxyJump` instead), a bastion with session recording, short-lived certificates from an SSH CA instead of static keys (so offboarding is automatic), MFA on the bastion, `AllowGroups`, modern ciphers only, and alerting on `Accepted password` where keys are policy. fail2ban cuts noise; it isn't the control.
</details>

<details>
<summary><b>Linux versus Windows: strengths and weaknesses?</b> · [DM]</summary>

Give both sides and no tribalism; Miessler flags bias as a red flag. Linux: transparent, scriptable, minimal installs, strong isolation primitives (namespaces, seccomp, MAC), but configuration sprawl and no single directory service built in. Windows: Active Directory and Group Policy give fleet control that Linux shops have to build, mature EDR and attestation (TPM, Credential Guard), but a huge legacy attack surface (NTLM, SMB, macros) and an ecosystem attackers study hardest. Most firms run both, so the answer is knowing the failure modes of each.
</details>

## Operations questions that leak into security loops

<details>
<summary><b>Common causes of high CPU on a Linux server, and how to find a memory leak in a Python script.</b> · [GD Meta]</summary>

CPU: a runaway process or loop, a crypto-miner (check `top`, unknown binaries in `/tmp`), I/O wait masquerading as CPU, too many threads, GC storms, a kernel soft lockup, `perf top` to see where cycles go. Python memory: `tracemalloc` snapshots over time, `objgraph` for what's growing, look for caches without bounds, global lists, closures holding references, C extensions. Security point: a miner is the most common "mystery CPU" on an exposed server.
</details>

<details>
<summary><b>How would you harden your work laptop for Defcon? Your home network?</b> · [TW]</summary>

Defcon: a clean loaner with nothing on it, full-disk encryption, Wi-Fi and Bluetooth off, mobile hotspot or a known VPN only, no charging from unknown USB, updates applied before travel, passkeys so a captured password is useless, and wipe it after. Home: router firmware current, WPA3, no WPS, separate guest and IoT networks, DNS with filtering, no port forwards, UPnP off, unique admin password, and the honest trade-off that you don't run enterprise controls at home because you'd stop using them.
</details>

## Linux recall list

Say these from memory.

```bash
ps auxf; pstree -p                            # process tree
ls -l /proc/<pid>/exe; tr '\0' ' ' < /proc/<pid>/cmdline; cat /proc/<pid>/environ | tr '\0' '\n'
ss -tulpn; ss -tanp state established         # sockets with owning process
lsof -p <pid>; lsof -i :443; lsof +L1          # open files, by port, deleted-but-open
crontab -l; ls /etc/cron.*; systemctl list-timers; systemctl list-units --type=service --state=running
cat ~/.ssh/authorized_keys /etc/ld.so.preload; ls -la /etc/systemd/system
sudo -l; find / -perm -4000 -type f 2>/dev/null; getcap -r / 2>/dev/null
last; lastb; journalctl -u ssh --since today   # unit is sshd on RHEL-family
find / -newer /etc/passwd -type f 2>/dev/null | head   # what changed recently
stat file; md5sum file; rpm -Va | head; debsums -c      # integrity
auditctl -l; ausearch -k <key>                 # audit rules and events
ip a; ip r; ip neigh                           # addresses, routes, ARP table
```

[← Networking](../05-networking/README.md) · [Index](../README.md) · [Next: Windows and endpoints →](../07-windows-and-endpoints/README.md)
