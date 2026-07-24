#  Page Replacement Algorithms

---

## Why Page Replacement?

When a page fault occurs and **RAM is full**, the OS must **evict a page** to make room.

**Goal**: Choose which page to evict so that future page faults are minimized.

```
RAM is full (all frames occupied)
New page needed → must evict one existing page
Which to evict? → Page replacement algorithm decides
```

---

## Setup for All Examples

**Reference String** (sequence of page accesses): `7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2`  
**Number of Frames** = 3

---

## 1.  FIFO — First In First Out

> **Evict the page that has been in memory the longest.**

Think of frames as a queue — oldest page is at the front.

### Trace (Frames = 3)

| Access | Frame 1 | Frame 2 | Frame 3 | Fault? |
|--------|---------|---------|---------|--------|
| 7 | **7** | - | - |  YES |
| 0 | 7 | **0** | - |  YES |
| 1 | 7 | 0 | **1** |  YES |
| 2 | **2** | 0 | 1 |  YES (evict 7, oldest) |
| 0 | 2 | 0 | 1 |  NO |
| 3 | 2 | **3** | 1 |  YES (evict 0, oldest) |
| 0 | 2 | 3 | **0** |  YES (evict 1, oldest) |
| 4 | **4** | 3 | 0 |  YES (evict 2, oldest) |
| 2 | 4 | **2** | 0 |  YES (evict 3, oldest) |
| 3 | 4 | 2 | **3** |  YES (evict 0, oldest) |
| 0 | **0** | 2 | 3 |  YES (evict 4, oldest) |
| 3 | 0 | 2 | 3 |  NO |
| 2 | 0 | 2 | 3 |  NO |

**Total Page Faults = 9**

### Properties

| | Detail |
|-|--------|
| Simple |  Easy to implement (queue) |
| Optimal? |  No |
| Belady's Anomaly |  Suffers from it |

---

## 2.  LRU — Least Recently Used

> **Evict the page that was accessed least recently.**

Based on **temporal locality** — recently used pages will likely be used again soon.

### Trace (Frames = 3)

| Access | Frames | Fault? | LRU order (left=least recent) |
|--------|--------|--------|-------------------------------|
| 7 | {7} |  | [7] |
| 0 | {7,0} |  | [7,0] |
| 1 | {7,0,1} |  | [7,0,1] |
| 2 | {0,1,2} |  | evict 7 (LRU) → [0,1,2] |
| 0 | {0,1,2} |  | [1,2,0] (0 now most recent) |
| 3 | {2,0,3} |  | evict 1 (LRU) → [2,0,3] |
| 0 | {2,0,3} |  | [2,3,0] |
| 4 | {0,3,4} |  | evict 2 (LRU) |
| 2 | {3,4,2} |  | evict 0 (LRU) |
| 3 | {3,4,2} |  | [4,2,3] |
| 0 | {2,3,0} |  | evict 4 (LRU) |
| 3 | {2,3,0} |  | [2,0,3] |
| 2 | {2,3,0} |  | [0,3,2] |

**Total Page Faults = 8**

### Properties

| | Detail |
|-|--------|
| Based on | Past access history |
| Optimal? |  No, but good approximation |
| Implementation | Counters or Stack |
| Belady's Anomaly |  Does NOT suffer |
| Overhead | Higher (tracking access times) |

---

## 3. ⭐ Optimal (OPT / Belady's Optimal)

> **Evict the page that will NOT be used for the longest time in the future.**

This is the **theoretically best** algorithm. It minimizes page faults.  
**NOT implementable** in practice (requires future knowledge), but used as a benchmark.

### Trace (Frames = 3)

| Access | Frames | Fault? | Evict |
|--------|--------|--------|-------|
| 7 | {7} |  | — |
| 0 | {7,0} |  | — |
| 1 | {7,0,1} |  | — |
| 2 | {0,1,2} |  | Evict 7 (next use: never again) |
| 0 | {0,1,2} |  | |
| 3 | {0,2,3} |  | Evict 1 (next use: never again) |
| 0 | {0,2,3} |  | |
| 4 | {0,3,4} |  | Evict 2 (next use: farthest) |
| 2 | {0,2,3} |  | Evict 4 (next use: never) |
| 3 | {0,2,3} |  | |
| 0 | {0,2,3} |  | |
| 3 | {0,2,3} |  | |
| 2 | {0,2,3} |  | |

**Total Page Faults = 7** ← Minimum possible

### Properties

| | Detail |
|-|--------|
| Optimal? |  YES — minimum faults |
| Implementable? |  NO — needs future knowledge |
| Purpose | Benchmark to compare other algorithms |

---

## Comparison Table

| Algorithm | Page Faults (3 frames) | Belady's Anomaly | Implementable |
|-----------|----------------------|-----------------|--------------|
| FIFO | 9 |  Yes |  Yes |
| LRU | 8 |  No |  Yes (with overhead) |
| Optimal | 7 |  No |  No (theoretical) |

---

##  Belady's Anomaly

### What is it?

> **Adding more frames INCREASES page faults** (only for FIFO).

This seems counterintuitive — more RAM should mean fewer faults!  
But with FIFO, it can happen.

### Example (Reference string: 1,2,3,4,1,2,5,1,2,3,4,5)

```
3 Frames → 9 page faults
4 Frames → 10 page faults ← MORE faults with more RAM!
```

### Which algorithms are immune to Belady's Anomaly?

> **Stack algorithms** (LRU, Optimal) are immune.  
> A stack algorithm has the property that the set of pages in memory with N frames is always a **subset** of pages in memory with N+1 frames.  
> LRU and Optimal are stack algorithms. FIFO is NOT.

---

##  Interview Questions & Answers

**Q: Compare FIFO, LRU, and Optimal page replacement.**
> FIFO evicts the oldest page — simple but inefficient (9 faults in example). LRU evicts the least recently used page — good approximation of optimal (8 faults). Optimal evicts the page used farthest in the future — theoretically best (7 faults) but not implementable. LRU is the practical choice.

**Q: What is Belady's Anomaly?**
> Belady's Anomaly is when increasing the number of frames leads to MORE page faults, not fewer. It occurs only in FIFO. Stack algorithms (LRU, Optimal) are immune because their page sets grow monotonically with more frames.

**Q: Why is the Optimal algorithm not used in practice?**
> The Optimal algorithm requires knowing which pages will be accessed in the future, which is impossible in a real system. It's used as a theoretical benchmark to evaluate how close other algorithms are to optimal.

**Q: Why does LRU perform better than FIFO?**
> LRU uses temporal locality — recently accessed pages are likely to be used again soon, so it makes sense to keep them. FIFO ignores access patterns entirely and evicts based only on age, which doesn't correlate with future usage.

---

*← [Page Fault](./05-Page-Fault.md) | Next → [Thrashing](./07-Thrashing.md)*
