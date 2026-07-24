#  OS & Main Functions

---

## What is an Operating System?

An **Operating System (OS)** is **system software** that acts as an **intermediary** between the user/applications and the computer hardware.

>  **One-liner for interviews**: "An OS manages hardware resources and provides services to application programs."

```
┌──────────────────────────────┐
│         User / Apps          │  ← You run Chrome, Python, etc.
├──────────────────────────────┤
│      Operating System        │  ← OS manages everything below
├──────────────────────────────┤
│    Hardware (CPU, RAM, I/O)  │  ← Physical devices
└──────────────────────────────┘
```

---

##  5 Main Functions of an OS

### 1.  Resource Management (CPU, Memory, I/O)

The OS decides **who gets what resource and when**.

| Resource | OS Role |
|----------|---------|
| CPU | Schedules which process runs |
| Memory (RAM) | Allocates and deallocates memory |
| I/O Devices | Manages access to disk, keyboard, mouse, etc. |

**Interview angle**: Without the OS, two programs might both try to write to the same memory address — chaos! The OS prevents this.

---

### 2.  Process Management

A **process** is a program in execution. The OS:
- Creates and terminates processes
- Switches the CPU between processes (**context switching**)
- Handles inter-process communication (IPC)
- Prevents one process from crashing another

```
OS creates Process A → runs it → pauses it → runs Process B → resumes A
```

---

### 3.  File System Management

The OS provides an abstraction over raw disk storage:
- Organizes data into **files and directories**
- Controls **permissions** (who can read/write)
- Manages **file creation, deletion, reading, writing**

> Without OS: you'd write raw sectors on a disk.  
> With OS: you just call `open("file.txt")`.

---

### 4.  Security & Protection

The OS protects:
- **Processes from each other** (one program can't read another's memory)
- **Users from each other** (multi-user systems)
- **System from unauthorized access** (authentication)

Mechanisms:
- Kernel mode vs User mode (hardware-level protection)
- Access control lists (ACLs)
- Memory protection (page tables with permissions)

---

### 5.  Abstraction (Hiding Hardware Complexity)

The OS hides the messy details of hardware behind simple interfaces (APIs / System Calls).

```
App calls: read(fd, buffer, size)
     ↓
OS translates → sends DMA request → disk controller → retrieves sectors → returns data
```

The app doesn't care HOW the disk works — the OS handles it.

> **Analogy**: A car's steering wheel is an abstraction over the complex steering mechanism underneath.

---

##  Summary Table

| Function | What it Does | Why It Matters |
|----------|-------------|----------------|
| Resource Management | Allocates CPU, RAM, I/O fairly | Prevents starvation, maximizes utilization |
| Process Management | Runs & switches between programs | Enables multitasking |
| File System Management | Organizes data storage | Simple, universal data access |
| Security & Protection | Isolates processes & users | Prevents crashes and data theft |
| Abstraction | Hides hardware complexity | Simplifies programming |

---

##  Interview Questions & Answers

**Q: What is an OS? Why is it needed?**
> An OS is system software that manages hardware resources and provides services to programs. Without it, every program would need to directly control hardware, leading to conflicts, crashes, and massive complexity.

**Q: What are the main functions of an OS?**
> Resource management, process management, file system management, security & protection, and abstraction of hardware.

**Q: How does an OS provide security?**
> Through kernel/user mode separation, memory protection via page tables, user authentication, and access control lists.

---

*Next → [Types of OS](./02-Types-of-OS.md)*
