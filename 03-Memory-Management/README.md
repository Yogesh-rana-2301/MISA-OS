# 🧠 Topic 3 — Memory Management

> **Goal**: Understand how the OS manages RAM — one of the most heavily tested areas in OS interviews.

---

## 📚 Subtopics

| # | File | Topic |
|---|------|-------|
| 1 | [01-Memory-Allocation.md](./01-Memory-Allocation.md) | Memory Allocation (First/Best/Worst Fit + Fragmentation) |
| 2 | [02-Paging.md](./02-Paging.md) | Paging — Pages, Frames, Address Translation |
| 3 | [03-Segmentation.md](./03-Segmentation.md) | Segmentation vs Paging |
| 4 | [04-Virtual-Memory.md](./04-Virtual-Memory.md) | Virtual Memory & Demand Paging |
| 5 | [05-Page-Fault.md](./05-Page-Fault.md) | Page Fault — Trigger & Handling |
| 6 | [06-Page-Replacement.md](./06-Page-Replacement.md) | Page Replacement Algorithms + Belady's Anomaly |
| 7 | [07-Thrashing.md](./07-Thrashing.md) | Thrashing & Working Set |
| 8 | [08-Cache-and-TLB.md](./08-Cache-and-TLB.md) | Cache Memory, Locality, Mapping, TLB |

---

## 🔥 Quick Revision Checklist

- [ ] First Fit vs Best Fit vs Worst Fit — when to use which?
- [ ] Internal vs External fragmentation — with examples
- [ ] How does paging eliminate external fragmentation?
- [ ] Logical address → Physical address translation (with formula)
- [ ] What is virtual memory? Why is it needed?
- [ ] Page fault handling — step by step
- [ ] FIFO, LRU, Optimal page replacement — worked examples
- [ ] What is Belady's Anomaly? Which algorithm is immune?
- [ ] What is thrashing? How does the working set model prevent it?
- [ ] What is the TLB? How does it speed up address translation?
- [ ] Temporal vs Spatial locality

---

## 🎯 Top Interview Questions from This Topic

1. What is the difference between internal and external fragmentation?
2. How does paging work? How is a logical address translated to a physical address?
3. What is virtual memory? How does demand paging work?
4. What happens during a page fault?
5. Compare FIFO, LRU, and Optimal page replacement algorithms.
6. What is Belady's Anomaly?
7. What is thrashing and how do you prevent it?
8. What is a TLB and why is it needed?
