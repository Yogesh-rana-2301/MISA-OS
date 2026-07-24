#  Kernel Mode vs User Mode

---

##  Why Two Modes?

The CPU needs to protect the OS from misbehaving programs.
Without protection, a buggy or malicious program could:
- Crash the entire system
- Access another process's memory
- Take over hardware devices

The solution: **hardware-enforced dual mode operation**.

```
┌──────────────────────────────────────────────┐
│  USER MODE (Ring 3)                          │
│  Your apps: Chrome, Python script, games     │
│  Can only run non-privileged instructions    │
├──────────────────────────────────────────────┤
│  KERNEL MODE (Ring 0)                        │
│  OS kernel: process scheduler, memory mgr   │
│  Can run ALL instructions + access hardware  │
└──────────────────────────────────────────────┘
```

> The CPU has a **mode bit** in a register:
> - `0` = Kernel Mode
> - `1` = User Mode

---

##  Privileged vs Non-Privileged Instructions

### Non-Privileged Instructions (User Mode allowed )

Any program can execute these safely:

| Instruction | Example |
|-------------|---------|
| Arithmetic & logic | `ADD`, `SUB`, `AND`, `OR` |
| Data movement | `MOV`, `LOAD`, `STORE` (to own memory) |
| Control flow | `JMP`, `CALL`, `RET` |
| Comparison | `CMP`, `TEST` |

---

### Privileged Instructions (Kernel Mode only )

Only the OS kernel can execute these:

| Instruction | Why Dangerous |
|-------------|--------------|
| `IN` / `OUT` | Direct I/O port access |
| `HLT` | Halt the CPU |
| `STI` / `CLI` | Enable/disable interrupts |
| `LGDT` / `LIDT` | Modify global descriptor table |
| Memory management registers | Modify page tables (CR3 on x86) |

> If a user-mode program tries a privileged instruction → **hardware trap** → OS terminates the process.

---

##  Mode Switching

### User Mode → Kernel Mode (3 ways)

```
1. SYSTEM CALL    ← App deliberately asks OS for a service
2. INTERRUPT      ← Hardware device signals CPU (e.g., keyboard press)
3. EXCEPTION/TRAP ← Program error (divide by zero, invalid memory access)
```

### Detailed Flow of a System Call Mode Switch

```
App (User Mode)
    │
    │  calls read() ─── library wrapper (glibc)
    │
    ▼
    SYSCALL instruction  ← triggers mode switch
    │
    ▼
CPU switches mode bit: 1 → 0
    │
    ▼
Kernel Mode
    │  OS reads from disk / handles request
    │
    ▼
SYSRET instruction ← return to user mode
    │
CPU switches mode bit: 0 → 1
    │
    ▼
App (User Mode) resumes
```

---

##  Why Protection is Needed

### Problem Without Protection

```
Process A (buggy):
  MOV [0x1000], "HACKED"    ← overwrites Process B's memory
  OUT 0x60, 0xFF            ← corrupts keyboard controller
  HLT                       ← freezes the entire machine
```

### Protection Mechanisms

| Mechanism | How It Protects |
|-----------|----------------|
| **Dual mode** | Privileged instructions only in kernel mode |
| **Memory protection** | Page tables — each process sees only its own virtual address space |
**I/O protection** | All I/O through OS system calls only |
| **CPU protection** | Timer interrupt prevents a process from hogging CPU forever |

### Consequences Without Protection

- Process A can read Process B's passwords
- Malware can access disk directly
- A crash in one program = full system crash
- No multi-user security possible

---

##  Kernel vs User Space

| | User Space | Kernel Space |
|-|-----------|-------------|
| **Who runs here** | Your apps (Chrome, Python) | OS kernel |
| **Mode bit** | 1 (User mode) | 0 (Kernel mode) |
| **Memory access** | Only own virtual memory | All physical memory |
| **Crash impact** | Only that process crashes | System crash (kernel panic) |
| **Example** | `printf()`, `malloc()` | `read()`, `fork()`, `write()` |

---

##  Summary

```
User Mode ──── SYSCALL / Interrupt / Exception ──→ Kernel Mode
              (saves context: PC, registers, PSW)
              
Kernel Mode ───────────── SYSRET / IRET ──────→ User Mode
              (restores context)
```

**Key Point**: Mode switching is **expensive** (saves/restores context, flushes pipelines) — which is why we minimize unnecessary system calls.

---

##  Interview Questions & Answers

**Q: What is the difference between kernel mode and user mode?**
> Kernel mode (Ring 0) allows execution of all instructions and direct hardware access. User mode (Ring 3) restricts programs to non-privileged instructions only. The CPU has a mode bit that determines the current mode. This separation prevents user programs from corrupting the OS or each other.

**Q: How does the CPU switch from user mode to kernel mode?**
> Via a system call, hardware interrupt, or exception/trap. The CPU saves the current context (PC, registers), switches the mode bit from 1 to 0, and jumps to the OS handler. After handling, it restores context and switches back to user mode.

**Q: What happens if a user program executes a privileged instruction?**
> The hardware raises a trap/exception. The OS handles it, typically terminating the offending process with a "general protection fault" (like SIGSEGV on Linux).

**Q: Why is kernel mode protection important?**
> Without it, any process could crash the entire system, read other processes' private data, or directly manipulate hardware — making multi-process, multi-user systems impossible.

---

*← [Multiprogramming vs Multitasking](./03-Multiprogramming-vs-Multitasking.md) | Next → [Process and Its States](./05-Process-and-States.md)*
