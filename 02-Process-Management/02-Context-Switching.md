# 🔀 Context Switching

---

## What is a Context Switch?

A **context switch** is the process of **saving the state of the current process** and **loading the state of the next process** so the CPU can switch between them.

> **Analogy**: Like a student saving their homework notes (bookmark + state), then picking up a different book to work on — and later returning to the first book exactly where they left off.

---

## 🔄 What Happens During a Context Switch

### Step-by-Step Flow

```
CPU running Process A
        │
        │  [Trigger: timer interrupt / I/O wait / higher priority arrives]
        ▼
┌────────────────────────────────────────┐
│  STEP 1: Save Process A's context      │
│  → Save: PC, registers, PSW, stack ptr │
│  → Store into A's PCB                  │
└────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────┐
│  STEP 2: Update A's state              │
│  → Change PCB state: Running → Ready   │
│  → OR Running → Waiting (if I/O)       │
└────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────┐
│  STEP 3: Scheduler selects Process B   │
│  → Picks from Ready queue              │
└────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────┐
│  STEP 4: Load Process B's context      │
│  → Load: PC, registers, PSW, stack ptr │
│  → Read from B's PCB                   │
│  → Change B's state: Ready → Running   │
└────────────────────────────────────────┘
        │
        ▼
CPU running Process B (from where it left off)
```

---

## 📋 What Gets Saved/Restored (Context)

The **context** of a process is everything the CPU needs to resume it:

| Category | What's Saved |
|----------|-------------|
| **Program Counter (PC)** | Address of next instruction to execute |
| **CPU Registers** | General purpose: EAX, EBX, ECX, EDX etc. |
| **Stack Pointer (SP)** | Top of process stack |
| **Base/Limit Registers** | Memory bounds |
| **Program Status Word (PSW)** | Flags: zero, carry, sign, overflow |
| **Memory Maps** | Page table pointer (CR3 on x86) |
| **I/O State** | Open file descriptors (stored in PCB) |

All of this is stored in the process's **PCB (Process Control Block)**.

---

## ⏱️ Timeline View

```
Time ──────────────────────────────────────────────────────────────→

Process A: [RUNNING──────][idle─idle─idle─idle─idle][RUNNING──]
                         ↑                         ↑
Context Switch START     │                         │
                     [SWITCH]                  [SWITCH]
                         ↓                         ↓
Process B: [idle─idle─idle][RUNNING──────────────][idle──────]

            ↑           ↑
     No useful work happens here (overhead!)
```

---

## 💸 Overhead — Why Context Switches Are Expensive

### Direct Costs

| Cost | Why |
|------|-----|
| Save/restore registers | ~dozens of memory writes |
| Update PCB, scheduler data structures | OS bookkeeping |
| Mode switch (user → kernel → user) | Two mode switches per switch |

### Indirect Costs (Often Larger!)

| Cost | Why |
|------|-----|
| **Cache flush** | Process A's data in L1/L2 cache → now cold for Process B |
| **TLB flush** | Translation Lookaside Buffer must be invalidated (different address space) |
| **Pipeline flush** | CPU pipeline stalls during mode switch |

> **Typical cost**: A context switch takes **1–10 microseconds** on modern hardware.
> If you do 1000 switches/sec → **1–10ms wasted per second** just on switching.

---

## 🔧 What Triggers a Context Switch?

| Trigger | Description |
|---------|-------------|
| **Timer interrupt** | Time quantum expired (Round Robin) |
| **I/O request** | Process calls `read()`/`write()` → blocks |
| **Higher priority arrival** | Preemptive priority scheduling |
| **Process terminates** | OS must pick next process |
| **System call** | Some system calls block the calling process |
| **Interrupt** | Hardware interrupt requires OS attention |

---

## 🧠 Context Switch vs Mode Switch

These are **different** concepts that often happen together:

| | Mode Switch | Context Switch |
|-|------------|----------------|
| **What changes** | CPU privilege level (user ↔ kernel) | Which process is running |
| **Saves context?** | No (just changes mode bit) | Yes (saves all registers, PC) |
| **Cost** | ~100ns | ~1–10μs |
| **Happens during** | System calls, interrupts | Process scheduling |
| **Example** | App calls `read()` → kernel handles it → returns | OS switches from Process A to Process B |

> A context switch **always** involves mode switches (user→kernel to save, kernel→user to resume).
> A mode switch does **not always** cause a context switch (e.g., a fast system call that returns quickly).

---

## ⚙️ Minimizing Context Switch Overhead

Modern systems use several tricks:

| Technique | How it Helps |
|-----------|-------------|
| **Process-specific TLBs** (ASID tags) | Avoid full TLB flush on switch |
| **Larger time quantum** | Fewer switches per unit time |
| **Thread switching** (same process) | Share address space → no TLB/cache flush |
| **CPU pinning** | Pin process to core → cache stays warm |
| **Kernel threads** | Lighter weight than full context switch |

---

## 🎯 Interview Questions & Answers

**Q: What is a context switch?**
> A context switch is when the OS saves the state (PC, registers, memory maps) of the currently running process into its PCB and loads the state of another process from its PCB. This allows multiple processes to share a single CPU.

**Q: What gets saved during a context switch?**
> Program counter, CPU registers (general purpose, stack pointer), program status word (flags), memory management registers (page table pointer). All stored in the process's PCB.

**Q: Why are context switches expensive?**
> Direct cost: saving/restoring dozens of registers, updating OS data structures, two mode switches. Indirect cost (often larger): the CPU cache and TLB contain the old process's data and must be repopulated for the new process — causing cache misses and TLB misses.

**Q: What is the difference between a context switch and a mode switch?**
> A mode switch only changes the CPU privilege level (user↔kernel). A context switch additionally saves/restores all process state and changes which process is running. Context switches always include mode switches; mode switches don't always cause context switches.

**Q: How can we reduce context switch overhead?**
> Use larger time quanta (fewer switches), ASID-tagged TLBs (avoid full TLB flush), prefer thread switching within the same process (shared address space), or CPU pinning to keep cache warm.

---

*← [CPU Scheduling](./01-CPU-Scheduling.md) | Next → [Starvation and Aging](./03-Starvation-and-Aging.md)*
