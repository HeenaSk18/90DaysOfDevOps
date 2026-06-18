# Linux Troubleshooting Runbook – Day 05
**Author:** [Your Name] | **90DaysOfDevOps** | **Date:** June 2026
**Target Service:** `docker` (Docker Engine daemon)

---

## Environment Basics

### `uname -a`
```bash
$ uname -a
Linux ubuntu-server 5.15.0-107-generic #117-Ubuntu SMP Fri Apr 26 12:26:49 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```
> Kernel `5.15.0` on 64-bit Ubuntu. Noting kernel version matters — some Docker networking features depend on kernel capabilities.

### `lsb_release -a`
```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.4 LTS
Release:        22.04
Codename:       jammy
```
> Ubuntu 22.04 LTS (Jammy) — confirmed long-term support OS. Docker is fully supported here.

---

## Filesystem Sanity Check

### Create throwaway demo folder
```bash
$ mkdir /tmp/runbook-demo
$ cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo
total 4
-rw-r--r-- 1 ubuntu ubuntu 223 Jun 15 10:12 hosts-copy
```
> Write works fine on `/tmp`. File copied with correct permissions. No disk write errors — filesystem is healthy.

---

## Snapshot: CPU & Memory

### `ps -o pid,pcpu,pmem,comm -p $(pgrep dockerd)`
```bash
$ ps -o pid,pcpu,pmem,comm -p $(pgrep dockerd)
    PID %CPU %MEM COMMAND
   1142  0.2  1.4 dockerd
```
> Docker daemon is calm — 0.2% CPU and 1.4% memory at idle. Nothing alarming. Under heavy container load I'd expect CPU to climb; this is baseline.

### `free -h`
```bash
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.9Gi       612Mi       183Mi        12Mi       1.1Gi       1.2Gi
Swap:          1.0Gi          0B       1.0Gi
```
> 1.9 GB total RAM, ~1.2 GB available after buff/cache. Swap is untouched — good sign. If swap starts being used, that's an early warning of memory pressure.

---

## Snapshot: Disk & IO

### `df -h`
```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           197M  1.1M  196M   1% /run
/dev/sda1        20G   11G  7.9G  58% /
tmpfs           985M     0  985M   0% /dev/shm
/dev/sda15      105M  6.1M   99M   6% /boot/efi
```
> Root filesystem at 58% — comfortable. I'd start watching if it crosses 80%. `/dev/shm` (shared memory) is empty, which is expected for current workload.

### `du -sh /var/log`
```bash
$ du -sh /var/log
47M     /var/log
```
> Log directory is 47 MB — reasonable. On busy servers `/var/log` can grow into GBs. If this spikes, I'd run `du -sh /var/log/* | sort -rh | head -10` to find the culprit.

### `vmstat 1 3`
```bash
$ vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 187604  93120 1148908   0    0     1     4   51   88  0  0 99  0  0
 0  0      0 187604  93120 1148908   0    0     0     0   48   82  0  0 100  0  0
 0  0      0 187348  93120 1148908   0    0     0     0   52   89  0  0 100  0  0
```
> CPU idle at 99–100%, zero swap IO (`si`/`so` both 0), no disk wait (`wa=0`). System is healthy at rest. `r=0` means no processes waiting for CPU.

---

## Snapshot: Network

### `ss -tulpn | grep dockerd`
```bash
$ ss -tulpn | grep dockerd
# (no output — dockerd does not listen on a TCP port by default)

$ ss -tulpn | grep -E ':(22|2375|2376)'
tcp   LISTEN 0  128   0.0.0.0:22   0.0.0.0:*   users:(("sshd",pid=978,fd=3))
```
> Docker daemon is running in Unix socket mode (default, more secure). Port `2375` (unencrypted Docker API) is NOT open — good. Port `22` for SSH is open and expected.

