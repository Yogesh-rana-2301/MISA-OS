#  Memory Allocation Techniques

---

## Why Memory Allocation?

When a process is created, the OS must find a **contiguous block of free RAM** to assign it.  
The OS maintains a **free list** — a list of free memory holes.

```
Physical RAM:
┌────────┬──────────┬──────────┬────────┬──────────┐
│OS (10K)│ FREE(30K)│ P1 (20K) │FREE(5K)│ FREE(15K)│
└────────┴──────────┴──────────┴────────┴──────────┘
           ↑                    ↑         ↑
        Hole 1               Hole 2    Hole 3
```

When a new process of size X arrives, which hole do we pick?

---

## Three Allocation Strategies

### 1.  First Fit
> Allocate the **first hole that is big enough**.

- Scan from the beginning of the list
- Stop at the first hole ≥ required size

```
Free holes: [30K, 5K, 15K]   Process needs: 12K

First Fit picks: 30K hole (first one ≥ 12K)
Remaining hole: 18K
```

** Pros**: Fast — minimal scanning  
** Cons**: Leaves many small unusable fragments at the start of memory

---

### 2.  Best Fit
> Allocate the **smallest hole that is big enough**.

- Scan entire list
- Pick the hole with minimum wasted space

```
Free holes: [30K, 15K, 13K]   Process needs: 12K

Best Fit picks: 13K hole (closest match)
Remaining hole: 1K
```

** Pros**: Minimizes wasted space per allocation  
** Cons**: Creates tiny leftover fragments that are often unusable; slower (full scan)

---

### 3.  Worst Fit
> Allocate the **largest available hole**.

- Scan entire list
- Pick the biggest hole

```
Free holes: [30K, 15K, 13K]   Process needs: 12K

Worst Fit picks: 30K hole
Remaining hole: 18K (still usefully large!)
```

** Pros**: Leftover fragment is large enough to be useful for future processes  
** Cons**: Wastes large holes; slowest in practice

---

### 4. Next Fit
> Like First Fit, but starts scanning from **where the last allocation was made** instead of the beginning.

- Maintains a "roving pointer" to the last allocated position
- Distributes allocations more evenly across memory
- Avoids the clustering of small fragments at the start that First Fit causes

```
Free holes: [30K at pos 0, 5K at pos 50, 15K at pos 80]
Last allocation was at pos 50.

Next Fit scans FROM pos 50:
  5K hole — too small for 12K request
  15K hole at pos 80 — fits! Picks this one.
  Remaining hole: 3K at pos 80
```

** Pros**: Avoids re-scanning from beginning; more even memory utilization  
** Cons**: Can miss better-fit holes earlier in the list; similar fragmentation to First Fit

---

## Comparison Table

| Strategy | Speed | Fragment Size | Best When |
|----------|-------|--------------|-----------|
| **First Fit** |  Fast | Medium | General purpose |
| **Best Fit** |  Slow | Tiny (wastes!) | When sizes are predictable |
| **Worst Fit** |  Slow | Large (reusable) | When future requests are large |
| **Next Fit** | Fast | Medium | Avoiding clustering at start |

> **Interview answer**: First Fit is generally considered the best in practice — fast and good average performance. Next Fit distributes fragmentation more evenly.

---

## Fragmentation

**Fragmentation** = wasted memory.  
There are two kinds — know both clearly.

---

###  External Fragmentation

> **Total free memory is enough, but it's scattered in non-contiguous pieces** that can't satisfy a large request.

```
RAM state:
[P1: 10K][FREE: 5K][P2: 20K][FREE: 5K][P3: 10K][FREE: 5K]

Total free: 15K
But no contiguous 15K block exists!
→ A process needing 12K CANNOT be loaded. ← External Fragmentation
```

**Caused by**: Variable-size allocation (Contiguous Allocation, Segmentation)  
**Solution**: **Compaction** (defragmentation — move all processes together) or **Paging**

---

###  Internal Fragmentation

> **Allocated memory is larger than what the process actually needs** — the extra space inside the allocated block is wasted.

```
Process needs: 18K
OS allocates:  20K  (nearest block size)
Wasted inside: 2K  ← Internal Fragmentation
```

**Caused by**: Fixed-size blocks (Paging — if page size = 4KB, a process needing 4097 bytes gets 2 pages = 8KB → 4095 bytes wasted)  
**Solution**: Smaller page/block sizes (trade-off with page table overhead)

---

## External vs Internal Fragmentation

| | External Fragmentation | Internal Fragmentation |
|-|----------------------|----------------------|
| **Location** | OUTSIDE allocated blocks | INSIDE allocated blocks |
| **Cause** | Non-contiguous free holes | Fixed-size allocation |
| **Seen in** | Contiguous allocation, Segmentation | Paging, Fixed partitioning |
| **Solution** | Compaction or Paging | Smaller page sizes |
| **Wastes** | Memory between processes | Memory within a process's allocation |

---

##  Interview Questions & Answers

**Q: What is the difference between First Fit, Best Fit, and Worst Fit?**
> First Fit allocates the first hole large enough (fast). Best Fit allocates the smallest sufficient hole (minimizes waste per allocation but creates tiny unusable fragments). Worst Fit allocates the largest hole (leftover is large and reusable). First Fit performs best in practice.

**Q: What is external fragmentation?**
> External fragmentation is when enough total free memory exists but it's scattered in small non-contiguous holes — no single hole is large enough for a new process. Solved by compaction or paging.

**Q: What is internal fragmentation?**
> Internal fragmentation is wasted memory inside an allocated block — the OS allocates more than the process needs (e.g., due to fixed page sizes). Common in paging systems.

**Q: How does paging solve external fragmentation?**
> Paging divides memory into fixed-size frames. A process's logical address space is divided into same-size pages. Pages can be mapped to any free frames — they don't need to be contiguous in physical memory. This eliminates external fragmentation entirely.

---

*Next → [Paging](./02-Paging.md)*
