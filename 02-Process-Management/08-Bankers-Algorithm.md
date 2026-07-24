#  Banker's Algorithm

---

## What is the Banker's Algorithm?

The **Banker's Algorithm** is a **deadlock avoidance** algorithm developed by Dijkstra.

> **Analogy**: A bank has limited cash (resources). Customers (processes) request loans (resources). The bank only lends if it can guarantee it can satisfy all current and future requests — ensuring it never runs out of cash.

**Goal**: Before granting a resource request, check if doing so would leave the system in a **safe state**. If safe → grant. If unsafe → make the process wait.

---

##  Data Structures

For **n processes** and **m resource types**:

| Structure | Size | Meaning |
|-----------|------|---------|
| **Available** | 1 × m | Number of available instances of each resource type |
| **Max** | n × m | Maximum demand of each process |
| **Allocation** | n × m | Resources currently allocated to each process |
| **Need** | n × m | Remaining need = Max − Allocation |

```
Need[i][j] = Max[i][j] − Allocation[i][j]
```

---

##  Worked Example

### Setup

- 5 processes: P0, P1, P2, P3, P4
- 3 resource types: A (10 units), B (5 units), C (7 units)

### Current State

**Allocation** (what each process currently holds):

| Process | A | B | C |
|---------|---|---|---|
| P0 | 0 | 1 | 0 |
| P1 | 2 | 0 | 0 |
| P2 | 3 | 0 | 2 |
| P3 | 2 | 1 | 1 |
| P4 | 0 | 0 | 2 |
| **Total** | **7** | **2** | **5** |

**Max** (maximum each process may ever need):

| Process | A | B | C |
|---------|---|---|---|
| P0 | 7 | 5 | 3 |
| P1 | 3 | 2 | 2 |
| P2 | 9 | 0 | 2 |
| P3 | 2 | 2 | 2 |
| P4 | 4 | 3 | 3 |

**Available** = Total resources − Total allocated = (10,5,7) − (7,2,5) = **(3, 3, 2)**

**Need** = Max − Allocation:

| Process | A | B | C |
|---------|---|---|---|
| P0 | 7 | 4 | 3 |
| P1 | 1 | 2 | 2 |
| P2 | 6 | 0 | 0 |
| P3 | 0 | 1 | 1 |
| P4 | 4 | 3 | 1 |

---

##  Safety Algorithm — Step by Step

**Goal**: Find a safe sequence (an order in which all processes can finish).

### Algorithm

```
1. Work = Available                    // Work = (3,3,2) initially
2. Finish[i] = false for all i

3. Loop:
   Find an i such that:
     (a) Finish[i] == false
     (b) Need[i] <= Work   (element-wise: Need[i][j] <= Work[j] for all j)

   If found:
     Work = Work + Allocation[i]   // Process i finishes, releases resources
     Finish[i] = true

4. If Finish[i] == true for all i → SAFE STATE 
   Else → UNSAFE STATE 
```

### Trace

**Work = (3,3,2), Finish = [F,F,F,F,F]**

```
Step 1: Find process with Need ≤ Work (3,3,2):
  P0: Need=(7,4,3) > (3,3,2) 
  P1: Need=(1,2,2) ≤ (3,3,2)  → Pick P1
      Work = (3,3,2) + (2,0,0) = (5,3,2)
      Finish[P1] = true

Step 2: Work=(5,3,2), Finish=[F,T,F,F,F]
  P0: Need=(7,4,3) > (5,3,2) 
  P2: Need=(6,0,0) > (5,3,2)   (6 > 5 for A)
  P3: Need=(0,1,1) ≤ (5,3,2)  → Pick P3
      Work = (5,3,2) + (2,1,1) = (7,4,3)
      Finish[P3] = true

Step 3: Work=(7,4,3), Finish=[F,T,F,T,F]
  P0: Need=(7,4,3) ≤ (7,4,3)  → Pick P0
      Work = (7,4,3) + (0,1,0) = (7,5,3)
      Finish[P0] = true

Step 4: Work=(7,5,3), Finish=[T,T,F,T,F]
  P2: Need=(6,0,0) ≤ (7,5,3)  → Pick P2
      Work = (7,5,3) + (3,0,2) = (10,5,5)
      Finish[P2] = true

Step 5: Work=(10,5,5), Finish=[T,T,T,T,F]
  P4: Need=(4,3,1) ≤ (10,5,5)  → Pick P4
      Work = (10,5,5) + (0,0,2) = (10,5,7)
      Finish[P4] = true
```

