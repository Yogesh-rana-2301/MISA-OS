# ⚡ Cache Memory, Locality & TLB

---

## Cache Memory

### The Memory Hierarchy Problem

Accessing RAM takes ~100ns. The CPU can execute instructions in ~0.3ns.  
**The CPU starves waiting for data from RAM.**

**Solution**: Place small, fast **cache memory** between CPU and RAM.

```
Speed vs Size Trade-off:

Registers   │ 1ns    │ ~KB    │ Inside CPU
L1 Cache    │ 2-4ns  │ 32KB   │ Inside CPU core
L2 Cache    │ 10ns   │ 256KB  │ Inside CPU (shared)
L3 Cache    │ 30ns   │ 8-32MB │ On CPU chip
RAM (DRAM)  │ 100ns  │ GBs    │ On motherboard
SSD         │ 100μs  │ TBs    │ Storage
HDD         │ 10ms   │ TBs    │ Storage
```

### Cache Hit vs Cache Miss

| Event | Meaning | Action |
|-------|---------|--------|
| **Cache Hit** | Data found in cache | Return it immediately (fast!) |
| **Cache Miss** | Data not in cache | Fetch from RAM (slow), store in cache |

```
Hit Rate = Hits / (Hits + Misses)
Effective Access Time = (hit_rate × cache_time) + (miss_rate × RAM_time)

Example: hit_rate=0.95, cache=5ns, RAM=100ns
EAT = 0.95 × 5ns + 0.05 × 100ns = 4.75 + 5 = 9.75ns ← much better than 100ns!
```

---

## Locality of Reference

The reason caches work so well — programs naturally exhibit **locality**:

### 1. Temporal Locality
> **A recently accessed memory location is likely to be accessed again soon.**

```
for (int i = 0; i < 1000; i++) {
    counter++;        ← 'counter' accessed 1000 times repeatedly
}
```

→ Keep `counter` in cache after first access.

### 2. Spatial Locality
> **If a memory location is accessed, nearby locations are likely to be accessed soon.**

```
int arr[1000];
for (int i = 0; i < 1000; i++) {
    sum += arr[i];   ← arr[0], arr[1], arr[2]... accessed sequentially
}
```

→ Cache loads a whole **cache line** (64 bytes) at once, anticipating nearby accesses.

---

## Cache Mapping Techniques

How does the cache decide where to store a block of RAM?

### 1. Direct Mapping
> Each RAM block maps to exactly **one specific cache line**.

```
Cache line = (RAM block number) % (number of cache lines)

RAM block 0  → Cache line 0
RAM block 4  → Cache line 0  (conflict! must evict)
RAM block 8  → Cache line 0  (conflict again!)
```

| ✅ Pros | ❌ Cons |
|--------|--------|
| Simple, fast lookup | Conflict misses (two frequently used blocks fight for same line) |

---

### 2. Fully Associative Mapping
> A RAM block can go into **any** cache line.

```
RAM block can be placed in cache line 0, 1, 2, ... (any free slot)
```

| ✅ Pros | ❌ Cons |
|--------|--------|
| No conflict misses | Slow lookup (must search all lines) |

---

### 3. Set-Associative Mapping (Best of Both)
> Cache is divided into **sets**, each with N lines (**N-way set associative**).  
> A RAM block maps to a specific **set** but can go in any of the **N lines** in that set.

```
4-way set associative:
  Each set has 4 cache lines
  RAM block → maps to 1 set → can go in any of 4 lines
```

| ✅ Pros | ❌ Cons |
|--------|--------|
| Fewer conflicts than direct | More complex than direct |
| Faster than fully associative | Slightly slower than direct |
| Used in modern CPUs (L1: 8-way, L2: 16-way) | |

---

## TLB — Translation Lookaside Buffer ⭐ IMPORTANT

### The Problem with Paging

Every memory access in paging requires **two RAM accesses**:
1. Access the **page table** in RAM → get frame number
2. Access the **actual data** in RAM

