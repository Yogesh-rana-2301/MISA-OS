#  Function Call vs System Call

---

##  The Core Distinction

| | Function Call | System Call |
|-|--------------|-------------|
| **Stays in** | User space | Crosses into Kernel space |
| **Mode switch** |  No |  Yes (User → Kernel → User) |
| **Who handles it** | Your program / library | OS Kernel |
| **Speed** | Very fast (nanoseconds) | Slower (microseconds) |
| **Examples** | `printf()`, `sqrt()`, `strlen()` | `read()`, `write()`, `fork()` |

---

##  User Space vs Kernel Space

```
┌────────────────────────────────────────────────────────────────┐
│                      USER SPACE                                │
│                                                                │
│  Your Program                                                  │
│  int main() {                                                  │
│      printf("hello");   ← function call (stays in user space) │
│      read(fd, buf, n);  ← system call (goes to kernel!)       │
│  }                                                             │
│                                                                │
│  C Standard Library (glibc / libc)                            │
│      printf() → formats string → calls write() (syscall)      │
│                                                    │           │
├────────────────────────────────────────────────────┼───────────┤
│                      KERNEL SPACE                  │           │
│                                                    ▼           │
│   OS handles: write() → disk I/O → return result              │
└────────────────────────────────────────────────────────────────┘
```

---

##  Function Call — How It Works

A regular function call is **simple and fast**:

```
main() calls printf()
        │
        ▼
Stack frame pushed  ←  parameters, return address
        │
printf() executes
        │
        ▼
Stack frame popped  ←  returns value
        │
main() resumes
```

- **No mode switch**
- **No context switch**
- All in user space
- Cost: ~1–5 nanoseconds

---

##  System Call — How It Works

A system call involves the **OS kernel**:

```
1. App calls read(fd, buffer, 1024)
        │
2. Library wrapper (glibc) sets up:
   - Syscall number in register (e.g., EAX = 0 for read on Linux)
   - Arguments in registers (RDI=fd, RSI=buffer, RDX=1024)
        │
3. Execute SYSCALL instruction  ←── triggers mode switch!
        │
4. CPU mode bit: 1 → 0 (User → Kernel)
   OS saves user context (PC, registers) to kernel stack
        │
5. OS kernel:
   - Validates arguments
   - Performs actual I/O operation
   - Copies data to user buffer
        │
6. Execute SYSRET instruction  ←── return to user mode
        │
7. CPU mode bit: 0 → 1 (Kernel → User)
   Restores user context
        │
8. App resumes — read() returns number of bytes read
```

---

##  Common System Call Examples

### File Operations

| System Call | Purpose | Example |
|-------------|---------|---------|
| `open()` | Open a file | `fd = open("file.txt", O_RDONLY)` |
| `read()` | Read from file/fd | `read(fd, buffer, 1024)` |
| `write()` | Write to file/fd | `write(fd, "hello", 5)` |
| `close()` | Close file descriptor | `close(fd)` |
| `lseek()` | Reposition file offset | `lseek(fd, 0, SEEK_SET)` |

### Process Control

| System Call | Purpose | Example |
|-------------|---------|---------|
| `fork()` | Create child process | `pid = fork()` |
| `exec()` | Replace process image | `execl("/bin/ls", "ls", NULL)` |
| `exit()` | Terminate process | `exit(0)` |
| `wait()` | Wait for child process | `wait(&status)` |
| `getpid()` | Get process ID | `pid = getpid()` |

### Memory

| System Call | Purpose |
|-------------|---------|
| `mmap()` | Map memory (or files) |
| `brk()` | Adjust heap size |
| `munmap()` | Unmap memory |

---

##  Deep Dive: `printf()` vs `write()`

```
printf("Hello\n")
    │
    ├── 1. Format the string (user space)
    ├── 2. Buffer the output (user space)  
    ├── 3. When buffer full or newline → calls write() ← SYSCALL
    │                                          │
    │                                    [mode switch]
    │                                          │
    │                                    Kernel writes to stdout
    │
    └── Returns to printf → returns to your code
```

**Key insight**: `printf()` is a **library function** that eventually calls `write()` (**system call**). Most C library functions are wrappers that eventually invoke system calls.

---

##  Overhead Comparison

| | Function Call | System Call |
|-|--------------|-------------|
| **Mode switch** | None | User → Kernel → User |
| **Context save** | None (just stack frame) | Registers, PC, status |
| **Validation** | None | OS validates args |
| **Cache effects** | Minimal | Flushes CPU pipeline, TLB |
| **Typical cost** | 1–5 ns | 100–1000 ns (100–1000× slower) |

> **Takeaway**: System calls are expensive. High-performance programs minimize syscalls (e.g., buffering I/O to batch `write()` calls).

### Real-World Optimization Example

```
// SLOW: 1 syscall per character (10,000 syscalls for 10KB)
for (char c : data) {
    write(fd, &c, 1);
}

// FAST: 1 syscall for all data
write(fd, data, data.size());
```

---

##  fork() — A Special System Call

`fork()` is one of the most important system calls in UNIX:

```
Parent Process (PID 100)
    │
    │ calls fork()
    ├─────────────────────────────────────────────────┐
    ▼                                                 ▼
Parent (PID 100)                              Child (PID 101)
fork() returns 101 (child's PID)              fork() returns 0
(continues from here)                         (exact copy of parent's memory)
```

- **fork()** creates an **exact copy** of the calling process
- Both parent and child continue from the line **after** `fork()`
- Child gets a new PID
- Parent gets child's PID, child gets 0 — this is how you tell them apart

```c
pid_t pid = fork();
if (pid == 0) {
    // CHILD process
    printf("I am the child, PID: %d\n", getpid());
} else {
    // PARENT process
    printf("I am the parent, child PID: %d\n", pid);
}
```

---

##  Summary

```
Function Call:
App → [User Space] → Library function → returns → App
(fast, no mode switch)

System Call:
App → [User Space] → SYSCALL instruction
                           ↓
                    [Kernel Space] → OS handles
                           ↓
                    SYSRET instruction → [User Space]
(slow, mode switch, but necessary for hardware access)
```

---

##  Interview Questions & Answers

**Q: What is the difference between a function call and a system call?**
> A function call stays within user space and is handled by the program or library — fast and cheap. A system call crosses from user space to kernel space via a CPU trap, allowing the OS to perform privileged operations. System calls are ~100–1000× slower due to the mode switch overhead.

**Q: What happens when you call read() in a C program?**
> The C library sets up the syscall number and arguments in registers, then executes the SYSCALL instruction. The CPU switches to kernel mode, the OS validates arguments and reads from the file/device, copies data to the user buffer, then returns via SYSRET back to user mode with the result.

**Q: Why are system calls expensive?**
> They require a mode switch (user → kernel → user), saving/restoring CPU registers and program counter, flushing the CPU pipeline, and potentially invalidating cache. This overhead is ~100–1000ns vs ~1–5ns for a regular function call.

**Q: What does fork() do?**
> fork() creates an exact copy of the calling process. The parent receives the child's PID, the child receives 0. Both continue execution from the instruction after fork(). The child gets a new PID and its own copy of the parent's address space.

**Q: Is printf() a system call?**
> No — printf() is a C library function (user space). It formats the string and buffers it, then internally calls write() which IS a system call that crosses into kernel space to output to stdout.

---

*← [Process and States](./05-Process-and-States.md) | Back to [Topic 1 Index](./README.md)*
