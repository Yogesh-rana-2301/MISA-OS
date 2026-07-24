#  Process & Its States

---

## Process vs Program

This is a **fundamental distinction** — interviewers love asking this!

| | Program | Process |
|-|---------|---------|
| **Definition** | Static set of instructions stored on disk | Program in execution (active entity) |
| **State** | Passive (just a file) | Active (has CPU, memory, resources) |
| **Existence** | Exists on disk (`.exe`, `.py`) | Exists in RAM during execution |
| **Multiple instances** | One copy of program | One program → Multiple processes |
| **Resources** | None | Has: memory, CPU time, file handles |

```
program.exe  ──── OS loads ────→  Process (PID 1234)
(on disk)                         (in RAM, running)

Same program.exe can create:
 Process PID 1234  (user A running it)
 Process PID 5678  (user B running it)  ← 2 processes, 1 program
```

> **Analogy**: A recipe (program) vs. cooking that recipe (process). You can cook the same recipe multiple times simultaneously.

---

##  Process Lifecycle & States

A process transitions through **5 states** during its lifetime:

```
                    ┌─────────────────────────────────────────────────┐
                    │                                                 │
  ┌─────┐  admitted ┌───────┐  scheduler ┌─────────┐  I/O wait   ┌─────────┐
  │ NEW │ ─────────▶│ READY │ ──dispatch─▶│ RUNNING │ ───────────▶│ WAITING │
  └─────┘           └───────┘            └─────────┘             └─────────┘
                        ▲                    │  │                     │
                        │    preempt/        │  │                     │ I/O done
                        └────time quantum────┘  │                     │
                                                │ exit            ┌───▼────┐
                                                └────────────────▶│TERMINATED│
                                                                  └────────┘
```

---

### State Descriptions

| State | Description | Who controls |
|-------|-------------|-------------|
| **New** | Process is being created | OS |
| **Ready** | Process loaded in memory, waiting for CPU | Scheduler |
| **Running** | Process is being executed by CPU | CPU |
| **Waiting (Blocked)** | Process waiting for I/O or event | I/O subsystem |
| **Terminated** | Process has finished execution | OS (cleanup) |

---

### State Transitions Explained

| Transition | Trigger | Example |
|-----------|---------|---------|
| New → Ready | Process created, loaded into memory | `fork()` returns in parent, new process enters Ready |
| Ready → Running | CPU scheduler selects this process | Scheduler picks from Ready queue |
| Running → Ready | Time quantum expires (preemption) | 20ms timer fires, OS preempts process |
| Running → Waiting | Process requests I/O or waits for event | `read()` from disk, waiting for input |
| Waiting → Ready | I/O completes, event occurs | Disk read done, process re-enters Ready queue |
| Running → Terminated | Process calls `exit()` or crashes | Program finishes or segfault |

---

### Ready Queue vs Waiting Queue

```
CPU
 ▲
 │
 │  ┌──────────────────────────────────────────────────┐
 │  │           READY QUEUE                            │
 │  │  P3 → P7 → P1 → P5  (waiting for CPU)           │
 │  └──────────────────────────────────────────────────┘
 │
 │  ┌──────────────────────────────────────────────────┐
 │  │           WAITING QUEUE (per device)             │
 │  │  Disk queue:    P2 → P6                          │
 │  │  Network queue: P4                               │
 │  └──────────────────────────────────────────────────┘
```

---

##  PCB — Process Control Block

The OS maintains a **PCB** for every process. It's the OS's "file" on each process.

> Think of PCB as a **student record** — the school (OS) keeps a file on every student (process).

### PCB Contents

```
┌────────────────────────────────────┐
│         PCB (PID: 1234)            │
├────────────────────────────────────┤
│ Process ID (PID)        │ 1234     │
│ Process State           │ Running  │
│ Program Counter (PC)    │ 0x40A1B2 │  ← Next instruction to execute
│ CPU Registers           │ EAX=5... │  ← Saved during context switch
│ CPU Scheduling Info     │ Priority │  ← For scheduler
│ Memory Management Info  │ Page table pointer │
│ I/O Status Info         │ Open files, devices │
│ Accounting Info         │ CPU time used │
│ Parent PID (PPID)       │ 1000     │
└────────────────────────────────────┘
```

### Why PCB Matters

During a **context switch**, the OS:
1. **Saves** all CPU registers + PC into the outgoing process's PCB
2. **Loads** all CPU registers + PC from the incoming process's PCB
3. Resumes the new process from exactly where it left off

```
Context Switch:
P1 running → Save P1's context to PCB1 → Load P2's context from PCB2 → P2 running
```

---

##  Key Concepts to Remember

### At any time:
- **ONE** process can be in **Running** state per CPU core
- **MANY** processes can be in **Ready** or **Waiting** state

### Preemption:
- **Preemptive OS**: Can interrupt a running process (modern OSes)
- **Non-preemptive OS**: Process runs until it voluntarily gives up CPU (old)

---

##  Interview Questions & Answers

**Q: What is the difference between a process and a program?**
> A program is a static set of instructions on disk. A process is a program in execution — an active entity with its own memory space, CPU registers, and OS resources. Multiple processes can run from the same program.

**Q: Explain process states with a diagram.**
> *(Draw the 5-state diagram)*: New → Ready → Running → (either) Waiting or Terminated. From Waiting → Ready when I/O completes. From Running → Ready when preempted.

**Q: What is a PCB and what does it contain?**
> PCB (Process Control Block) is a data structure the OS maintains for each process. It stores: PID, process state, program counter, CPU registers, memory info, I/O status, scheduling priority, and accounting info. It's used during context switches to save and restore process state.

**Q: Can a process go directly from Waiting to Running?**
> No. A process must go Waiting → Ready → Running. It goes to the Ready queue after I/O completes, then waits for the CPU scheduler to select it.

**Q: What is context switching?**
> The OS saves the state of the current process (into its PCB) and loads the state of the next process (from its PCB). This allows multiple processes to share one CPU. It's overhead — no useful work happens during a context switch.

---

*← [Kernel and User Mode](./04-Kernel-and-User-Mode.md) | Next → [Function Call vs System Call](./06-Function-Call-vs-System-Call.md)*
