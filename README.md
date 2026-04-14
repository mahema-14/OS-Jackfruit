# Multi-Container Runtime in C

> A lightweight Linux container runtime featuring a long-running user-space supervisor and a real-time kernel-space memory monitoring module.

![C](https://img.shields.io/badge/Language-C-00d4aa?style=flat-square)
![Linux](https://img.shields.io/badge/Platform-Linux-4f9eff?style=flat-square)
![Kernel](https://img.shields.io/badge/Kernel%20Module-LKM-ff6b6b?style=flat-square)
![IPC](https://img.shields.io/badge/IPC-IOCTL-ffbe5a?style=flat-square)
![License](https://img.shields.io/badge/Purpose-Academic-6b7a8d?style=flat-square)

---

## Overview

This project implements a minimal container runtime entirely in C, demonstrating core operating systems concepts through hands-on systems programming. The runtime manages concurrent workload processes from user space while a companion kernel module tracks memory usage in real time — the two halves communicating cleanly via IOCTL system calls.

---

## Features

- **Runtime Supervisor** — A persistent `engine.c` process that orchestrates and manages all workload lifecycles
- **Kernel Memory Monitor** — A loadable kernel module (`monitor.ko`) that accesses system memory internals directly
- **IOCTL Bridge** — Clean, structured user↔kernel communication defined in a shared header
- **Pluggable Workloads** — Modular CPU, memory, and IO stress generators to simulate diverse container behaviours

---

## Architecture

```
.
├── User Space
│   ├── engine.c          →  Main runtime supervisor (long-running)
│   ├── cpu_hog.c         →  CPU stress generator
│   ├── memory_hog.c      →  Memory stress generator
│   └── io_pulse.c        →  IO workload generator
│
├── Kernel Space
│   ├── monitor.c         →  Kernel module (real-time memory tracking)
│   └── monitor_ioctl.h   →  IOCTL interface (shared user↔kernel contract)
│
└── Build System
    └── Makefile          →  Compiles kernel module and user-space binaries
```

---

## How It Works

1. **Engine starts** — `engine.c` runs as a long-lived supervisor, ready to spawn and manage workload processes.
2. **Workloads execute** — CPU, memory, and IO stress generators run concurrently, simulating real container activity.
3. **Kernel watches** — `monitor.ko` tracks memory usage from inside the kernel with direct access to system internals.
4. **IOCTL reports** — The engine queries the kernel module via `ioctl()` calls to retrieve live memory statistics.

> All user↔kernel communication goes through `monitor_ioctl.h` — no shared memory, no sockets. Just clean system calls.

---

## Setup & Execution

### 1. Build

```bash
# Compiles the kernel module and all user-space binaries
make
```

### 2. Load the Kernel Module

```bash
# Insert the module (requires root)
sudo insmod monitor.ko

# Verify it loaded successfully
lsmod | grep monitor
```

### 3. Run the Engine

```bash
./engine
```

---

## Workload Simulation

| Program | Resource | Description |
|---|---|---|
| `cpu_hog.c` | CPU | Spins tight computation loops to saturate processor cores |
| `memory_hog.c` | Memory | Continuously allocates and accesses heap blocks to pressure RAM |
| `io_pulse.c` | Disk IO | Issues bursty read/write cycles to simulate disk-bound workloads |

---

## Key Concepts Demonstrated

- Linux Loadable Kernel Module (LKM) development
- Process management and supervision in C
- Real-time memory monitoring via kernel APIs
- IOCTL system call design and implementation
- User-space to kernel-space communication
- System resource simulation and stress testing

---

## Demo

Screenshots available here: [View Demo](screenshots.md)

---

## Requirements & Notes

> ⚠️ **Linux only.** Tested on Ubuntu 22.04 LTS. Other distributions may require minor adjustments to the Makefile.

> 🔐 **Root required** for `insmod` and kernel module operations.

---

## Author

**P Mahema Sai** — SRN: PES2UG24AM107  
**Nihira Hassan** — SRN: PES2UG24AM106  
4B-CSE(AIML) · Operating Systems Course
