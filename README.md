# Multi-Container Runtime in C

A lightweight Linux container runtime implemented in C, featuring a user-space supervisor, kernel-space memory monitoring, and a producer-consumer logging system.

---

## Team

| Member | SRN |
|---|---|
| P Mahema Sai | PES2UG24AM107 |
| Nihira Hassan | PES2UG24AM106 |

---

## Build, Load & Run

### Build

```bash
make
```

### Start the Supervisor

```bash
sudo ./engine supervisor ./rootfs-base
```

### Run Containers

```bash
sudo ./engine start alpha ./rootfs-alpha /cpu_hog --soft-mib 40 --hard-mib 80
sudo ./engine start beta  ./rootfs-beta  /io_pulse --soft-mib 40 --hard-mib 80
```

### List Containers

```bash
sudo ./engine ps
```

### View Logs

```bash
sudo ./engine logs alpha
sudo ./engine logs beta
```

### Stop Containers

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
```

### Check Kernel Logs (Memory Limits)

```bash
sudo dmesg | tail
```

Expected output: soft limit warning, followed by hard limit enforcement.

### Verify No Zombie Processes

```bash
ps aux | grep defunct
```

Expected output: no `<defunct>` processes.

---

## Demo

### Multi-Container Supervision & Metadata Tracking

Two containers (`alpha` and `beta`) are launched under a single supervisor process using the CLI. Each container is assigned a unique PID by the OS.

The supervisor maintains metadata including container ID, PID, and current state (running, stopped).

```bash
sudo ./engine ps
```

Displays all active containers, confirming correct lifecycle management and multi-container supervision.

---

### CLI & IPC Communication

The system follows a client-server architecture:

- The supervisor runs as a long-lived process.
- CLI commands communicate with the supervisor over UNIX domain sockets.

When a command like:

```bash
sudo ./engine start alpha ./rootfs-alpha /cpu_hog --soft-mib 40 --hard-mib 80
```

is executed, it is sent to the supervisor, which parses the command, creates a new container process, applies namespace isolation, and registers the container.

---

### Bounded-Buffer Logging & IPC

Each container's output is captured using pipes. A producer-consumer model is used:

- **Producer thread** reads container output.
- Data is stored in a **bounded buffer**.
- **Consumer thread** writes logs to files.

Logs are stored at `logs/alpha.log` and `logs/beta.log`.

```bash
sudo ./engine logs alpha
```

This ensures non-blocking logging, no data loss, and proper synchronization.

---

### Scheduling Experiments

Containers run different workloads:

```bash
sudo ./engine start alpha ./rootfs-alpha /cpu_hog --soft-mib 40 --hard-mib 80
sudo ./engine start beta  ./rootfs-beta  /io_pulse --soft-mib 40 --hard-mib 80
```

| Container | Workload | Behaviour |
|---|---|---|
| alpha | cpu_hog | High CPU usage, continuous execution |
| beta | io_pulse | Lower CPU usage, intermittent execution |

This demonstrates how the Linux CFS scheduler allocates CPU time based on process behaviour — CPU-bound processes consume more CPU time, while I/O-bound processes yield during wait states.

---

### Kernel-Level Memory Monitoring

Containers are started with memory limits:

```bash
--soft-mib 40 --hard-mib 80
```

```bash
sudo dmesg | tail
```

Expected output:

- Soft limit warning message
- Hard limit enforcement (process termination)

This demonstrates kernel-level resource monitoring and IOCTL-based communication between user space and kernel space.

---

### Clean Teardown

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
ps aux | grep defunct
```

Expected: no zombie processes, confirming proper lifecycle management and resource cleanup.

---

## Key Concepts

- Linux namespaces (PID, mount, UTS)
- Process lifecycle management
- Inter-process communication (pipes, UNIX sockets)
- Producer-consumer synchronization
- Kernel module programming
- Memory monitoring via IOCTL
- Linux scheduling (CFS)

---

## System Architecture
CLI
↓
UNIX Socket (IPC)
↓
Supervisor (engine.c)
├── Container creation (clone + namespaces)
├── Logging (pipe → bounded buffer → thread)
├── Metadata tracking
└── ioctl → Kernel module

Kernel Module (monitor.c)
├── Memory tracking
├── Soft limit warning
└── Hard limit enforcement

---

## Notes

- Tested on Linux (Ubuntu)
- Root privileges required for kernel module operations
- Designed for academic learning purposes

---

## Demo Screenshots

[View Screenshots](screenshots.md)

---

## Conclusion

This project demonstrates the design of a simplified container runtime combining user-space process management with kernel-level resource enforcement. It highlights key operating system concepts such as isolation, synchronization, IPC, and scheduling in a practical implementation.
