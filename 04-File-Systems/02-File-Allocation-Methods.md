# 💾 File Allocation Methods

---

## The Problem

A file's data is split across multiple **disk blocks**.  
**How does the OS track which blocks belong to which file?**

Three approaches:

```
File Allocation Methods:
├── 1. Contiguous Allocation
├── 2. Linked Allocation
└── 3. Indexed Allocation
```

---

## 1. 📦 Contiguous Allocation

> All blocks of a file are stored **consecutively** on disk.

```
File "notes.txt" starts at block 10, size = 4 blocks:
Block:  10   11   12   13   14   15   16   17   18
Data:  [A1] [A2] [A3] [A4] [free][B1] [B2] [free][free]
                              ↑
                    Gap — can't be used by A
```

**Directory entry** stores: `{start_block, length}`

```
notes.txt → start=10, length=4
→ blocks 10, 11, 12, 13 belong to notes.txt
```

### Advantages ✅
- **Fast sequential access** — blocks are adjacent
- **Fast random access** — block n = start + n (direct calculation)
- Simple directory entry (just start + length)

### Disadvantages ❌
- **External fragmentation** — free holes scattered, large files can't fit
- **File size must be known at creation** — hard to grow a file
- **Inflexible** — moving or extending a file requires copying

---

## 2. 🔗 Linked Allocation

> Each block contains a **pointer to the next block**.  
> Blocks of a file can be scattered anywhere on disk.

```
notes.txt → starts at block 9

Block 9  → [data][→ Block 16]
Block 16 → [data][→ Block 1]
Block 1  → [data][→ Block 10]
Block 10 → [data][→ NULL]  ← end of file
```

**Directory entry** stores only: `{start_block}`  
Each block uses part of its space to store the **next-block pointer** (e.g., 4 bytes per 512-byte block).

### Advantages ✅
- **No external fragmentation** — any free block can be used
- **Files can grow dynamically** — just append a new block anywhere
- No need to know file size upfront

### Disadvantages ❌
- **No random access** — to read block 100, must follow 99 pointers (slow!)
- **Pointer overhead** — each block loses a few bytes to the pointer
- **Reliability** — one broken pointer loses the rest of the file

### FAT (File Allocation Table) — Linked Allocation Variant

Instead of storing next-block pointer inside each block, FAT keeps all pointers in a **separate table in memory**:

```
FAT Table (in RAM):
Index:  0   1   2   3   4   5   6   7   8   9  10  11
Value: [/] [10][EOF][/] [/] [/] [/] [/] [/] [16][NULL][1]

notes.txt starts at block 9:
  FAT[9]  = 16  → next block is 16
  FAT[16] = 1   → next block is 1
  FAT[1]  = 10  → next block is 10
  FAT[10] = EOF → end of file
```

FAT is loaded entirely in RAM → fast traversal, but RAM-hungry for large disks.

---

## 3. 📇 Indexed Allocation

> One special **index block** per file stores **all block pointers** for that file.

```
notes.txt's index block (block 4):
┌───────────────────────────────────┐
│ Block pointer 0 → Block 15        │
│ Block pointer 1 → Block 23        │
│ Block pointer 2 → Block 8         │
│ Block pointer 3 → Block 40        │
│ ...                               │
└───────────────────────────────────┘
```

**Directory entry** stores: `{index_block_number}`

### Advantages ✅
- **Supports random access** — index_block[n] gives block directly
- **No external fragmentation** — blocks scattered anywhere
- **Files can grow** — add more pointers to index block

### Disadvantages ❌
- **Index block overhead** — even tiny files waste an entire index block
- **Limited file size** — one index block can only hold so many pointers

### Handling Large Files — Multi-Level Indexing (UNIX inode style)

For large files, a single index block isn't enough. Unix uses:

```
Inode:
┌─────────────────────────────────────────────────┐
│ Direct pointers (12)   → point to data blocks   │ (12 × 4KB = 48KB)
│ Single-indirect ptr    → points to a block of   │
│                          pointers (1024 ptrs)    │ (1024 × 4KB = 4MB)
│ Double-indirect ptr    → points to block of      │
│                          single-indirect blocks  │ (1024² × 4KB = 4GB)
│ Triple-indirect ptr    → 3 levels of indirection │ (huge!)
└─────────────────────────────────────────────────┘
```

This is exactly how **Linux ext file systems** work (see [04-Inodes.md](./04-Inodes.md)).

---

## Comparison Table

| Feature | Contiguous | Linked | Indexed |
|---------|-----------|--------|---------|
| **Sequential access** | ✅ Fast | ✅ OK | ✅ OK |
| **Random access** | ✅ Fast | ❌ Slow (follow chain) | ✅ Fast |
| **External fragmentation** | ✅ Yes | ❌ No | ❌ No |
| **File growth** | ❌ Hard | ✅ Easy | ✅ Easy |
| **Overhead** | None | Pointer per block | Index block |
| **Used in** | CD-ROMs, DVDs | FAT (MS-DOS) | UNIX/Linux ext |

---

## 🎯 Interview Questions & Answers

**Q: Compare contiguous, linked, and indexed file allocation.**
> **Contiguous**: Files in consecutive blocks — fast access (sequential and random), but causes external fragmentation and can't grow easily. **Linked**: Each block points to the next — no fragmentation, files grow easily, but random access is O(n) slow. **Indexed**: One index block holds all pointers — no fragmentation, fast random access, but small files waste an index block. Used in Linux (inodes).

**Q: How does random access work in each method?**
> Contiguous: block_n = start + n (O(1)). Linked: must traverse n pointers from the start (O(n)). Indexed: read index_block[n] (O(1) pointer lookup).

**Q: What is FAT and which allocation method does it use?**
> FAT (File Allocation Table) uses a variant of linked allocation. Instead of storing next-block pointers inside each block, a separate FAT table in memory holds all next-block pointers. This allows faster traversal since the FAT is in RAM, but the FAT itself can be large for big disks.

**Q: What is multi-level indexing? Why is it needed?**
> Multi-level indexing uses a hierarchy of index blocks: direct pointers, single-indirect, double-indirect, triple-indirect. This is needed because a single index block can only hold a limited number of pointers (e.g., 1024 on a 4KB block with 4-byte pointers = max 4MB file). Multi-level allows files to grow to gigabytes or terabytes. Used in UNIX inode structure.

---

*← [File System Basics](./01-File-System-Basics.md) | Next → [Fragmentation](./03-Fragmentation.md)*
