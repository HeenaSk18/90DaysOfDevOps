# Linux Architecture, Processes, and systemd

## Linux Architecture

Linux can be understood in 3 main layers:

### 1. Kernel

* Core part of the operating system.
* Communicates directly with hardware.
* Manages CPU, memory, disk, networking, and processes.
* Acts as a bridge between applications and hardware.

### 2. User Space

* Area where users and applications run.
* Examples: bash, nginx, docker, python applications.
* Applications cannot access hardware directly; they use kernel services.

### 3. Init / systemd

* First process started when Linux boots.
* Process ID (PID) = 1.
* Responsible for starting and managing system services.
* Most modern Linux distributions use systemd.

---

## How Processes Work

A process is simply a running program.

Example:

* Running `chrome` creates a Chrome process.
* Running `nginx` starts Nginx processes.

### Process Lifecycle

1. Process is created (fork).
2. Process executes work.
3. Process waits for resources if needed.
4. Process finishes and exits.

### Common Process States

| State        | Meaning                                           |
| ------------ | ------------------------------------------------- |
| Running (R)  | Currently using CPU                               |
| Sleeping (S) | Waiting for an event or resource                  |
| Stopped (T)  | Paused manually or by signal                      |
| Zombie (Z)   | Process finished but parent has not cleaned it up |

### Why DevOps Engineers Care

* High CPU usage usually comes from running processes.
* Memory issues are caused by processes consuming resources.
* Crashed applications are often process-related problems.

---

## What is systemd?

systemd is the service manager used by most Linux systems.

It helps:

* Start services during boot.
* Stop and restart services.
* Monitor service health.
* Manage system logs.

Example services:

* nginx
* docker
* sshd
* jenkins

### Why systemd Matters

* Automatically restarts failed services.
* Makes troubleshooting easier.
* Standard way to manage services in production Linux servers.

---

## Daily Commands I Will Use

### Check Running Processes

```bash
ps -ef
```

### Live Resource Monitoring

```bash
top
```

### Check Service Status

```bash
systemctl status nginx
```

### Start or Restart a Service

```bash
systemctl restart nginx
```

### View Logs

```bash
journalctl -u nginx
```

---

## My Key Takeaway

Linux systems run applications as processes. The kernel manages resources, user space runs applications, and systemd manages services. Understanding these basics helps troubleshoot server issues, service failures, CPU spikes, and application crashes more effectively.
