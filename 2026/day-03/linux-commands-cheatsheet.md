# Linux Commands Cheat Sheet
> Focus: Process Management · File System · Networking Troubleshooting

---

## 🔧 Process Management

| Command | Usage | What It Does |
|---------|-------|--------------|
| `ps aux` | `ps aux \| grep nginx` | List all running processes; filter by name |
| `top` | `top` | Live view of CPU/memory usage per process |
| `htop` | `htop` | Better interactive version of `top` (install if missing) |
| `kill` | `kill 1234` | Send SIGTERM to process with PID 1234 (graceful stop) |
| `kill -9` | `kill -9 1234` | Force-kill a process — use when `kill` doesn't work |
| `pkill` | `pkill nginx` | Kill process by name instead of PID |
| `jobs` | `jobs` | List background jobs in current shell session |
| `nohup` | `nohup ./script.sh &` | Run process that survives terminal logout |
| `systemctl status` | `systemctl status nginx` | Check if a service is running, failed, or active |
| `systemctl restart` | `systemctl restart nginx` | Restart a service (use after config changes) |
| `journalctl -u` | `journalctl -u nginx --since "1h ago"` | View logs for a specific service |

---

## 📁 File System

| Command | Usage | What It Does |
|---------|-------|--------------|
| `ls -lh` | `ls -lh /var/log` | List files with human-readable sizes and permissions |
| `du -sh` | `du -sh /var/log/*` | Show disk usage per file/folder — find what's eating space |
| `df -h` | `df -h` | Show disk space on all mounted filesystems |
| `find` | `find /etc -name "*.conf" -type f` | Search for files by name or type recursively |
| `tail -f` | `tail -f /var/log/syslog` | Live-follow a log file as it writes — essential in prod |
| `grep -r` | `grep -r "ERROR" /var/log/` | Search for a string recursively across all files |
| `chmod` | `chmod 755 script.sh` | Set file permissions (owner=rwx, group=rx, other=rx) |
| `chown` | `chown ubuntu:ubuntu app/` | Change file/folder owner and group |
| `ln -s` | `ln -s /opt/app /usr/local/bin/app` | Create a symbolic link (shortcut) |
| `tar -xzvf` | `tar -xzvf archive.tar.gz` | Extract a gzipped tar archive |

---

## 🌐 Networking Troubleshooting

| Command | Usage | What It Does |
|---------|-------|--------------|
| `ping` | `ping -c 4 google.com` | Test basic connectivity to a host (4 packets) |
| `ip addr` | `ip addr show` | Show all network interfaces and their IP addresses |
| `curl` | `curl -I https://example.com` | Check HTTP response headers; test if an endpoint is alive |
| `dig` | `dig google.com A` | DNS lookup — see what IP a domain resolves to |
| `netstat -tulnp` | `netstat -tulnp` | List all open ports and which process is listening |
| `ss -tulnp` | `ss -tulnp` | Faster modern replacement for `netstat` |
| `traceroute` | `traceroute google.com` | Show network path (hops) to a destination |
| `wget` | `wget https://example.com/file.zip` | Download a file from a URL |
| `nslookup` | `nslookup example.com` | Quick DNS check — alternative to `dig` |
| `ufw status` | `ufw status verbose` | Check firewall rules (Ubuntu UFW) |

---

## 📌 Quick Reference — Most Reached-For Commands

```bash
# Who is eating my disk?
du -sh /var/log/* | sort -rh | head -10

# What process is using port 8080?
ss -tulnp | grep 8080

# Is my app service alive?
systemctl status myapp

# Follow logs live
tail -f /var/log/myapp/app.log

# Did my DNS change?
dig myapp.example.com A

# Test if API endpoint responds
curl -I https://api.myapp.com/health
```

---

## 💡 SDET → DevOps Notes

- `tail -f` + `grep` is your best friend when debugging failing CI/CD pipelines
- Always try `systemctl status` before restarting a service — logs tell you *why* it failed
- `curl -I` is faster than opening a browser to check if a server is up
- `kill -9` is a last resort — always try `systemctl restart` first

---

> *"The terminal is your production cockpit. Know it well."*
