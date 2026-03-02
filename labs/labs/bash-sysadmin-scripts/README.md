# Linux Bash Sysadmin Scripts

## Objective
To develop practical Bash automation scripts for common system 
administration tasks including disk monitoring, user account auditing, 
and system health reporting. All scripts were written and executed on 
Kali Linux.

## Tools Used
- Bash
- Kali Linux
- Built-in Linux utilities: df, free, ps, who, last, ip, uptime, awk, grep

---

## Scripts

### 1. Disk Usage Monitor (`disk_monitor.sh`)
Checks all mounted partitions and reports usage percentage against a 
configurable threshold (default 80%). Outputs a warning if any 
partition exceeds the threshold.

**Output on test run:**
- udev: 0% capacity — OK
- /dev/sda1: 57% capacity — OK

<img width="544" height="194" alt="image" src="https://github.com/user-attachments/assets/ece99fa4-79b5-4730-8576-a265c3497dc5" />

```bash
#!/bin/bash
THRESHOLD=80
echo "============================================"
echo "         DISK USAGE MONITOR"
echo "         $(date)"
echo "============================================"
df -H | grep -vE '^Filesystem|tmpfs|cdrom' | while read line;
do
  partition=$(echo $line | awk '{print $1}')
  usage=$(echo $line | awk '{print $5}' | cut -d'%' -f1)
  if [ "$usage" -ge "$THRESHOLD" ] 2>/dev/null; then
    echo "WARNING: $partition is at ${usage}% capacity"
  elif [ -n "$usage" ]; then
    echo "OK: $partition is at ${usage}% capacity"
  fi
done
echo "============================================"
echo "Scan complete."
echo "============================================"
```

---

### 2. User Account Info (`user_info.sh`)
Audits system user accounts by pulling current user details, all users 
with active login shells, currently logged in sessions, and the last 5 
login events from the auth log.

**Key findings on test run:**
- Current user: kali (zsh shell)
- 4 accounts with login shells: root, sync, postgres, kali
- Notable: postgres installed on system — database service present
- Login history dating back to Feb 28 2026

<img width="891" height="623" alt="image" src="https://github.com/user-attachments/assets/7e41f6c3-b43b-4125-aa25-557c221d2f86" />

```bash
#!/bin/bash
echo "============================================"
echo "         USER ACCOUNT INFORMATION"
echo "         $(date)"
echo "============================================"
echo "Current User: $(whoami)"
echo "Hostname: $(hostname)"
echo "Home Directory: $HOME"
echo "Shell: $SHELL"
echo ""
echo "All system users with login shells:"
grep -v nologin /etc/passwd | grep -v false | cut -d: -f1,3,6
echo ""
echo "Currently logged in users:"
who
echo ""
echo "Last 5 logins:"
last -n 5
echo "============================================"
echo "Scan complete."
echo "============================================"
```

---

### 3. System Health Check (`system_health.sh`)
Generates a full system health report covering uptime, top CPU 
consuming processes, memory usage, disk usage summary, and active 
network interfaces.

**Key findings on test run:**
- Uptime: 1 hour 21 minutes, load average 0.20 (healthy)
- RAM: 1.9GB total, 875MB used, 1.1GB available
- Swap: 953MB total, only 12KB used — memory pressure minimal
- Disk: 79GB total, 43GB used (57%), 33GB free
- Top process: Xorg at 1.0% CPU (display server)
- Network: eth0 UP on 192.168.11.0/24 subnet

<img width="1568" height="659" alt="image" src="https://github.com/user-attachments/assets/435d00f1-8ee7-414b-b548-172ce110abea" />

```bash
#!/bin/bash
echo "============================================"
echo "         SYSTEM HEALTH CHECK"
echo "         $(date)"
echo "============================================"
echo "System Uptime:"
uptime
echo ""
echo "CPU Usage (top 5 processes):"
ps aux --sort=-%cpu | head -6
echo ""
echo "Memory Usage:"
free -h
echo ""
echo "Disk Usage Summary:"
df -h | grep -vE '^tmpfs|udev'
echo ""
echo "Network Interfaces:"
ip -brief addr
echo "============================================"
echo "Health check complete."
echo "============================================"
```

---

## Key Takeaways
- Bash scripting allows repetitive sysadmin tasks to be automated and 
  run on demand, reducing manual effort and human error
- Disk monitoring scripts are essential in production environments where 
  full partitions can cause service outages
- User auditing reveals all accounts with login access — a critical 
  step in access control reviews and security audits
- System health checks provide a fast snapshot of machine state, useful 
  for incident response and routine maintenance
- Combining tools like `ps`, `free`, `df`, and `ip` into a single 
  script mirrors real-world sysadmin workflows

## Skills Demonstrated
- Bash scripting and shell logic (if/elif, while loops, variables)
- File permissions and executable management (chmod)
- Linux system utilities (df, free, ps, who, last, ip, uptime)
- Text processing with awk and grep
- Script output formatting and reporting
- File creation using heredoc syntax (cat << EOF)
