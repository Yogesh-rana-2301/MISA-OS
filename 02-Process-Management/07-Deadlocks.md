# 💀 Deadlocks

---

## What is a Deadlock?

A **deadlock** is a situation where a set of processes are **permanently blocked**, each waiting for a resource held by another process in the set.

> **Analogy**: A 4-way stop sign where 4 cars arrive simultaneously, each waiting for the car to their right to move — nobody moves forever.

```
Process A holds Resource 1, waits for Resource 2
Process B holds Resource 2, waits for Resource 1

A ──holds──→ R1 ←──waits── B
A ──waits──→ R2 ──holds──→ B

Neither A nor B can proceed. DEADLOCK!
```

---

## 4 Necessary Conditions for Deadlock

**ALL FOUR must hold simultaneously** for a deadlock to occur.  
Remove ANY ONE → deadlock cannot happen.

### 1. 🔒 Mutual Exclusion
> At least one resource must be held in a **non-shareable** mode (only one process at a time).

```
Resource R can only be held by ONE process at a time.
```

- Can't be eliminated for inherently non-shareable resources (e.g., printer, mutex)

---

### 2. 🤝 Hold and Wait
> A process holds at least one resource AND is waiting to acquire additional resources held by other processes.

```
Process A: holds R1, waits for R2
```

- If processes request all resources at once (or release before requesting new ones), this is eliminated

---

### 3. 🚫 No Preemption
> Resources cannot be forcibly taken from a process — they must be **voluntarily released**.

```
If A holds R1 and needs R2 (held by B), the OS cannot forcibly take R1 from A.
```

- Some resources can be preempted (e.g., CPU), others cannot (e.g., printer mid-print)

---

### 4. 🔄 Circular Wait
> A set of processes {P1, P2, ..., Pn} such that P1 waits for a resource held by P2, P2 waits for P3, ..., Pn waits for P1.

```
P1 → P2 → P3 → P1  (circular chain)
```

**Resource Allocation Graph with Circular Wait:**
```
P1 ──request──→ R1 ──assigned──→ P2
↑                                  │
└──assigned── R2 ←──request─────── ┘
```

---

## Memory Aid: "MHNC"
> **M**utual exclusion · **H**old and wait · **N**o preemption · **C**ircular wait

---

## Handling Deadlocks: 3 Approaches

```
Deadlock Handling
├── 1. Prevention  — Eliminate one of the 4 conditions
├── 2. Avoidance   — Don't enter unsafe states (Banker's Algorithm)
└── 3. Detection & Recovery — Allow deadlock, detect it, recover
    (+ Ignorance: "Ostrich Algorithm" — used in most real OSes!)
```

---

## 1. 🛡️ Deadlock Prevention

**Break at least one of the 4 necessary conditions:**

| Condition | How to Prevent | Drawback |
|-----------|---------------|----------|
| **Mutual Exclusion** | Use shareable resources | Not always possible (printers!) |
| **Hold and Wait** | Request ALL resources upfront | Low utilization; starvation possible |
| **No Preemption** | Allow OS to forcibly take resources | Only works for preemptible resources |
| **Circular Wait** | Order all resources; must acquire in order | Rigid, limits flexibility |

### Preventing Circular Wait (Most Common)

```
Assign a unique number to every resource type.
Processes MUST request resources in INCREASING order.

Resource order: R1 < R2 < R3 < R4

Process MUST acquire R1 before R2, R2 before R3, etc.
→ Circular wait is impossible by construction!
```

---

## 2. 🧠 Deadlock Avoidance

The OS dynamically checks if granting a resource request leads to an **unsafe state**.

### Safe State vs Unsafe State

```
SAFE STATE:   There exists a sequence of processes that can all finish
              (even if it requires waiting for resources to be freed in order)

UNSAFE STATE: No such sequence exists → deadlock POSSIBLE (not guaranteed)

DEADLOCK:     Processes are permanently blocked
```

```
Safe ──────────────── Unsafe ─────────── Deadlock
  ↑                      ↑
OS stays here          OS never enters here (avoidance)
```

**Key algorithm**: **Banker's Algorithm** (see [08-Bankers-Algorithm.md](./08-Bankers-Algorithm.md))

---

## 3. 🔍 Deadlock Detection & Recovery

**Don't prevent or avoid** — let deadlocks happen, then:
1. **Detect** the deadlock (run detection algorithm periodically)
2. **Recover** by breaking the deadlock

### Detection

Use a **Resource Allocation Graph** or Wait-For Graph:
- If a **cycle** exists → deadlock detected

```
Wait-For Graph (only processes, no resource nodes):
P1 → P2 → P3 → P1    ← cycle = DEADLOCK!
```

### Recovery Methods

| Method | How | Trade-off |
|--------|-----|-----------|
| **Process Termination** | Kill one or all deadlocked processes | Data loss |
| **Resource Preemption** | Take a resource away, roll back process | Complex rollback |
| **Rollback** | Roll a process back to a safe checkpoint | Requires checkpointing |

---

## Prevention vs Avoidance vs Detection

| | Prevention | Avoidance | Detection & Recovery |
|-|-----------|-----------|---------------------|
| **When** | At design time | At runtime (per request) | At runtime (after deadlock) |
| **How** | Eliminate a condition | Banker's algorithm | RAG / Wait-for graph |
| **Resource utilization** | Low | Medium | High |
| **Overhead** | Low | Medium | High (if frequent detection) |
| **Starvation risk** | Yes | Maybe | Low |
| **Used in** | Databases, some RTOS | Batch systems | General-purpose OS |

---

## 🦦 The Ostrich Algorithm

Most **general-purpose OSes** (Windows, Linux) use the **"Ostrich Algorithm"**:

> Ignore deadlocks. If one happens, the user reboots.

**Why?** Deadlocks are rare in practice, and the overhead of prevention/avoidance is not worth it for desktop/server OS.

---

## 🎯 Interview Questions & Answers

**Q: What is a deadlock?**
> A deadlock is a situation where a group of processes are permanently blocked, each holding a resource and waiting for a resource held by another process in the group. No process can proceed.

**Q: What are the 4 necessary conditions for deadlock?**
> 1. **Mutual Exclusion**: Resource held by only one process. 2. **Hold and Wait**: Process holds one resource while waiting for another. 3. **No Preemption**: Resources can't be forcibly taken. 4. **Circular Wait**: Circular chain of processes waiting for each other's resources. ALL four must hold simultaneously.

**Q: What is the difference between deadlock prevention and avoidance?**
> Prevention eliminates one of the 4 necessary conditions at design time (e.g., impose resource ordering to prevent circular wait). Avoidance dynamically checks each resource request at runtime and refuses if it would lead to an unsafe state (Banker's Algorithm). Prevention is simpler but more restrictive; avoidance is more flexible but needs advance information.

**Q: What is an unsafe state?**
> An unsafe state is one where no safe execution sequence exists — the OS cannot guarantee that all processes will eventually complete. An unsafe state may or may not lead to actual deadlock, but an avoidance algorithm prevents entering unsafe states to be safe.

**Q: How would you detect a deadlock?**
> Build a Wait-For Graph where there's an edge P1→P2 if P1 is waiting for a resource held by P2. If the graph has a cycle, a deadlock exists among the processes in the cycle.

---

*← [Semaphores](./06-Semaphores.md) | Next → [Banker's Algorithm](./08-Bankers-Algorithm.md)*
