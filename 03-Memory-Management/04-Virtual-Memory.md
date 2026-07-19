# 🌐 Virtual Memory & Demand Paging

---

## What is Virtual Memory?

**Virtual memory** is a technique that lets a process use **more memory than physically available** in RAM by using disk space as an extension.

> Each process gets its own **large virtual address space** (e.g., 4GB on 32-bit, 256TB on 64-bit), regardless of actual RAM.

```
Process A thinks it has 4GB of RAM.
Process B thinks it has 4GB of RAM.
Actual RAM: only 8GB physical.

How? Most of each process's memory is stored on DISK.
Only the ACTIVE pages are loaded into RAM at any time.
```

---

## Why Virtual Memory is Needed

| Reason | Explanation |
|--------|-------------|
| **Programs larger than RAM** | Run programs that don't fit entirely in RAM |
| **Multiprogramming** | More processes fit "in memory" simultaneously |
| **Isolation** | Each process has its own address space → protection |
| **Simplified programming** | Programmers don't worry about physical memory limits |
| **Memory sharing** | Multiple processes can share physical frames (e.g., shared libraries) |

---

## The Virtual Address Space

```
Virtual Address Space (per process, e.g., 32-bit = 4GB):
0x00000000  ┌────────────────────┐
            │      Text (Code)   │  ← read-only
            ├────────────────────┤
            │   Data (globals)   │
            ├────────────────────┤
            │       Heap         │  → grows upward
            │                    │
            │   (unused/unmapped)│  ← NOT in RAM, NOT on disk
            │                    │
            │       Stack        │  ← grows downward
0xFFFFFFFF  └────────────────────┘
```

Only **mapped** and **actually-used** portions need RAM.

---

## Demand Paging

**Demand paging** is the mechanism behind virtual memory:  
Pages are only loaded into RAM **on demand** — when they are first accessed.

### The Process

```
1. Process starts → NO pages loaded into RAM
2. CPU tries to access a page
3. Page NOT in RAM → PAGE FAULT triggered (hardware exception)
4. OS handles the page fault → loads the page from disk to RAM
5. CPU retries the instruction → success
```

### Valid-Invalid Bit

Each page table entry has a **valid bit**:

```
Page Table:
┌────────┬───────────┬──────────────┐
│  Page  │ Valid Bit │ Frame/Disk   │
├────────┼───────────┼──────────────┤
│   0    │     1     │  Frame 3     │ ← in RAM
│   1    │     0     │  Disk block 7│ ← NOT in RAM
│   2    │     1     │  Frame 5     │ ← in RAM
│   3    │     0     │  Disk block 2│ ← NOT in RAM
└────────┴───────────┴──────────────┘
```

Valid = 1 → in RAM, access directly  
Valid = 0 → page fault → load from disk

---

## Benefits of Demand Paging

| Benefit | Explanation |
|---------|-------------|
| **Lazy loading** | Only load what's actually needed |
| **Fast startup** | Process starts executing immediately (no need to load everything first) |
| **Efficient RAM use** | Unaccessed pages never waste RAM |
| **More processes** | More processes fit in RAM simultaneously |

---

## Key Terms Summary

```
Virtual Address Space  → What the process THINKS it has (large, fake)
Physical Address Space → What actually exists in RAM (limited, real)
Disk (Swap Space)      → Extension of RAM (slow)

MMU (Memory Management Unit) → Hardware that translates virtual → physical
Page Table             → OS data structure for mapping
Valid Bit              → 1 = in RAM, 0 = on disk (causes page fault)
```

---

## 🎯 Interview Questions & Answers

**Q: What is virtual memory?**
> Virtual memory is a technique where the OS creates an illusion for each process that it has a large, private address space, even if the physical RAM is smaller. Pages not currently in RAM are stored on disk (swap space) and loaded on demand when accessed.

**Q: What is demand paging?**
> Demand paging loads pages from disk into RAM only when they are accessed (on demand). When a process starts, no pages are loaded. As it executes, page faults trigger loading of needed pages. This is lazy loading — only used pages consume RAM.

**Q: What is the difference between virtual memory and physical memory?**
> Physical memory is the actual RAM installed in the system. Virtual memory is the OS abstraction that gives each process its own large address space — backed partially by RAM and partially by disk swap space. The MMU translates virtual addresses to physical ones.

**Q: What happens when a process accesses a virtual address not in RAM?**
> The MMU detects the valid bit is 0 in the page table and raises a page fault exception. The OS page fault handler loads the required page from disk into a free RAM frame, updates the page table, and retries the instruction. See [05-Page-Fault.md](./05-Page-Fault.md).

**Q: Why does virtual memory allow more processes to run simultaneously?**
> Each process only needs its actively used pages in RAM. Unused pages stay on disk. So many processes can be "in memory" (in terms of virtual address space) while sharing a small amount of actual RAM — their inactive pages wait on disk until needed.

---

*← [Segmentation](./03-Segmentation.md) | Next → [Page Fault](./05-Page-Fault.md)*
