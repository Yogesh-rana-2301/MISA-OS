#  Segmentation

---

## What is Segmentation?

Segmentation divides a process's memory into **logically meaningful, variable-size segments** — as the programmer sees the program.

> **Paging** divides memory into fixed-size pieces (invisible to programmer).  
> **Segmentation** divides memory into logical units that match program structure.

---

## Logical Division of a Program

A typical program has several natural logical sections:

```
Process Memory (Programmer's View):
┌─────────────────┐  ← Segment 0: Code (Text)
│  int main() {}  │
│  void foo() {}  │
├─────────────────┤  ← Segment 1: Data (Global/Static vars)
│  int x = 5;     │
│  char buf[100]; │
├─────────────────┤  ← Segment 2: Heap (dynamic malloc)
│  (grows down)   │
├─────────────────┤  ← Segment 3: Stack (local vars, returns)
│  (grows up)     │
└─────────────────┘  ← Segment 4: Shared libraries
```

Each segment has a **name/number**, a **base address**, and a **limit (size)**.

---

## Segment Table

The OS maintains a **segment table** (one per process):

```
Segment Table:
┌─────────┬──────────────┬────────┐
│ Segment │ Base Address │ Limit  │
├─────────┼──────────────┼────────┤
│    0    │   0x4000     │  500   │  ← Code
│    1    │   0x7000     │  200   │  ← Data
│    2    │   0x9000     │  600   │  ← Heap
│    3    │   0xC000     │  300   │  ← Stack
└─────────┴──────────────┴────────┘
```

---

## Address Translation in Segmentation

```
Logical Address = [ Segment Number (s) | Offset (d) ]

Physical Address = segment_table[s].base + d

Protection check: if d >= segment_table[s].limit → SEGFAULT!
```

### Example

```
Segment table[1]: base = 0x7000, limit = 200

Logical address (1, 53):
  segment = 1, offset = 53
  Check: 53 < 200 
  Physical = 0x7000 + 53 = 0x7035
```

---

## Segmentation vs Paging

| Feature | Paging | Segmentation |
|---------|--------|-------------|
| **Division** | Fixed-size pages | Variable-size logical segments |
| **User visibility** | Transparent | Programmer-defined |
| **External fragmentation** |  None |  Yes (variable sizes cause holes) |
| **Internal fragmentation** |  Yes (last page) |  None |
| **Protection** | Per-page bits | Per-segment (code=read-only, stack=RW) |
| **Sharing** | Hard (fixed boundaries) | Easy (share code segment) |
| **Address format** | (page#, offset) | (segment#, offset) |

---

## Why Segmentation is Better for Protection & Sharing

```
Code segment → read + execute only (no write) → prevents code modification
Stack segment → read + write (no execute) → prevents stack execution attacks
Heap segment → read + write (no execute)

Sharing: Two processes can map their Code segment to the same physical memory
         → saves RAM (e.g., shared libraries like libc)
```

---

## Segmented Paging (Best of Both)

Most modern OSes use **segmented paging** (or just paging):
- Logical address space divided into segments (programmer's view)
- Each segment further divided into pages
- Eliminates both external fragmentation (paging) and maintains logical structure (segmentation)

```
Logical Address → [Segment | Page | Offset]
                     ↓         ↓
               Segment Table  Page Table  → Physical Frame
```

---

##  Interview Questions & Answers

**Q: What is segmentation?**
> Segmentation divides a process's memory into variable-size, logically meaningful units — code, data, heap, stack. Each segment has a base address and a limit. The logical address is (segment number, offset); the OS translates via a segment table.

**Q: What is the difference between paging and segmentation?**
> Paging uses fixed-size units (invisible to programmer), eliminates external fragmentation but causes internal fragmentation. Segmentation uses variable-size logical units visible to the programmer, eliminates internal fragmentation but causes external fragmentation. Segmentation is better for sharing and protection; paging is simpler to manage.

**Q: What causes a segmentation fault (SIGSEGV)?**
> A segfault occurs when a process accesses a memory address outside its valid segment limits, or violates access permissions (e.g., writing to a read-only code segment). The hardware detects the violation via the segment table's limit check, and the OS sends SIGSEGV to the process.

**Q: How does segmentation help with code sharing between processes?**
> Two processes can have their code segment (read-only) mapped to the same physical memory. Only one copy of the code exists in RAM, saving memory. This is how shared libraries (like libc.so) work.

---

*← [Paging](./02-Paging.md) | Next → [Virtual Memory](./04-Virtual-Memory.md)*