This doubles memory access time — unacceptable!

### What is TLB?

The **TLB** is a **small, fast hardware cache** for recent page table entries.

```
TLB: (page number → frame number) mappings
Size: 64–1024 entries (very fast SRAM)
Location: Inside MMU (hardware)
```

### TLB Address Translation Flow

```
CPU generates logical address (page p, offset d)
         │
         ▼
┌─────────────────────────────────────────────┐
│  TLB Lookup: Is page p in TLB?              │
├─────────────────────────────────────────────┤
│  TLB HIT (fast path ~1ns extra):            │
│    frame = TLB[p]                           │
│    physical = frame × page_size + d         │
│    → Access memory directly ✅              │
├─────────────────────────────────────────────┤
│  TLB MISS (slow path):                      │
│    Look up page table in RAM → get frame    │
│    Store (p → frame) in TLB                 │
│    physical = frame × page_size + d         │
│    → Access memory ✅                       │
└─────────────────────────────────────────────┘
```

### EAT with TLB

```
EAT = h × (TLB_time + mem_time) + (1 − h) × (TLB_time + 2 × mem_time)

Where:
  h = TLB hit rate (typically 95–99%)
  TLB_time ≈ 1ns
  mem_time ≈ 100ns

h = 0.98 (98% hit rate):
EAT = 0.98 × (1 + 100) + 0.02 × (1 + 200)
    = 0.98 × 101 + 0.02 × 201
    = 98.98 + 4.02
    = 103ns  ← close to single memory access!
```

vs. without TLB: 200ns (double access)

### TLB Flush

When is the TLB invalidated?
- **Context switch** to another process (different page table) → flush TLB
- **Page table modification** (page swapped out)
- Modern CPUs use **ASIDs (Address Space IDs)** to tag entries → avoids full flush on context switch

---

## Cache vs TLB — Quick Comparison

| | Cache | TLB |
|-|-------|-----|
| **Stores** | Data/instructions from RAM | Page table entries (page→frame) |
| **Purpose** | Reduce RAM access latency | Speed up address translation |
| **On miss** | Fetch from RAM | Walk page table in RAM |
| **Size** | 32KB–32MB | 64–1024 entries |
| **Miss penalty** | ~100ns | One extra RAM access |

---

## 🎯 Interview Questions & Answers

**Q: What is a TLB and why is it needed?**
> The TLB (Translation Lookaside Buffer) is a small, fast hardware cache that stores recent page table entries (page number → frame number). Without TLB, paging requires two RAM accesses per memory operation (page table + actual data). TLB eliminates the first access for frequently used pages, cutting memory access time nearly in half.

**Q: What is temporal and spatial locality?**
> Temporal locality: a recently accessed memory location will likely be accessed again soon (e.g., loop variables). Spatial locality: if a location is accessed, nearby locations will likely be accessed soon (e.g., array traversal). These properties make caches effective.

**Q: What happens on a TLB miss?**
> The MMU must walk the page table in RAM to find the frame number (one extra RAM access). The (page → frame) mapping is then stored in the TLB for future accesses. If the page itself is not in RAM, a page fault occurs.

**Q: What is the difference between direct mapping and set-associative cache mapping?**
> Direct mapping assigns each RAM block to exactly one cache line — simple but causes conflict misses when two frequently used blocks compete for the same line. Set-associative (N-way) divides the cache into sets of N lines; a block maps to one set but can use any of the N lines — reduces conflicts. Modern CPUs use set-associative (8-way, 16-way).

**Q: Why is the TLB flushed on a context switch?**
> Different processes have different page tables (different virtual-to-physical mappings). Using another process's TLB entries would give wrong physical addresses. The TLB is flushed (or tagged with ASIDs) on each context switch to ensure correct translations.

---

*← [Thrashing](./07-Thrashing.md) | Back to [Topic 3 Index](./README.md)*
