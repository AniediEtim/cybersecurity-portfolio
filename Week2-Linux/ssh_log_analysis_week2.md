
# SSH Log Analysis — Hands-on Investigation

> **Focus:** SSH authentication log analysis, timeline reconstruction, grep & pipe fundamentals  
> **Tools:** Kali Linux, journalctl, grep, cat, last, who  
> **Environment:** Local Kali Linux machine

---

## What I Did

Investigated SSH authentication logs on a live Linux machine to practice real SOC analyst skills — parsing logs, reconstructing a timeline, identifying suspicious events, and pivoting between entities.

---

## Commands & What Each One Does

### Reading the logs
```bash
sudo cat /var/log/auth.log 2>/dev/null || sudo journalctl -u ssh --no-pager | head -30
```
| Part | What it does |
|------|-------------|
| `sudo` | Run as superuser — log files need root access |
| `cat /var/log/auth.log` | Print the full auth log to screen |
| `2>/dev/null` | Suppress error messages silently |
| `\|\|` | OR — if first command fails, run the second |
| `journalctl -u ssh` | Read SSH service logs from systemd journal |
| `--no-pager` | Show all output at once, no scrolling |
| `head -30` | Show first 30 lines only |

---

### grep — Filter logs by pattern
```bash
# Find all successful logins
sudo journalctl -u ssh --no-pager | grep "Accepted"

# Find failed, timed out, or invalid attempts (OR logic)
sudo journalctl -u ssh --no-pager | grep -i "failed\|timeout\|invalid"
```

**grep flags:**

| Flag | What it does |
|------|-------------|
| `-i` | Case-insensitive — matches Failed, FAILED, failed |
| `-v` | Invert — show lines that do NOT match |
| `-c` | Count — how many lines match |
| `-n` | Show line numbers next to matches |

---

### Pipe ( | ) — Chain commands together
```bash
# Output of command 1 becomes input of command 2
journalctl -u ssh | grep "Failed" | wc -l
```

Think of it as an assembly line:
```
journalctl → produces logs
grep       → filters to what matters
wc -l      → counts the results
```

**Full brute-force detector in one line:**
```bash
sudo journalctl -u ssh | grep "Failed" | awk '{print $11}' | sort | uniq -c | sort -rn
```
Shows which IPs have the most failed login attempts, sorted by frequency.

---

### Follow-up investigation commands
```bash
# Confirm if user exists
cat /etc/passwd | grep -i "anny"

# Full login history
last | head -20

# What did the user do after login?
sudo cat /home/Anny/.bash_history

# Any privilege escalation attempts?
sudo grep -i "sudo\|su\|root" /var/log/auth.log

# Who is currently logged in?
who -a && w
```

---

## Findings — Timeline Reconstructed

| Timestamp | Event | Verdict |
|-----------|-------|---------|
| Apr 04 06:06 | SSH service started | Normal boot |
| Apr 04 15:28 | SSH restarted | Second reboot same day |
| Apr 05 00:33 | SSH restarted again | Third boot |
| **Apr 05 02:23:08** | **Auth timeout from 127.0.0.1** | Suspicious |
| **Apr 05 02:29:49** | **Login accepted — user Anny** | Notable — 6 mins after timeout |
| Apr 09 10:15 | SSH restarted | 4-day gap |

---

## Key Findings

1. **Auth timeout at 02:23** — connection dropped before any credentials submitted. Source: localhost
2. **Successful login at 02:29** — user `Anny` via password auth, same source, 6 minutes later
3. **Password auth used** — not SSH keys. Weaker and more brute-forceable
4. **SSHing into localhost** — unusual. Could indicate tunneling or scripted automation
5. **3 reboots in 2 days** — instability worth investigating

---

## Investigative Questions Raised

- Who created user `Anny` and when?
- What commands did Anny run after login?
- Is password authentication intentionally enabled in `/etc/ssh/sshd_config`?
- What caused three reboots in 48 hours?

---

## Real World Connection

The **2020 Twitter breach** started with attackers SSHing into internal admin systems via stolen credentials. Authentication logs showed clear anomalies — unusual time, password auth, unexpected source. Nobody caught it. Result: 130 accounts compromised including world leaders and major corporations.

The same `grep` and `pipe` commands in this lab are what a SOC analyst would have used to catch it early.



## SOC Skills Demonstrated

| Skill | Applied |
|-------|---------|
| Log analysis | Parsed raw SSH journal logs |
| Timeline reconstruction | Ordered events chronologically |
| Entity pivoting | Event → user → source IP |
| True/false positive analysis | Flagged suspicious timeout + login sequence |
| grep & pipe | Used to filter and chain investigation commands |
| Documentation | This writeup |


## Key Takeaways

- `grep` is your scalpel — cuts noise, finds signal fast
- Pipes turn single commands into full investigation workflows  
- Always ask: **who, what, when, where, why** for every alert
- Localhost SSH connections deserve the same scrutiny as external ones
- Post-login activity is where the real damage usually happens

---

*Hands-on SOC training — SSH log analysis & Linux command fundamentals*  
*Stack: Kali Linux · journalctl · grep · pipe · SSH authentication logs*
