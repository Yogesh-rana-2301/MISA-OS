#  Topic 2 — Process Management

> **Goal**: Understand how the OS schedules, synchronizes, and protects processes — heavily tested in interviews.

---

##  Subtopics

| # | File | Topic |
|---|------|-------|
| 1 | [01-CPU-Scheduling.md](./01-CPU-Scheduling.md) | CPU Scheduling Algorithms |
| 2 | [02-Context-Switching.md](./02-Context-Switching.md) | Context Switching |
| 3 | [03-Starvation-and-Aging.md](./03-Starvation-and-Aging.md) | Starvation & Aging |
| 4 | [04-Process-Synchronization.md](./04-Process-Synchronization.md) | Process Synchronization & Critical Section |
| 5 | [05-Mutex-and-Spinlocks.md](./05-Mutex-and-Spinlocks.md) | Mutex & Spinlocks |
| 6 | [06-Semaphores.md](./06-Semaphores.md) | Semaphores |
| 7 | [07-Deadlocks.md](./07-Deadlocks.md) | Deadlocks |
| 8 | [08-Bankers-Algorithm.md](./08-Bankers-Algorithm.md) | Banker's Algorithm |

---

##  Quick Revision Checklist

- [ ] FCFS, SJF, Round Robin, Priority — know Gantt chart + formulas
- [ ] Calculate **waiting time** and **turnaround time** for each algorithm
- [ ] What exactly happens during a context switch?
- [ ] What is starvation? How does aging solve it?
- [ ] Three requirements of the critical section problem
- [ ] Mutex vs Semaphore vs Spinlock — differences
- [ ] Binary semaphore vs Counting semaphore
- [ ] **4 necessary conditions** for deadlock (memorize these!)
- [ ] Deadlock prevention vs avoidance vs detection
- [ ] Banker's algorithm — find safe sequence step by step

---

##  Top Interview Questions from This Topic

1. Compare FCFS, SJF, and Round Robin with examples.
2. What is a context switch? Why is it expensive?
3. What is the critical section problem? What are its requirements?
4. Difference between mutex and semaphore?
5. What are the four conditions for a deadlock?
6. How does Banker's algorithm work? Give an example.
7. What is starvation and how is it different from deadlock?
