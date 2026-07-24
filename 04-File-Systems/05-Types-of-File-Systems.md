#  Types of File Systems

---

## Overview

Different OSes and use cases use different file systems. For interviews, know the **3 key ones**: FAT, NTFS, ext.

| File System | Used By | Type |
|-------------|---------|------|
| **FAT32** | USB drives, memory cards, old Windows | Linked allocation |
| **NTFS** | Modern Windows (Vista onward) | B+ tree + indexed |
| **ext4** | Modern Linux | Inode-based + journaling |

---

## 1.  FAT — File Allocation Table

### Variants

| Variant | Max File Size | Max Volume Size | Used For |
|---------|--------------|-----------------|---------|
| FAT12 | 32MB | 32MB | Old floppy disks |
| FAT16 | 2GB | 2GB | Old DOS/Windows |
| **FAT32** | **4GB** | **2TB** | USB drives, SD cards |
| exFAT | 16EB | 128PB | Modern flash (replaces FAT32) |

### How FAT Works

```
File Allocation Table (stored near beginning of disk):
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 0  │ 1  │ 2  │ 3  │ 4  │ 5  │ 6  │ 7  │  ← cluster number
├────┼────┼────┼────┼────┼────┼────┼────┤
│RSVD│RSVD│EOF │  5 │FREE│  7 │FREE│EOF │  ← next cluster (or EOF/FREE)
└────┴────┴────┴────┴────┴────┴────┴────┘

File "A" starts at cluster 2:
  FAT[2] = EOF → only 1 cluster (small file)

File "B" starts at cluster 3:
  FAT[3] = 5 → FAT[5] = 7 → FAT[7] = EOF → 3 clusters
```

### FAT Characteristics

| Feature | Detail |
|---------|--------|
| Allocation | Linked (via FAT table) |
| Max file size | 4GB (FAT32 — 32-bit cluster number) |
| Journaling |  No (data loss risk on crash) |
| Permissions |  No (no access control) |
| Cross-platform |  Excellent (Windows, Mac, Linux all support it) |
| Use case | USB drives, SD cards, compatibility |

### FAT32 Limitation: 4GB Max File Size
```
FAT32 uses 32-bit cluster entries → max 2^32 clusters
But cluster entry has 4 bits reserved → only 28 bits used → 2^28 clusters
With 32KB cluster size → 2^28 × 32KB = 8TB volume, but max file = 4GB (2^32 bytes - 1)
→ Can't store a single file larger than 4GB! (e.g., Blu-ray ISO, large VM images)
```

---

## 2.  NTFS — New Technology File System

Windows' primary file system since Windows NT (1993).

### Key Features

| Feature | Detail |
|---------|--------|
| Allocation | B+ tree based (Master File Table) |
| Max file size | 16EB (theoretical) |
| Max volume | 256TB |
| Journaling |  Yes (metadata + optional data journaling) |
| Permissions |  Full ACL (Access Control Lists) |
| Encryption |  EFS (Encrypting File System) |
| Compression |  Per-file/folder compression |
| Hard links |  Yes |
| Symbolic links |  Yes |

### Master File Table (MFT)

NTFS's central structure (like inode table):

```
MFT: Every file/directory has an MFT entry (1KB each)
MFT entry contains:
  - File attributes (name, timestamps, permissions)
  - Data runs (extents = contiguous block ranges)
  - Small files: data stored DIRECTLY in MFT entry (resident attribute)
```

### Journaling in NTFS

```
Before modifying disk:
1. Write intended operation to JOURNAL (log file)
2. Perform actual disk modification
3. Mark journal entry as complete

On crash:
  OS reads journal → replays incomplete operations → consistent state
```

**Why journaling matters**: Without it, a crash mid-write can corrupt the file system (FAT's problem).

---

## 3.  ext — Extended File System (Linux)

### Variants

| Variant | Max File Size | Max Volume | Key Feature |
|---------|--------------|-----------|-------------|
| ext2 | 2TB | 32TB | Basic inode FS, no journaling |
| ext3 | 2TB | 32TB | Adds journaling to ext2 |
| **ext4** | **16TB** | **1EB** | Extents, larger sizes, fast |

### ext4 Key Features

| Feature | Detail |
|---------|--------|
| Allocation | Inode-based (indexed allocation) |
| Journaling |  Yes (3 modes: journal, ordered, writeback) |
| Extents | Replaces block pointers with extent trees (contiguous block ranges) |
| Max file size | 16TB |
| Delayed allocation | Writes gathered in RAM, then flushed (reduces fragmentation) |
| Permissions |  POSIX permissions + ACLs |

### ext4 Extents (Improvement over ext2/3)

Instead of storing individual block numbers (up to 4 per block pointer), ext4 uses **extents**:

```
Extent: (start_block, length) — describes a RANGE of contiguous blocks

ext2: [block 15, block 16, block 17, block 18, ...]  ← one pointer per block
ext4: [start=15, length=4]                           ← one extent = 4 blocks

For a 1GB file: ext2 needs 262,144 pointers; ext4 needs FAR fewer extents
```

---

## Side-by-Side Comparison

| Feature | FAT32 | NTFS | ext4 |
|---------|-------|------|------|
| **OS** | All (cross-platform) | Windows | Linux |
| **Max File Size** | 4GB | 16EB | 16TB |
| **Max Volume** | 2TB | 256TB | 1EB |
| **Journaling** |  No |  Yes |  Yes |
| **Permissions** |  No |  ACLs |  POSIX |
| **Encryption** |  No |  EFS |  (dm-crypt) |
| **Fragmentation** |  High |  Low |  Low |
| **Use Case** | USB drives | Windows system drives | Linux system drives |

---

##  Interview Questions & Answers

**Q: What is the difference between FAT32, NTFS, and ext4?**
> FAT32 is a simple linked-allocation file system supported by all OSes — no journaling, no permissions, max 4GB file size. Used for flash drives and compatibility. NTFS is Windows' primary FS with ACL permissions, journaling, encryption, and supports huge files. ext4 is Linux's primary FS with inode-based allocation, extents, journaling, and POSIX permissions.

**Q: Why can't FAT32 store files larger than 4GB?**
> FAT32 uses 32-bit fields for file size, but only 28 bits are actually used for cluster indexing. The file size field in the directory entry is 32 bits — max value is 2³² - 1 = 4,294,967,295 bytes ≈ 4GB. This is why large video files don't fit on FAT32 USB drives.

**Q: What is journaling in a file system?**
> Journaling records intended disk operations in a log (journal) before performing them. If the system crashes mid-operation, the OS replays the journal on next boot to restore consistency. Without journaling (like FAT32), a crash can corrupt the file system, requiring a full disk scan (fsck/chkdsk) to repair.

**Q: What is the difference between ext2, ext3, and ext4?**
> ext2: Original inode-based FS, no journaling — data loss risk on crash. ext3: Added journaling to ext2. ext4: Added extents (contiguous block ranges instead of individual block pointers), larger file/volume sizes, delayed allocation, and improved performance over ext3.

---

*← [Inodes](./04-Inodes.md) | Back to [Topic 4 Index](./README.md)*
