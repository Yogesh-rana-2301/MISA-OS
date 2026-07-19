# 📂 File System Basics

---

## What is a File System?

A **file system** is the OS component responsible for **organizing, storing, retrieving, and managing data on storage devices** (HDD, SSD, flash).

> Without a file system, a disk is just raw bytes with no structure. The file system creates the **abstraction of files and directories**.

```
Without File System:          With File System:
┌─────────────────────┐       ┌─────────────────────────┐
│ 01001000 01100101   │  →    │ /home/user/             │
│ 01101100 01101100   │       │   ├── notes.txt         │
│ 01101111 00100000   │       │   ├── photo.jpg         │
│ ... raw bytes ...   │       │   └── projects/         │
└─────────────────────┘       └─────────────────────────┘
```

---

## What Does a File System Do?

| Function | Description |
|----------|-------------|
| **File naming** | Maps human-readable names to disk locations |
| **Directory structure** | Organizes files into a tree hierarchy |
| **Storage allocation** | Decides where files are stored on disk |
| **Free space management** | Tracks which disk blocks are free |
| **Access control** | Permissions — who can read/write/execute |
| **Metadata management** | Stores file size, timestamps, ownership |
| **Reliability** | Ensures data isn't corrupted (journaling) |

---

## Key Components

### 1. Files
The basic unit — a named collection of related data.

**File attributes (metadata)**:
```
Name:         notes.txt
Size:         4096 bytes
Type:         regular file
Owner:        user_id 1000
Permissions:  rw-r--r--  (owner:rw, group:r, others:r)
Created:      2024-01-15 10:30:00
Modified:     2024-03-20 14:22:10
Accessed:     2024-07-16 08:00:00
Disk blocks:  [45, 46, 47, 92]
```

### 2. Directories
A special file that maps **file names → file metadata (inode numbers)**.

```
Directory "/home/user/":
┌───────────────┬──────────────┐
│  File Name    │  Inode #     │
├───────────────┼──────────────┤
│  notes.txt    │  1234        │
│  photo.jpg    │  5678        │
│  projects/    │  9012        │
│  .            │  9000        │  ← current dir
│  ..           │  8500        │  ← parent dir
└───────────────┴──────────────┘
```

### 3. Disk Blocks / Clusters
The smallest unit of disk storage the file system manages (typically 4KB).

```
Disk divided into blocks:
┌──────┬──────┬──────┬──────┬──────┬──────┐
│Block0│Block1│Block2│Block3│Block4│Block5│...
│(boot)│(superblk)│(inode)│(data)│(data)│(free)│
└──────┴──────┴──────┴──────┴──────┴──────┘
```

### 4. Superblock
The **superblock** contains critical file system metadata:
- Total number of blocks
- Number of free blocks
- Inode count, free inode count
- Block size
- File system type/magic number

---

## File Operations (System Calls)

| Operation | System Call | Description |
|-----------|------------|-------------|
| Create | `open(O_CREAT)` | Create new file |
| Open | `open()` | Get file descriptor |
| Read | `read()` | Read bytes from file |
| Write | `write()` | Write bytes to file |
| Close | `close()` | Release file descriptor |
| Delete | `unlink()` | Remove directory entry |
| Seek | `lseek()` | Move file pointer |
| Stat | `stat()` | Get file metadata |

---

## File Descriptor

When a process opens a file, the OS returns a **file descriptor (fd)** — an integer index into the process's open file table.

```
fd = 0 → stdin
fd = 1 → stdout
fd = 2 → stderr
fd = 3 → first opened file
fd = 4 → second opened file ...
```

---

## 🎯 Interview Questions & Answers

**Q: What is a file system?**
> A file system is the OS subsystem that organizes, stores, and retrieves data on storage devices. It provides the abstraction of files and directories, manages disk space allocation, controls access permissions, and maintains metadata.

**Q: What is the difference between a file and a directory?**
> A file is a named collection of data. A directory is a special file that contains a mapping of file names to their inode numbers — it's essentially a table of contents for a folder. Directories can contain other directories (subdirectories), creating a hierarchical tree structure.

**Q: What is a file descriptor?**
> A file descriptor is a small integer returned by the OS when a process opens a file. It indexes into the process's open file table, which points to the actual file's in-memory data structures. Standard descriptors: 0=stdin, 1=stdout, 2=stderr.

**Q: What does a superblock contain?**
> The superblock stores critical file system metadata: total blocks, free blocks, inode count, free inodes, block size, and the file system type. It's typically stored at the start of the partition. A corrupted superblock means the entire file system may be unreadable.

---

*Next → [File Allocation Methods](./02-File-Allocation-Methods.md)*
