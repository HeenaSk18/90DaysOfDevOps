# Linux Practice Log – Day 04
**Service Inspected:** `ssh` (sshd)

---

## 1. Process Checks

### `ps aux | grep sshd`
```bash
$ ps aux | grep sshd
root         978  0.0  0.1  15432  7216 ?  Ss   09:14   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
ubuntu      3241  0.0  0.0   6432   736 pts/0  S+   10:02   0:00 grep --color=auto sshd
```
**What I learned:** The first line is the actual sshd process owned by `root`, running as a daemon (`Ss`). The second line is just my grep command itself showing up — that's normal.

---

### `pgrep -a sshd`
```bash
$ pgrep -a sshd
978 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
```
**What I learned:** `pgrep -a` is cleaner than `ps aux | grep` — shows PID + full command, no noise. PID `978` is sshd. I can use this PID to inspect or signal the process directly.

---

## 2. Service Checks

### `systemctl status ssh`
```bash
$ systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2026-06-15 09:14:23 UTC; 48min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 947 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 978 (sshd)
      Tasks: 1 (limit: 1141)
     Memory: 5.8M
        CPU: 67ms
     CGroup: /system.slice/ssh.service
             └─978 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Jun 15 09:14:23 ubuntu-server systemd[1]: Starting OpenBSD Secure Shell server...
Jun 15 09:14:23 ubuntu-server sshd[978]: Server listening on 0.0.0.0 port 22.
Jun 15 09:14:23 ubuntu-server systemd[1]: Started OpenBSD Secure Shell server.
```
**What I learned:**
- Status is `active (running)` — healthy
- `enabled` means it will auto-start on reboot
- Main PID matches what `pgrep` showed — consistent
- The service started cleanly with `ExecStartPre` passing a config check first

---

### `systemctl list-units --type=service --state=running`
```bash
$ systemctl list-units --type=service --state=running
  UNIT                        LOAD   ACTIVE SUB     DESCRIPTION
  cron.service                loaded active running Regular background program processing daemon
  dbus.service                loaded active running D-Bus System Message Bus
  docker.service              loaded active running Docker Application Container Engine
  networkd-dispatcher.service loaded active running Networkd-dispatcher for network
  ssh.service                 loaded active running OpenBSD Secure Shell server
  systemd-journald.service    loaded active running Journal Service
  systemd-logind.service      loaded active running User Login Management
  systemd-udevd.service       loaded active running Rule-based Manager for Device Events

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state.
SUB    = The low-level unit activation state.

8 loaded units listed.
```
**What I learned:** I can see everything that's actively running at once. `docker.service` and `cron.service` are up — useful to know for later days. This is my go-to command when I SSH into a new server and want to understand what's running.

---

## 3. Log Checks

### `journalctl -u ssh --since "1 hour ago"`
```bash
$ journalctl -u ssh --since "1 hour ago"
Jun 15 09:14:23 ubuntu-server systemd[1]: Starting OpenBSD Secure Shell server...
Jun 15 09:14:23 ubuntu-server sshd[978]: Server listening on 0.0.0.0 port 22.
Jun 15 09:14:23 ubuntu-server systemd[1]: Started OpenBSD Secure Shell server.
Jun 15 09:47:11 ubuntu-server sshd[978]: Accepted publickey for ubuntu from 192.168.1.5 port 54321 ssh2
Jun 15 09:47:11 ubuntu-server sshd[978]: pam_unix(sshd:session): session opened for user ubuntu
```
**What I learned:** I can see exactly when someone (me) connected via SSH — IP, port, auth method. In a real incident, this log would tell me if there were unauthorized login attempts.

---

### `tail -n 20 /var/log/auth.log`
```bash
$ tail -n 20 /var/log/auth.log
Jun 15 09:14:23 ubuntu-server sshd[978]: Server listening on 0.0.0.0 port 22.
Jun 15 09:47:11 ubuntu-server sshd[978]: Accepted publickey for ubuntu from 192.168.1.5 port 54321 ssh2
Jun 15 09:47:11 ubuntu-server sshd[978]: pam_unix(sshd:session): session opened for user ubuntu(uid=1000)
Jun 15 09:50:33 ubuntu-server sudo: ubuntu : TTY=pts/0 ; PWD=/home/ubuntu ; USER=root ; COMMAND=/usr/bin/systemctl status ssh
Jun 15 09:51:10 ubuntu-server sudo: ubuntu : TTY=pts/0 ; PWD=/home/ubuntu ; USER=root ; COMMAND=/usr/bin/journalctl -u ssh
```
**What I learned:** `/var/log/auth.log` records every authentication event — SSH logins, sudo usage. I can see my own `sudo` commands logged here. This is critical for auditing who did what on a server.

---

## 4. Mini Troubleshooting Flow

**Scenario:** *"SSH is not responding — how do I investigate?"*

```bash
# Step 1 — Is the service running at all?
systemctl status ssh
# → If "inactive (dead)" → service crashed or was stopped

# Step 2 — Check recent logs for errors
journalctl -u ssh --since "10 minutes ago"
# → Look for: "Failed to start", "Address already in use", permission errors

# Step 3 — Is the process actually alive?
pgrep -a sshd
# → No output = process is dead → go to step 4

# Step 4 — Restart and watch
systemctl restart ssh
systemctl status ssh
# → Did it come back? Did it fail again immediately?

# Step 5 — Is port 22 actually open?
ss -tulnp | grep :22
# → Should show sshd listening on 0.0.0.0:22

# Step 6 — Firewall blocking?
ufw status | grep 22
# → Should show "22/tcp ALLOW"
```

**Resolution approach:**
- If logs show `Address already in use` → another process grabbed port 22, find it with `ss -tulnp | grep :22`
- If logs show config errors → run `sshd -t` to test config syntax
- If service keeps dying → check disk space with `df -h` (full disk kills processes)

---

## Key Takeaways from Today

| Command | When I'll Use It |
|---------|-----------------|
| `pgrep -a <name>` | Quick sanity check — is this process alive? |
| `systemctl status <service>` | First thing to run on any service issue |
| `journalctl -u <service> --since "Xh ago"` | Scoped log view without drowning in noise |
| `tail -n 20 /var/log/auth.log` | Security checks — who logged in, who ran sudo |
| `ss -tulnp \| grep :PORT` | Is my service actually listening on the right port? |
