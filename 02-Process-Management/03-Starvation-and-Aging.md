#  Starvation & Aging

---

## The Problem: Starvation

**Starvation** occurs when a process is **perpetually denied CPU time** because higher-priority or shorter processes keep arriving.

> **Analogy**: You're standing in a checkout queue, but every time you're about to reach the counter, someone with "10 items or fewer" jumps ahead — forever.

---

## When Does Starvation Happen?

Starvation is a risk in **priority-based** and **SJF** scheduling.

### Example: Priority Scheduling Starvation

```
Processes arriving continuously with high priority:
t=0:   P_low (priority 10) enters Ready queue
t=1:   P_high1 (priority 1) arrives  → runs
t=5:   P_high2 (priority 1) arrives  → runs
t=9:   P_high3 (priority 2) arrives  → runs
...
→ P_low NEVER gets CPU time — it starves!
```

### Example: SJF Starvation

```
P_long (BT=100) is in Ready queue.
Every few seconds, short jobs (BT=1) arrive.
→ SJF always picks the short jobs first → P_long starves.
```

---

## Starvation vs Deadlock

| | Starvation | Deadlock |
|-|-----------|---------|
| **Definition** | Process waits indefinitely due to scheduling | Processes wait for each other indefinitely |
| **Cause** | Unfair scheduling policy | Circular resource dependency |
| **Resources** | Being used by others (not held) | Held by each other (circular wait) |
| **Solution** | Aging | Prevention/avoidance/detection |
| **Progress** | Other processes do make progress | No process makes progress |

---

## The Solution: Aging

**Aging** is a technique where the **priority of a waiting process is gradually increased** the longer it waits.

```
Initial priority of P_low = 10 (low priority)

After waiting 5 min  → priority bumped to 9
After waiting 10 min → priority bumped to 8
After waiting 15 min → priority bumped to 7
...
After waiting 50 min → priority bumped to 1
→ P_low now has highest priority → MUST run
```

### How Aging Works

```
Every time unit T a process waits without running:
  priority = max(priority − 1, max_priority)

OR:

  effective_priority = original_priority − (wait_time / aging_factor)
```

### Aging Properties

| Property | Detail |
|----------|--------|
| **Prevents starvation** |  Every process eventually reaches highest priority |
| **Fair** |  Long-waiting processes get priority boost |
| **Complexity** | Slight overhead to recalculate priorities |
| **Used in** | Linux nice values (partial aging), Windows thread priority boost |

---

## Real-World Example: Linux

Linux uses a **dynamic priority** system:
- Processes that haven't run recently get a **bonus** to their priority
- Processes that have used a lot of CPU get a **penalty**
- This is a form of aging built into the scheduler

```
Linux effective priority = nice_value − bonus

bonus increases as process waits longer without CPU time
→ prevents starvation naturally
```

---

##  Other Solutions to Starvation

| Solution | How |
|----------|-----|
| **Aging** | Increase priority of waiting processes over time |
| **Round Robin** | Use RR instead of strict priority (fair by design) |
| **Time-slice guarantee** | Give every process a minimum CPU time slice |
| **Multilevel Feedback Queue** | Move starving processes to higher-priority queue |

---

##  Interview Questions & Answers

**Q: What is starvation in operating systems?**
> Starvation occurs when a process is indefinitely denied CPU time because other processes always get priority. It happens in SJF (long jobs never run if short jobs keep arriving) and priority scheduling (low-priority processes never run if high-priority ones keep arriving).

**Q: What is the difference between starvation and deadlock?**
> In starvation, some processes make progress while the starving process waits indefinitely. In deadlock, no involved process makes progress — they all wait for each other in a circular chain. Starvation is a fairness problem; deadlock is a correctness problem.

**Q: How does aging solve starvation?**
> Aging gradually increases the priority of a waiting process over time. Even a low-priority process will eventually reach the highest priority level and be scheduled. This guarantees that every process eventually runs.

**Q: Does Round Robin cause starvation?**
> No. Round Robin gives each process a guaranteed time slice in rotation, so every process runs eventually. Starvation only occurs in algorithms that can indefinitely skip over certain processes.

---

*← [Context Switching](./02-Context-Switching.md) | Next → [Process Synchronization](./04-Process-Synchronization.md)*
