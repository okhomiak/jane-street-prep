# 03d · Shell one-liners for the "script up a few things" round

[← Coding overview](README.md) · [Log-analysis tasks](log-analysis-tasks.md) · [Classic tasks](classic-tasks.md)

The 2018 London loop had a phone call where the candidate was asked to "script up a few things" in a language of their choice ([Glassdoor](https://www.glassdoor.com/Interview/Jane-Street-Cyber-Security-Engineer-Interview-Questions-EI_IE255549.0,11_KO12,35.htm)); HRT and Jane Street's DevOps loops ask for common bash commands and FizzBuzz-level Unix ([Glassdoor](https://www.glassdoor.com/Interview/Hudson-River-Trading-Systems-Engineer-Interview-Questions-EI_IE470937.0,20_KO21,37.htm), [Glassdoor](https://www.glassdoor.co.in/Interview/Jane-Street-Interview-E255549-RVW23314043.htm)). Every command below was run against sample `auth.log`, `access.log`, `wiki.csv` and `events.jsonl` files in the formats shown in [block 02](../02-log-investigation/README.md).

Rules for saying them out loud: name the fields before you use them (`$(NF-2)` is "the IP, counting from the end so a variable-width message doesn't break it"); prefer `grep -F` for fixed strings; say when you'd switch to Python (anything with state across lines beyond `uniq -c`).

```bash
# Auth logs (syslog style: "... Failed password for USER from IP port N")
grep 'Failed password' auth.log | awk '{print $(NF-2)}' | sort | uniq -c | sort -rn           # failures by source IP
grep 'Failed password' auth.log | awk '{print $(NF-2), $(NF-4)}' | sort -u | awk '{print $1}' | uniq -c | sort -rn   # distinct users per IP: spray
grep Accepted auth.log | awk '{print $(NF-2)}' | sort -u | while read ip; do n=$(grep -c "Failed password for .* from $ip " auth.log); [ "$n" -gt 0 ] && echo "$ip: $n failures then a success"; done
cut -c1-13 auth.log | sort | uniq -c                                                          # events per hour (timestamp prefix)
awk '$2 >= "02:00:00" && $2 <= "03:00:00"' auth.log | wc -l                                   # lines in a time range
grep -oE 'for [a-z_]+ from [0-9.]+' auth.log | sort | uniq -c | sort -rn | head               # who from where, ranked

# Web access logs (combined format)
awk '$9 ~ /^4/ {print $1}' access.log | sort | uniq -c | sort -rn                              # 4xx by client IP
awk -F'"' '{print $2}' access.log | awk '{print $2}' | sort | uniq -c | sort -rn | head        # top paths
grep -F '../' access.log | cut -d' ' -f1,7                                                     # traversal attempts: who, what
awk '{s+=$10} END {printf "%.1f MB\n", s/1e6}' access.log                                      # bytes served

# CSV and JSON lines
cut -d, -f1 wiki.csv | tail -n +2 | sort | uniq -c | sort -rn                                  # count by first column, skip header
grep -o '"image": "[^"]*"' events.jsonl | grep -i -e AppData -e Temp                           # JSON without jq
jq -r 'select(.ppid != 1000) | [.ts, .host, .image] | @tsv' events.jsonl                      # JSON with jq

# Across files, hashes, recency, decoding
cat auth.log access.log | grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' | sort -u | wc -l           # unique IPs across logs
for h in ws-09 ws-02; do printf '%s: %s\n' "$h" "$(grep -c "\"host\": \"$h\"" events.jsonl)"; done   # a loop
find . -type f -mmin -60                                                                       # files changed in the last hour
sha256sum *.log                                                                                # hashes for the write-up
echo '<b64>' | base64 -d | iconv -f UTF-16LE -t UTF-8                                         # PowerShell -enc
```

Commands they may ask you to explain (reported at HRT and Jane Street's DevOps and Desktop Support loops):

| Question | Answer in one breath |
| --- | --- |
| Pipe versus redirect? | `|` connects one command's stdout to the next's stdin; `>` writes stdout to a file (`2>` for stderr, `&>` both); `<` reads a file as stdin. |
| `kill -15` versus `kill -9`? | SIGTERM asks the process to exit and lets it clean up; SIGKILL can't be caught and the kernel ends it; try 15 first. |
| What does `mv *` do? | The glob expands and the last match becomes the destination, so everything moves into the last directory, or it errors. |
| `du` versus `df` disagree? | A deleted-but-open file (`lsof +L1`), a mount hiding files, sparse files, reserved blocks. |
| Hard link versus soft link? | Same inode versus a file containing a path; hard links can't cross filesystems or point at directories. |
| Alternatives to `ls`? | `find . -maxdepth 1`, `echo *`, `stat`, `printf '%s\n' *`; useful when `ls` is trojaned. |
| Where do you find help? | `man`, `--help`, `type`, `which`, `apropos`; `man 5` for file formats, `man 7` for concepts. |
| Why is `cd` not a binary? | A child process can't change its parent's working directory, so the shell does it itself. |
| How do you push updates to many machines? | Configuration management (Ansible, Salt), a package mirror, and a ring-based rollout; never a loop of `ssh` from your laptop. |
| Python `yield`, generators, decorators? | `yield` makes a lazy iterator so you can stream a file without loading it; a decorator wraps a function to add behaviour (timing, retries, auth checks). |

Jane Street's own line: "know your language" means the one you'll code in. If you say bash is a strength, be able to write a safe loop with quoting and `set -euo pipefail` on the spot.

[← Coding overview](README.md) · [Log-analysis tasks](log-analysis-tasks.md) · [Classic tasks](classic-tasks.md)
