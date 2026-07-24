#  Paging

---

## The Core Problem Paging Solves

Contiguous allocation requires a process to fit in one block of RAM.  
**Paging removes this requirement** — a process can be split across non-contiguous physical memory.

---

## Key Concepts

| Term | Definition |
|------|-----------|
| **Page** | Fixed-size block of **logical** (virtual) address space |
| **Frame** | Fixed-size block of **physical** memory (RAM) |
| **Page Size** | Always a power of 2 (e.g., 4KB, 8KB) |
| **Page Table** | OS data structure mapping pages → frames |

> **Rule**: Page size = Frame size (always)

```
Logical Address Space (Process view):      Physical Memory (RAM):
┌──────┐                                   ┌──────┐ Frame 0
│Page 0│ ──────────────────────────────────▶│      │
├──────┤                                   ├──────┤ Frame 1
│Page 1│ ──────────┐                       │ OS   │
├──────┤           │                       ├──────┤ Frame 2
│Page 2│ ────────┐ │                       │      │◀──── Page 1
├──────┤         │ │                       ├──────┤ Frame 3
│Page 3│ ──────┐ │ └──────────────────────▶│      │ Frame 4 ◀── Page 0
└──────┘       │ └────────────────────────▶│      │ Frame 5 ◀── Page 2
               └──────────────────────────▶│      │ Frame 6 ◀── Page 3
                                           └──────┘
Pages need NOT be in consecutive frames!
```

---

## Address Translation

The CPU generates **logical addresses**. The OS/hardware converts them to **physical addresses**.

### Logical Address Structure

```
Logical Address = [ Page Number (p) | Page Offset (d) ]
                   ←─── upper bits ───▶←─ lower bits ─▶
```

- **Page Number (p)**: Index into the page table → gives frame number
- **Offset (d)**: Byte position within the page/frame (same in both logical and physical)

### Formula

```
Logical Address  → Split into (p, d)
Physical Address = Frame_Base_Address + d
                 = (page_table[p] × page_size) + d
```

### Example

```
Page Size = 4KB = 4096 bytes = 2¹² bytes
→ Offset bits = 12
→ Page number bits = remaining bits (say 32-bit address → 20 bits for page number)

Logical address: 0x00003ABC
  Page size = 4KB = 0x1000
  Page number = 0x3ABC / 0x1000 = 3
  Offset      = 0x3ABC % 0x1000 = 0xABC

Page table[3] = Frame 7

Physical address = Frame 7 × 4KB + 0xABC
                 = 0x7000 + 0xABC
                 = 0x7ABC
```

---

## Numeric Example (Small)

**Setup**: Page size = 100 bytes, 4 pages, 8 frames

| Page | Frame |
|------|-------|
| 0 | 5 |
| 1 | 3 |
| 2 | 7 |
| 3 | 2 |

**Translate logical address 350:**
```
Page number = 350 / 100 = 3 (integer division)
Offset      = 350 % 100 = 50

Page 3 → Frame 2

Physical address = 2 × 100 + 50 = 250
```

---

## Page Table

The **page table** is stored in main memory (RAM) and maintained by the OS.

Each **page table entry (PTE)** contains:

```
┌────────────────────────────────────────────────────────┐
│ Frame Number │ Valid bit │ Dirty bit │ Access bits │... │
└────────────────────────────────────────────────────────┘
```

| Bit | Meaning |
|-----|---------|
| **Valid bit** | 1 = page is in memory, 0 = page not in RAM (page fault!) |
| **Dirty bit** | 1 = page modified since loaded (must write to disk when evicted) |
| **Reference bit** | 1 = page accessed recently (used by LRU approximation) |

---

## The Double Memory Access Problem

Without optimization, paging doubles memory accesses:
1. Access page table in RAM → get frame number
2. Access actual data in RAM → using physical address

**Solution**: TLB (Translation Lookaside Buffer) — see [08-Cache-and-TLB.md](./08-Cache-and-TLB.md)

---

## Advantages & Disadvantages of Paging

|  Advantages |  Disadvantages |
|--------------|----------------|
| Eliminates external fragmentation | Internal fragmentation (last page may be partially used) |
| Non-contiguous allocation | Page table memory overhead |
| Easy to implement protection (per-page bits) | Double memory access (solved by TLB) |
| Enables virtual memory | Large page tables for large address spaces |

---

## Paging vs Segmentation Quick Ref

| | Paging | Segmentation |
|-|--------|-------------|
| Division basis | Fixed size | Logical meaning (code, stack, heap) |
| External fragmentation |  No |  Yes |
| Internal fragmentation |  Yes (last page) |  No |
| User visibility | Transparent | Visible (programmer-defined) |

*(Full segmentation → see [03-Segmentation.md](./03-Segmentation.md))*

---

##  Interview Questions & Answers

**Q: What is paging?**
> Paging divides a process's logical address space into fixed-size pages and physical memory into same-size frames. Pages can be mapped to any free frame — they don't need to be contiguous. The OS maintains a page table mapping page numbers to frame numbers.

**Q: How is a logical address translated to a physical address in paging?**
> Split the logical address into page number (p) and offset (d). Look up p in the page table to get frame number f. Physical address = f × page_size + d. In hardware, this is done by the MMU.

**Q: Why does paging eliminate external fragmentation?**
> Because pages can be placed in any free frame regardless of location in memory. There's no need for a contiguous block. Any collection of free frames can be used for a process.

**Q: What is a page table entry? What bits does it contain?**
> Each entry contains the frame number plus control bits: valid bit (is the page in RAM?), dirty bit (was it modified?), and reference bit (was it recently accessed?). These bits are used for memory protection and page replacement.

---

*← [Memory Allocation](./01-Memory-Allocation.md) | Next → [Segmentation](./03-Segmentation.md)*
