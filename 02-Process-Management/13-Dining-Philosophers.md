# Classic Synchronization Problem 3 — Dining Philosophers

---

## Problem Statement

- 5 philosophers sit around a circular table.
- 5 forks (chopsticks) — one between each pair of adjacent philosophers.
- Each philosopher alternates between **thinking** and **eating**.
- To eat, a philosopher needs **both the left and right fork**.
- Philosophers cannot share forks.

```
          Philosopher 0
         /              \
      fork 0          fork 4
       /                    \
 Phil 4                    Phil 1
       \                    /
      fork 3          fork 1
         \              /
          Philosopher 3
                |
              fork 2
                |
          Philosopher 2
```

---

## The Deadlock Scenario (Naive Solution Fails)

```c
// WRONG — deadlock possible!
void philosopher(int i) {
    while (true) {
        think();
        pick_up(fork[i]);          // pick left fork
        pick_up(fork[(i+1) % 5]); // pick right fork
        eat();
        put_down(fork[i]);
        put_down(fork[(i+1) % 5]);
    }
}
```

**Deadlock scenario:**
```
All 5 philosophers pick up their LEFT fork simultaneously:
  P0 holds fork[0], waiting for fork[1]
  P1 holds fork[1], waiting for fork[2]
  P2 holds fork[2], waiting for fork[3]
  P3 holds fork[3], waiting for fork[4]
  P4 holds fork[4], waiting for fork[0]

Circular wait → DEADLOCK! No philosopher can eat, ever.
```

This satisfies all 4 deadlock conditions:
- Mutual exclusion (one fork, one philosopher)
- Hold and wait (holding left, waiting for right)
- No preemption (forks not taken away)
- Circular wait (P0→P1→P2→P3→P4→P0)

---

## Solution 1: Allow Only 4 Philosophers at a Time

Break circular wait by limiting simultaneous access:

```c
semaphore room = 4;   // at most 4 philosophers at a table at once
semaphore fork[5] = {1,1,1,1,1};

void philosopher(int i) {
    while (true) {
        think();

        wait(room);                     // enter room (at most 4)
        wait(fork[i]);                  // pick left fork
        wait(fork[(i+1) % 5]);          // pick right fork
        eat();
        signal(fork[(i+1) % 5]);
        signal(fork[i]);
        signal(room);                   // leave room
    }
}
```

With at most 4 philosophers at the table, at least one can always pick up both forks.

---

## Solution 2: Asymmetric Solution (Resource Ordering)

Break circular wait by making **one philosopher pick up forks in opposite order**:

```c
semaphore fork[5] = {1,1,1,1,1};

void philosopher(int i) {
    while (true) {
        think();

        if (i % 2 == 0) {              // even philosophers: left then right
            wait(fork[i]);
            wait(fork[(i+1) % 5]);
        } else {                        // odd philosophers: right then left
            wait(fork[(i+1) % 5]);
            wait(fork[i]);
        }

        eat();

        signal(fork[i]);
        signal(fork[(i+1) % 5]);
    }
}
```

Why it works: P4 (even) picks fork[4] then fork[0]. P0 (even) picks fork[0] then fork[1]. Since P4 and P0 both want fork[0] first, one will get it and proceed, breaking the circular wait.

---

## Solution 3: Monitor-Based (Pickup/Putdown)

More elegant — a philosopher only picks up forks when BOTH are available:

```c
enum State { THINKING, HUNGRY, EATING };
State state[5];
semaphore self[5] = {0,0,0,0,0};
semaphore mutex = 1;

void test(int i) {
    // Try to let philosopher i eat
    if (state[i] == HUNGRY &&
        state[(i+4) % 5] != EATING &&
        state[(i+1) % 5] != EATING) {
        state[i] = EATING;
        signal(self[i]);   // philosopher i can now eat
    }
}

void pickup(int i) {
    wait(mutex);
    state[i] = HUNGRY;
    test(i);               // try to eat immediately
    signal(mutex);
    wait(self[i]);         // wait if can't eat yet (self[i]=0 means blocked)
}

void putdown(int i) {
    wait(mutex);
    state[i] = THINKING;
    test((i+4) % 5);       // check if left neighbor can now eat
    test((i+1) % 5);       // check if right neighbor can now eat
    signal(mutex);
}

void philosopher(int i) {
    while (true) {
        think();
        pickup(i);   // atomic: only eats if both forks available
        eat();
        putdown(i);
    }
}
```

This solution has **no deadlock and no starvation**.

---

## Solutions Comparison

| Solution | Deadlock Free | Starvation Free | Complexity |
|----------|:------------:|:---------------:|:----------:|
| Naive (pick both) | No | No | Simple |
| Limit to 4 philosophers | Yes | Maybe | Simple |
| Asymmetric ordering | Yes | Maybe | Simple |
| Monitor (pickup/putdown) | Yes | Yes | Complex |

---

## Interview Questions & Answers

**Q: What is the Dining Philosophers problem?**
> A classic deadlock illustration: 5 philosophers around a table with 5 forks. Each needs two forks to eat but can only pick one at a time. If all pick their left fork simultaneously, no one can pick the right — circular wait deadlock. Used to study deadlock conditions and synchronization.

**Q: How does the naive solution lead to deadlock?**
> If all 5 philosophers simultaneously pick up their left fork, each holds one fork and waits for the right (which the next philosopher holds). This forms a circular wait (P0→P1→P2→P3→P4→P0), satisfying all 4 deadlock conditions — resulting in permanent deadlock.

**Q: What is the asymmetric solution to the Dining Philosophers problem?**
> Make even-numbered philosophers pick up left fork then right, and odd-numbered philosophers pick up right then left. This breaks the circular wait condition because adjacent philosophers attempt to pick up forks in opposite orders — at least one can proceed.

**Q: How does the monitor solution prevent both deadlock and starvation?**
> A philosopher only transitions to EATING (and picks up both forks) when both neighbors are not eating — enforced atomically. When a philosopher finishes, it checks both neighbors. This prevents deadlock (no partial resource holding) and with a fair queue, prevents starvation too.

---

*Back to [Topic 2 Index](./README.md)*
