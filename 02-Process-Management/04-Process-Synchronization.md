# 🔐 Process Synchronization & Critical Section

---

## Why Synchronization?

When multiple processes/threads **share resources** (shared memory, files, devices), they can **interfere** with each other — leading to **incorrect results**.

### The Race Condition Problem

```c
// Shared variable
int counter = 0;

// Process A runs:        // Process B runs (simultaneously):
counter = counter + 1;    counter = counter + 1;
```

At the hardware level, `counter = counter + 1` is **three steps**:

```
LOAD  R1, counter    // R1 = 0
ADD   R1, 1          // R1 = 1
STORE counter, R1    // counter = 1
```

If two processes interleave:
```
Process A: LOAD  R1 = 0
Process B: LOAD  R2 = 0     ← reads before A writes back!
Process A: ADD   R1 = 1
Process B: ADD   R2 = 1
Process A: STORE counter = 1
Process B: STORE counter = 1  ← overwrites A's result!
```
**Result: counter = 1 instead of 2 ← WRONG!**

This is a **race condition** — the result depends on the execution order.

---

## 🎯 The Critical Section Problem

The **Critical Section (CS)** is the portion of code that **accesses shared resources**.

```c
// General structure of any process:

while (true) {
    entry section;       // Request permission to enter CS
    
    // ── CRITICAL SECTION ──
    // Access shared resource here
    // ─────────────────────
    
    exit section;        // Release permission
    
    remainder section;   // Non-critical code
}
```

The goal: **design the entry and exit sections** so that concurrent processes don't corrupt shared data.

---

## ✅ Three Requirements for a Valid CS Solution

A correct solution to the critical section problem MUST satisfy all three:

### 1. Mutual Exclusion
> **Only ONE process can be in the critical section at a time.**

```
If P1 is in CS → P2, P3, ... cannot enter CS.
```

- This is the most fundamental requirement.
- Prevents race conditions.

---

### 2. Progress
> **If no process is in the CS and some processes want to enter, ONE of them must be allowed in — in finite time.**

- The system must not be stuck with CS empty but no one allowed in.
- The decision of who enters cannot be postponed indefinitely.
- Only processes **not in their remainder section** can participate in the decision.

```
CS is empty + P1 and P2 both want to enter
→ Eventually, one of them MUST be allowed in (can't block forever)
```

---

### 3. Bounded Waiting
> **There is a limit on how many times OTHER processes can enter the CS after a process has requested entry and before that request is granted.**

- Prevents **starvation** within the CS entry mechanism.
- If P1 requests entry, P2 and P3 can't enter the CS more than N times before P1 gets in.

```
P1 requests CS entry.
P2 and P3 can each enter at most K more times before P1 must be allowed in.
```

---

## Summary of 3 Requirements

| Requirement | Prevents | Key Idea |
|-------------|---------|---------|
| **Mutual Exclusion** | Race conditions | Only 1 in CS at a time |
| **Progress** | Deadlock (CS unused but blocked) | Empty CS → someone enters eventually |
| **Bounded Waiting** | Starvation | No process waits forever |

---

## ❌ Simple (Broken) Attempts at CS Solution

### Attempt 1: Disable Interrupts
```
entry: disable_interrupts()
// Critical Section
exit: enable_interrupts()
```
- ✅ Works on single-core
- ❌ Doesn't work on multi-core (other CPUs still run)
- ❌ Dangerous — user code disabling interrupts

---

### Attempt 2: Simple Flag (Broken)
```c
bool lock = false;

entry:
    while (lock == true);  // spin until free
    lock = true;

exit:
    lock = false;
```
- ❌ **Violates mutual exclusion!**
- Two processes can both see `lock = false` and both enter CS simultaneously (race condition on the flag itself!)

---

### Attempt 3: Peterson's Algorithm (Correct ✅)

For **2 processes** (P0 and P1):

```c
int turn;          // whose turn it is
bool flag[2];      // flag[i] = true if Pi wants to enter CS

// Process P0:
flag[0] = true;    // "I want to enter"
turn = 1;          // "But you go first if you want"
while (flag[1] && turn == 1);  // wait if P1 wants AND it's P1's turn
// --- CRITICAL SECTION ---
flag[0] = false;   // "I'm done"
```

Peterson's satisfies all 3 requirements but is **software-only** and works for 2 processes only.

---

## 🔧 Hardware Solutions

Modern CPUs provide **atomic instructions**:

| Instruction | What it does |
|-------------|-------------|
| `test_and_set` | Atomically reads and sets a lock variable |
| `compare_and_swap (CAS)` | Atomically compare and conditionally update |
| `fetch_and_add` | Atomically increment |

These are used to build **mutex locks and semaphores** (see next files).

---

## 🎯 Interview Questions & Answers

**Q: What is the critical section problem?**
> The critical section problem is about designing a protocol so that when multiple processes share data, only one process accesses the shared data (critical section) at a time, preventing race conditions.

**Q: What are the three requirements for a critical section solution?**
> 1. **Mutual Exclusion**: Only one process in CS at a time. 2. **Progress**: If CS is empty, one of the waiting processes must be allowed in finite time. 3. **Bounded Waiting**: A process waiting to enter CS will eventually get in — it won't wait forever.

**Q: What is a race condition?**
> A race condition occurs when the result of a computation depends on the relative timing/order of concurrent processes. Two processes reading and writing shared data without synchronization can produce incorrect results depending on how their operations interleave.

**Q: What is Peterson's solution?**
> A software-based solution to the critical section problem for 2 processes, using a `turn` variable and a `flag` array. It satisfies all three requirements but only works for 2 processes and requires busy waiting.

**Q: Why can't we just disable interrupts to solve the critical section problem?**
> Disabling interrupts works on single-core systems but fails on multi-core systems where other CPUs continue running and can still access shared data. It's also dangerous to give user processes the ability to disable system interrupts.

---

*← [Starvation and Aging](./03-Starvation-and-Aging.md) | Next → [Mutex and Spinlocks](./05-Mutex-and-Spinlocks.md)*