**Safe Sequence: P1 → P3 → P0 → P2 → P4 **

All Finish[i] = true → System is in a **SAFE STATE**.

---

##  Resource Request Algorithm

When a process Pi requests resources **Request[i]**:

```
1. If Request[i] > Need[i]:
       ERROR — process exceeds its maximum claim

2. If Request[i] > Available:
       WAIT — resources not available right now

3. Pretend to allocate:
       Available  = Available  − Request[i]
       Allocation[i] = Allocation[i] + Request[i]
       Need[i]    = Need[i]    − Request[i]

4. Run Safety Algorithm:
       If result is SAFE  → grant the request 
       If result is UNSAFE → rollback step 3, make Pi wait 
```

### Example Request

P1 requests (1, 0, 2):

```
Check 1: (1,0,2) ≤ Need[P1]=(1,2,2)? 
Check 2: (1,0,2) ≤ Available=(3,3,2)? 
Pretend: Available=(2,3,0), Alloc[P1]=(3,0,2), Need[P1]=(0,2,0)
Run safety → safe sequence exists → GRANT 
```

---

##  Limitations of Banker's Algorithm

| Limitation | Detail |
|-----------|--------|
| **Must know Max in advance** | Processes must declare maximum resource need upfront (not practical) |
| **Fixed resources** | Assumes total resources don't change |
| **Fixed processes** | Assumes process count is fixed |
| **Overhead** | Safety check runs on every request — O(n²·m) |
| **Conservative** | May refuse safe grants (too pessimistic) |

---

##  Quick Reference Summary

```
Key Tables:
  Max        = Maximum need (declared upfront)
  Allocation = Currently held
  Need       = Max − Allocation
  Available  = Total − sum(Allocation)

Safety Algorithm:
  Simulate: find process whose Need ≤ Work
  Release its resources: Work += Allocation
  Repeat until all done (SAFE) or stuck (UNSAFE)

Request Algorithm:
  Request ≤ Need ≤ Available → pretend → safety check → grant/wait
```

---

##  Interview Questions & Answers

**Q: What is the Banker's Algorithm?**
> The Banker's Algorithm is a deadlock avoidance algorithm. Before granting a resource request, it simulates the allocation and checks if the resulting system state is safe (there exists a sequence where all processes can complete). If safe, it grants the request; otherwise, the process waits.

**Q: What is a safe state?**
> A safe state is one where there exists at least one safe sequence — an ordering of processes such that each process can obtain its needed resources (using currently available resources plus resources released by earlier processes in the sequence), execute, and terminate.

**Q: How do you find the safe sequence?**
> Set Work = Available. Find a process whose Need ≤ Work. Add its Allocation to Work (simulating it finishing). Mark it done and repeat. If all processes finish, the sequence found is the safe sequence. If you get stuck, the state is unsafe.

**Q: What is the Need matrix?**
> Need[i][j] = Max[i][j] − Allocation[i][j]. It represents how many more resources of each type process i may still request.

**Q: What are the limitations of Banker's Algorithm?**
> Processes must declare their maximum resource needs upfront (unrealistic). It assumes a fixed number of resources and processes. It has O(n²·m) overhead per request. It can be overly conservative, denying some requests that would actually be safe.

---

*← [Deadlocks](./07-Deadlocks.md) | Back to [Topic 2 Index](./README.md)*
