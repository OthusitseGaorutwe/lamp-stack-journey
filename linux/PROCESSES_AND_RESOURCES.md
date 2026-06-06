# Linux Processes & System Administration

A practical reference for Linux process management, resource control, and common sysadmin commands.

---

## Table of Contents

- [Processes & Hierarchy](#processes--hierarchy)
- [Process States](#process-states)
- [Resources Per Process](#resources-per-process)
- [Killing & Terminating Processes](#killing--terminating-processes)
- [Running Processes with nohup](#running-processes-with-nohup)
- [Kernel & System Parameters](#kernel--system-parameters)
- [Useful Inspection Commands](#useful-inspection-commands)
- [Troubleshooting](#troubleshooting)

---

## Processes & Hierarchy

Every process in Linux descends from **PID 1** (`systemd`). New processes are created with:

```bash
fork()   # clone the parent process
exec()   # load a new program into the clone
```

Key process types:

| Type | Example | Description |
|---|---|---|
| Init process | `systemd` (PID 1) | Ancestor of all user processes |
| Kernel threads | `kthreadd` | Low-level kernel work |
| Daemons | `sshd`, `nginx` | Background services |
| Shell children | `bash` → script | Spawned via fork + exec |

---

## Process States

The kernel scheduler moves processes between these states:

| State | Symbol | Meaning |
|---|---|---|
| Running | `R` | Actively executing on a CPU core |
| Runnable | `R` | Ready, waiting in the scheduler queue |
| Sleeping (interruptible) | `S` | Waiting on I/O, signal, or lock |
| Sleeping (uninterruptible) | `D` | Blocked in kernel I/O — cannot be killed |
| Zombie | `Z` | Exited but not yet reaped by parent |
| Stopped | `T` | Paused via SIGSTOP or Ctrl+Z |


---

## Resources Per Process

The kernel tracks six main resource categories per process:

| Resource | Details |
|---|---|
| **CPU / Scheduling** | CFS scheduler, priority, `nice` value (−20 to +19) |
| **Virtual Memory** | VMAs — heap, stack, code segments, `mmap` regions |
| **File Descriptors** | 0 = stdin, 1 = stdout, 2 = stderr + any open files/sockets |
| **Signals** | Async notifications: `SIGKILL`, `SIGTERM`, `SIGCHLD`, etc. |
| **IPC** | Pipes, Unix/TCP sockets, shared memory, message queues |
| **Namespaces / cgroups** | Isolation and resource limits (used by Docker, containers) |

---

## Killing & Terminating Processes

### The Golden Rule

Always try **SIGTERM** first (graceful), then escalate to **SIGKILL** (force) only if needed.

```bash
kill <PID>          # SIGTERM — asks process to shut down cleanly
sleep 3
kill -9 <PID>       # SIGKILL — kernel forcibly removes it
```

> **Why not always use `-9`?** SIGKILL bypasses the process — no chance to flush buffers, close sockets, or remove lock files. This can corrupt databases or leave stale PID/lock files.

### Common Signals

| Signal | Number | Description |
|---|---|---|
| `SIGTERM` | 15 | Graceful shutdown request (default for `kill`) |
| `SIGKILL` | 9 | Force kill — uncatchable, unblockable |
| `SIGHUP` | 1 | Hangup / reload config |
| `SIGINT` | 2 | Interrupt from keyboard (Ctrl+C) |
| `SIGSTOP` | 19 | Pause process (uncatchable) |
| `SIGCONT` | 18 | Resume a stopped process |
| `SIGCHLD` | 17 | Sent to parent when child exits |

### Kill Commands

```bash
# By PID
kill <PID>                  # SIGTERM (graceful)
kill -9 <PID>               # SIGKILL (force)
kill -HUP <PID>             # reload config
kill -STOP <PID>            # pause
kill -CONT <PID>            # resume

# By name (no PID needed)
killall nginx               # exact name match
pkill -f "python script.py" # match against full command line
pkill -u username           # kill all processes by a user

# Pipeline pattern
pgrep -f myapp | xargs kill -9
```

### Finding a PID

```bash
pidof nginx              # exact binary name
pgrep -l python          # list PIDs + names
ps aux | grep nginx      # full snapshot with details
```

---

## Running Processes with nohup

`nohup` keeps a process running after the terminal closes by blocking `SIGHUP`.

### What that message means

```
nohup: ignoring input and appending output to 'nohup.out'
```

- **"ignoring input"** — stdin redirected to `/dev/null` (terminal is gone)
- **"appending output to 'nohup.out'"** — stdout/stderr saved to file since the terminal will close

### Recommended usage

```bash
# Redirect output yourself — suppresses the message
nohup ./myscript.sh > app.log 2>&1 &
```

The trailing `&` backgrounds the process so your shell prompt returns immediately.

### Output control

```bash
nohup cmd > out.log 2>&1        # stdout + stderr to one file
nohup cmd > out.log 2> err.log  # split stdout and stderr
nohup cmd > /dev/null 2>&1      # discard all output silently
```

### Monitor a nohup process

```bash
jobs -l                  # if still in same shell session
ps aux | grep myscript
tail -f app.log          # follow live output
```

### Alternatives to nohup

| Tool | Best for |
|---|---|
| `tmux` / `screen` | Full persistent sessions you can reattach to |
| `disown` | Detach an already-running foreground job |
| `systemd` service | Proper daemon with restart policies and logging |
| `setsid` | New session, immune to SIGHUP without nohup |

---

    ## Kernel & System Parameters

### Modify with sysctl (preferred)

```bash
# Read a value
sysctl net.ipv4.ip_forward

# Set temporarily (lost on reboot)
sudo sysctl -w net.ipv4.ip_forward=0

# Set permanently
echo "net.ipv4.ip_forward = 0" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p    # reload from /etc/sysctl.conf
```

### Write directly to /proc/sys

# CORRECT — use tee so the write happens with sudo privileges
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```

### Common sysctl parameters

| Parameter | Purpose |
|---|---|
| `net.ipv4.ip_forward` | Enable/disable IP routing (needed for Docker, VPNs, NAT) |

---

## Useful Inspection Commands

```bash
# Snapshot of all processes
ps aux

# Live CPU/memory view
top
htop                        # more user-friendly

# Process tree
pstree -p

# Open files and sockets for a process
lsof -p <PID>

# System calls made by a process (debug/trace)
strace -p <PID>

# Kernel data for a specific PID
ls /proc/<PID>/
cat /proc/<PID>/status       # process info
cat /proc/<PID>/cmdline      # full command line
cat /proc/<PID>/fd/          # open file descriptors

# Memory map
cat /proc/<PID>/maps
```


### disown vs nohup

Use `disown` when the process is *already running*:
```bash
Ctrl+Z          # pause the foreground job
bg              # resume it in the background
disown          # detach it from the shell's job table
```
Use `nohup` when *starting* a new process.
