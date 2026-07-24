#  Inodes (Index Nodes)

---

## What is an Inode?

An **inode (index node)** is a data structure in Unix/Linux file systems that stores **all metadata about a file EXCEPT its name**.

> **Key insight**: In Unix, the **file name** is stored in the **directory**. The **inode** stores everything else.

```
Directory:
┌──────────────────┬───────────┐
│   File Name      │  Inode #  │
├──────────────────┼───────────┤
│   notes.txt      │   1234    │ ← name maps to inode number
│   photo.jpg      │   5678    │
└──────────────────┴───────────┘

Inode 1234:
┌─────────────────────────────────────────────┐
│ File type: regular file                     │
│ Permissions: rw-r--r--                      │
│ Owner UID: 1000                             │
│ Group GID: 1000                             │
│ File size: 4096 bytes                       │
│ Link count: 1                               │
│ Timestamps: created, modified, accessed     │
│ Block pointers: [15, 23, 8, 40, ...]        │ ← WHERE data is on disk
└─────────────────────────────────────────────┘
```

---

## What Does an Inode Store?

| Field | Description |
|-------|-------------|
| **File type** | Regular file, directory, symlink, device, etc. |
| **Permissions** | Read/write/execute for owner, group, others |
| **Owner (UID)** | User ID of file owner |
| **Group (GID)** | Group ID |
| **File size** | In bytes |
| **Link count** | Number of hard links pointing to this inode |
| **Timestamps** | ctime (metadata change), mtime (content modify), atime (last access) |
| **Block pointers** | Addresses of disk blocks containing file data |
| **Inode number** | Unique identifier within the file system |

### What an Inode Does NOT Store
-  File name (stored in directory)
-  Path (stored in directory)

---

## Inode Block Pointer Structure

This is the indexed allocation structure used by ext file systems:

```
Inode Block Pointers:
┌─────────────────────────────────────────────────────────┐
│ Direct pointers [0..11]  → 12 direct data block addrs  │
│                            12 × 4KB = 48KB max          │
├─────────────────────────────────────────────────────────┤
│ Single-indirect pointer  → points to 1 block of ptrs   │
│                            1 block = 1024 ptrs (4-byte) │
│                            1024 × 4KB = 4MB             │
├─────────────────────────────────────────────────────────┤
│ Double-indirect pointer  → ptr → block of ptr blocks    │
│                            1024 × 1024 × 4KB = 4GB      │
├─────────────────────────────────────────────────────────┤
│ Triple-indirect pointer  → 3 levels deep                │
│                            1024³ × 4KB = 4TB+           │
└─────────────────────────────────────────────────────────┘
```

### How a File is Read Using an Inode

```
open("/home/user/notes.txt")

Step 1: Read directory "/" → find "home" → inode 2
Step 2: Read inode 2 (/ dir) → get "home" dir block
Step 3: Read directory "home/" → find "user" → inode 150
Step 4: Read inode 150 (user dir) → get "user" dir block
Step 5: Read directory "user/" → find "notes.txt" → inode 1234
Step 6: Read inode 1234 → get file metadata + block pointers
Step 7: Read data blocks [15, 23, 8, 40] → file content
```

---

## Inode Table

All inodes are stored in the **inode table** — a fixed region on disk, created when the file system is formatted.

```
Disk layout (ext2/ext3):
┌────────────┬─────────────┬──────────────┬──────────────────┐
│ Boot Block │ Superblock  │ Inode Table  │  Data Blocks     │
└────────────┴─────────────┴──────────────┴──────────────────┘

Total inodes = fixed at format time
→ You can run out of inodes even if disk space is available!
   (Happens with millions of tiny files)
```

### Check inode usage: `df -i`

---

## Hard Links vs Soft Links (Inode Perspective)

### Hard Link
```
notes.txt  → Inode 1234
backup.txt → Inode 1234  ← same inode! different name, same file

Inode 1234: link_count = 2

Deleting notes.txt → link_count = 1 → data NOT deleted
Deleting backup.txt → link_count = 0 → data IS deleted
```

### Soft Link (Symlink)
```
shortcut.txt → Inode 9999 (a new inode)
               Inode 9999 contains: "/home/user/notes.txt" (path string)

Deleting notes.txt → shortcut.txt becomes a DANGLING link (broken)
```

| | Hard Link | Soft Link |
|-|-----------|-----------|
| Same inode? |  Yes |  No (new inode) |
| Works across file systems? |  No |  Yes |
| If original deleted | File still accessible | Broken link |
| Can link directories? |  No (usually) |  Yes |

---

##  Interview Questions & Answers

**Q: What is an inode?**
> An inode is a data structure in Unix/Linux file systems that stores all metadata about a file — type, permissions, owner, size, timestamps, and disk block pointers — but NOT the file name. The file name is stored in the directory, which maps names to inode numbers.

**Q: What does an inode contain?**
> File type, permissions (rwxrwxrwx), owner/group UIDs, file size, link count, three timestamps (ctime/mtime/atime), and block pointers (direct, single-indirect, double-indirect, triple-indirect).

**Q: How does the OS find a file's data given its path?**
> Starting from the root directory inode (always inode 2), the OS reads each directory entry to get the next component's inode number, until it reaches the file's inode. The inode's block pointers then give the disk locations of the actual file data.

**Q: What is the difference between a hard link and a soft link?**
> A hard link creates another directory entry pointing to the same inode. The file's data is only deleted when the link count drops to 0. A soft (symbolic) link is a separate file containing a path string to the target. Deleting the original makes the symlink dangling. Hard links can't cross file systems; symlinks can.

**Q: Can you run out of inodes even with free disk space?**
> Yes! The inode table is created with a fixed count at format time. If you create millions of tiny files, you can exhaust all inodes even if data blocks are still available. Check with `df -i`.

---

*← [Fragmentation](./03-Fragmentation.md) | Next → [Types of File Systems](./05-Types-of-File-Systems.md)*
