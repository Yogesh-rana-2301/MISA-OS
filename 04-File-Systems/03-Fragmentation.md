# 🧩 Fragmentation in File Systems

---

## Context: Fragmentation Revisited (File System Perspective)

You've seen fragmentation in memory management. File systems have the **same concepts** but applied to **disk blocks** instead of RAM.

---

## External Fragmentation (File Systems)

> **Free disk space is scattered in small non-contiguous pieces** — can't fit a large file even though total free space is enough.

### Where It Occurs
**Contiguous Allocation** — files must occupy consecutive blocks.

```
Disk State:
[File A][FREE 2 blocks][File B][FREE 3 blocks][File C][FREE 2 blocks]

Total free: 7 blocks
But no contiguous 5-block hole exists!
→ A new 5-block file CANNOT be stored. ← External Fragmentation
```

### Solution
- **Disk Compaction / Defragmentation**: Move files to consolidate free space.
  - Windows: "Defragment and Optimize Drives"
  - HDD: Beneficial (physical read head moves less)
  - SSD: Not needed (no mechanical parts, block access is uniform speed)

- **Switch to Linked or Indexed Allocation**: Blocks don't need to be contiguous.

---

## Internal Fragmentation (File Systems)

> **Last block of a file is not fully used** — the remaining space in that block is wasted.

### Where It Occurs
**Any allocation method with fixed block sizes** (which is all of them in practice).

```
File size: 4097 bytes
Block size: 4096 bytes (4KB)

Blocks needed: ⌈4097 / 4096⌉ = 2 blocks = 8192 bytes allocated
Wasted space:  8192 - 4097 = 4095 bytes ← Internal Fragmentation
```

```
Block 1: [████████████████] 4096 bytes — fully used
Block 2: [█░░░░░░░░░░░░░░░] 1 byte used, 4095 bytes wasted
```

### Solution
- Smaller block sizes → less waste per file, but more blocks to manage (more overhead)
- Larger block sizes → faster I/O (fewer blocks), but more wasted space

**Trade-off**: Choosing block size is a design decision balancing waste vs performance.

---

## File Fragmentation (Disk Fragmentation)

A third type specific to file systems:

> **A single file's blocks are scattered across the disk** (not contiguous), causing slow sequential reads due to seek time.

```
notes.txt blocks: 5, 23, 67, 89, 12 (scattered!)

HDD must seek physically across disk:
Block 5 → seek → Block 12 → seek → Block 23 → seek → ...
Each seek: ~10ms  → reading a fragmented 1MB file could take seconds!

SSD: no mechanical head → seek time is near zero → fragmentation doesn't matter!
```

### Solution
- **Defragmentation** (for HDDs) — reorganizes file blocks to be contiguous
- **Use file systems that minimize fragmentation** (ext4, NTFS both have strategies)

---

## Summary: All Three Types

| Type | What is Wasted | Where Occurs | Solution |
|------|---------------|-------------|---------|
| **External** | Free space (scattered holes) | Contiguous allocation | Compaction / Indexed allocation |
| **Internal** | Partial last block | Fixed block sizes (all FS) | Smaller block size (trade-off) |
| **File (disk)** | Seek time (blocks scattered) | Any FS over time | Defragmentation (HDD only) |

---

## Free Space Management

To track which blocks are free, the OS uses:

### Bit Vector (Bitmap)
```
One bit per block: 0 = free, 1 = used

Bitmap: 1 1 0 0 1 1 0 1 0 0 ...
Block:  0 1 2 3 4 5 6 7 8 9 ...

Free blocks: 2, 3, 6, 8, 9 ...
```
- Simple, fast to find free blocks
- Must be kept in RAM for performance (can be large)

### Free List (Linked List)
```
Free block 2 → Free block 3 → Free block 6 → NULL
```
- Memory efficient (only stores free blocks)
- Slow to find N contiguous free blocks

---

## 🎯 Interview Questions & Answers

**Q: What is file system fragmentation?**
> There are three types: External fragmentation (free disk space scattered in non-contiguous holes — only in contiguous allocation), internal fragmentation (last block partially used — wasted space in fixed-size blocks), and file/disk fragmentation (one file's blocks scattered across disk — causes slow reads on HDD).

**Q: Why does defragmentation help HDDs but not SSDs?**
> HDDs have a physical read head that must mechanically seek to different disk locations. Scattered file blocks require many seeks (~10ms each), making fragmented files slow to read. SSDs have no mechanical parts — all blocks are accessed in ~100μs regardless of location. Defragmenting an SSD wastes write cycles without improving performance.

**Q: How does the OS track free disk space?**
> Commonly with a bitmap/bit vector — one bit per disk block, 0=free, 1=used. The OS scans for 0 bits to find free blocks. Alternatively, a linked free list chains all free blocks together.

**Q: How can internal fragmentation be reduced?**
> By using smaller block sizes — each file wastes at most (block_size - 1) bytes in its last block. However, smaller blocks mean more blocks per file, larger directory entries, and more overhead. Modern file systems use 4KB blocks as a balanced trade-off.

---

*← [File Allocation Methods](./02-File-Allocation-Methods.md) | Next → [Inodes](./04-Inodes.md)*
