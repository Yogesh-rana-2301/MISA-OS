# 💥 Page Fault

---

## What is a Page Fault?

A **page fault** is a hardware exception that occurs when a process tries to access a **virtual page that is not currently loaded in RAM**.

> A page fault is **not an error** (usually) — it's a normal part of virtual memory operation.

```
Process tries to access address 0x3000
  ↓
MMU checks page table → valid bit = 0 (not in RAM!)
  ↓
Hardware raises PAGE FAULT exception
  ↓
OS takes over (page fault handler)
  ↓
OS loads page from disk → retries instruction
```

---

## What Triggers a Page Fault?

| Trigger | Description |
|---------|-------------|
| **First access to a page** | Demand paging — page never loaded yet |
| **Page was swapped out** | OS removed it from RAM to free space (paging out) |
| **Access violation** | Process accesses invalid address → **Segfault** (fatal, not recoverable!) |

```
Page fault types:
  Minor (soft) fault → page exists but not mapped yet (e.g., COW, shared mem)
  Major (hard) fault → page must be read from disk (slow, I/O needed)
  Invalid fault      → address is truly illegal → SIGSEGV sent to process
```

---

## Page Fault Handling — Step by Step

```
┌─────────────────────────────────────────────────────────┐
│  STEP 1: Process accesses virtual address                │
│           ↓                                             │
│  STEP 2: MMU checks page table                          │
│           Valid bit = 0 → raises page fault interrupt   │
│           ↓                                             │
│  STEP 3: OS page fault handler runs (kernel mode)       │
│           Check if address is valid:                    │
│             Invalid → send SIGSEGV → process dies ❌    │
│             Valid   → continue ✅                       │
│           ↓                                             │
│  STEP 4: Find a free frame in RAM                       │
│           If no free frame → run Page Replacement algo  │
│           (evict a victim page — possibly write to disk) │
│           ↓                                             │
│  STEP 5: Load required page from disk into free frame   │
│           (disk I/O — slow, process put to WAITING)     │
│           ↓                                             │
│  STEP 6: Update page table                              │
│           page_table[page_num].frame = new_frame        │
│           page_table[page_num].valid = 1                │
│           ↓                                             │
│  STEP 7: Restart the faulting instruction               │
│           Process moves WAITING → READY → RUNNING       │
└─────────────────────────────────────────────────────────┘
```

---

## Timeline View

```
──────────────────────────────────────────────────────────→ time

Process:  [running] [fault!] [WAITING──────────────] [running again]
                       ↑             ↑                    ↑
                  Page fault     Disk I/O           Page loaded,
                  detected       (loading page)      retry instruction

Other processes run here while this one waits for disk!
```

---

## Performance Impact

**Effective Memory Access Time (EAT)** with page faults:

```
EAT = (1 − p) × memory_access_time + p × page_fault_time

Where:
  p = page fault rate (probability of fault per access)
  memory_access_time ≈ 100ns
  page_fault_time    ≈ 8,000,000ns (8ms disk seek!)

Example:
  p = 1/1000 (1 fault per 1000 accesses)
  EAT = 0.999 × 100ns + 0.001 × 8,000,000ns
      = 99.9ns + 8,000ns
      = ~8,100ns  ← 81× slower than no faults!
```

> **Key takeaway**: Even a small page fault rate dramatically degrades performance. This is why minimizing page faults is critical.

---

## Page Fault Rate Reduction

| Technique | How it Helps |
|-----------|-------------|
| **Good page replacement** | Keep frequently used pages in RAM |
| **Prepaging** | Load pages likely to be needed soon (predict) |
| **Larger RAM** | More pages fit → fewer evictions |
| **Locality of reference** | Programs that access nearby memory cause fewer faults |
| **Working set model** | Keep each process's "working set" in RAM |

---

## 🎯 Interview Questions & Answers

**Q: What is a page fault?**
> A page fault is a hardware exception raised when a process accesses a virtual page that is not currently in RAM (valid bit = 0 in page table). The OS page fault handler loads the required page from disk into RAM and retries the instruction.

**Q: Walk me through what happens during a page fault.**
> 1. Process accesses a virtual address. 2. MMU finds valid bit = 0 in page table → raises page fault. 3. OS checks if the address is valid (invalid → SIGSEGV). 4. OS finds a free frame (or evicts a page). 5. OS loads the page from disk into the frame (process waits — disk I/O). 6. OS updates page table (frame number + valid bit = 1). 7. Faulting instruction is retried.

**Q: What is the difference between a page fault and a segmentation fault?**
> A page fault (recoverable) means a valid page is not currently in RAM — the OS loads it and continues. A segmentation fault (SIGSEGV, fatal) means the process accessed an invalid/illegal address that doesn't belong to its address space — the OS terminates the process.

**Q: Why does a page fault cause the process to go to the WAITING state?**
> Disk I/O takes ~8ms while CPU operations take ~100ns. Rather than wasting CPU time waiting, the OS puts the faulting process in the WAITING state and runs another process. When disk I/O completes, the process moves back to READY.

---

*← [Virtual Memory](./04-Virtual-Memory.md) | Next → [Page Replacement](./06-Page-Replacement.md)*