### `curl -I --unix-socket /var/run/docker.sock http://localhost/version`
```bash
$ curl -I --unix-socket /var/run/docker.sock http://localhost/version
HTTP/1.1 200 OK
Api-Version: 1.45
Content-Type: application/json
Docker-Experimental: false
Ostype: linux
Server: Docker/26.1.4 (linux)
Date: Mon, 15 Jun 2026 10:18:44 GMT
```
> Docker API responding with `200 OK` on the Unix socket. Engine version `26.1.4`. This is the fastest way to confirm Docker is actually serving requests, not just appearing alive via `systemctl`.

---

## Logs Reviewed

### `journalctl -u docker -n 50`
```bash
$ journalctl -u docker -n 50
Jun 15 09:14:41 ubuntu-server systemd[1]: Starting Docker Application Container Engine...
Jun 15 09:14:42 ubuntu-server dockerd[1142]: time="2026-06-15T09:14:42Z" level=info msg="Starting up"
Jun 15 09:14:42 ubuntu-server dockerd[1142]: time="2026-06-15T09:14:42Z" level=info msg="Loading containers: start."
Jun 15 09:14:43 ubuntu-server dockerd[1142]: time="2026-06-15T09:14:43Z" level=info msg="Loading containers: done."
Jun 15 09:14:43 ubuntu-server dockerd[1142]: time="2026-06-15T09:14:43Z" level=info msg="Docker daemon" commit=f21b197 graphdriver=overlay2 version=26.1.4
Jun 15 09:14:43 ubuntu-server dockerd[1142]: time="2026-06-15T09:14:43Z" level=info msg="Daemon has completed initialization"
Jun 15 09:14:43 ubuntu-server systemd[1]: Started Docker Application Container Engine.
```
> Clean startup. No `level=error` or `level=warning` lines. `graphdriver=overlay2` is the correct modern storage driver. All containers loaded successfully.

### `tail -n 30 /var/log/syslog | grep -i docker`
```bash
$ tail -n 30 /var/log/syslog | grep -i docker
Jun 15 09:14:43 ubuntu-server dockerd[1142]: time="2026-06-15T09:14:43Z" level=info msg="Daemon has completed initialization"
Jun 15 09:14:43 ubuntu-server systemd[1]: Started Docker Application Container Engine.
```
> Syslog confirms same clean start. No OOM killer events, no segfaults near Docker entries.

---

## Quick Findings

| Check | Status | Note |
|-------|--------|------|
| CPU | ✅ Normal | 0.2% at idle, 99% system-wide idle |
| Memory | ✅ Normal | 1.2 GB available, swap untouched |
| Disk | ✅ Normal | 58% used on root, logs at 47 MB |
| IO | ✅ Normal | No disk wait, no swap pressure |
| Network | ✅ Normal | Docker on Unix socket (secure), SSH on port 22 |
| Docker API | ✅ Healthy | HTTP 200 from Unix socket |
| Service Logs | ✅ Clean | No errors or warnings in last 50 lines |

**Verdict:** Docker daemon is fully healthy. This snapshot is now my baseline — any future deviation will be measurable against this.

---

## If This Worsens — Next Steps

**1. If CPU spikes consistently above 70% on `dockerd`:**
```bash
# Find which container is causing it
docker stats --no-stream
# Inspect the noisy container
docker inspect <container_id>
# Check its logs
docker logs --tail 100 <container_id>
# If runaway — stop it gracefully first
docker stop <container_id>
```

**2. If disk crosses 85% (especially `/var/lib/docker`):**
```bash
# Find what Docker is holding
du -sh /var/lib/docker/*
# Remove stopped containers, unused images, dangling volumes
docker system prune -f
# If logs are the culprit, check container log sizes
ls -lh /var/lib/docker/containers/*/*-json.log | sort -k5 -rh | head -10
# Set log rotation in /etc/docker/daemon.json
```

**3. If Docker daemon stops responding entirely:**
```bash
# Check last known error
journalctl -u docker -n 100 --no-pager | grep -E "error|fatal|panic"
# Collect strace on the daemon for 10 seconds
sudo strace -p $(pgrep dockerd) -e trace=network,file -o /tmp/dockerd-strace.txt &
sleep 10 && kill %1
# Attempt graceful restart only after capturing logs
systemctl restart docker
systemctl status docker
```
