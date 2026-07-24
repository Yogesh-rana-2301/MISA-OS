# Disk Scheduling Algorithms

---

## Why Disk Scheduling?

An HDD has a physical **read/write head** that must mechanically **seek** to the correct track before reading/writing. Seek time is the dominant cost (~5–15ms).

When multiple I/O requests queue up, the order we service them affects total seek time significantly.

```
Disk tracks: 0 ─────────────────────── 199
Head position: currently at track 53

Requests waiting: 98, 183, 37, 122, 14, 124, 65, 67
```

**Goal**: Order requests to **minimize total head movement** (seek distance).

---

## Key Terms

| Term | Meaning |
|------|---------|
| **Track** | Concentric ring on disk platter |
| **Seek time** | Time to move head to target track |
| **Rotational latency** | Time for disk to rotate to target sector |
| **Transfer time** | Time to actually read/write data |
| **Total I/O time** | Seek + Rotation + Transfer |

Seek time dominates — scheduling algorithms minimize this.

---

## Setup for All Examples

```
Disk tracks: 0 to 199
Initial head position: 53
Request queue: 98, 183, 37, 122, 14, 124, 65, 67
```

---

## 1. FCFS — First Come First Served

> Service requests in the order they arrive.

```
Order: 53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67

Movement:
53→98:   45
98→183:  85
183→37:  146
37→122:  85
122→14:  108
14→124:  110
124→65:  59
65→67:   2

Total movement = 45+85+146+85+108+110+59+2 = 640 tracks
```

**Simple but inefficient** — head zigzags wildly.

---

## 2. SSTF — Shortest Seek Time First

> Service the request **closest to the current head position** first.

```
Head at 53. Requests: {98, 183, 37, 122, 14, 124, 65, 67}

Step 1: closest to 53 → 65 (diff=12)  → head: 65
Step 2: closest to 65 → 67 (diff=2)   → head: 67
Step 3: closest to 67 → 37 (diff=30)  → head: 37
Step 4: closest to 37 → 14 (diff=23)  → head: 14
Step 5: closest to 14 → 98 (diff=84)  → head: 98
Step 6: closest to 98 → 122 (diff=24) → head: 122
Step 7: closest to 122 → 124 (diff=2) → head: 124
Step 8: closest to 124 → 183 (diff=59)→ head: 183

Movement: 12+2+30+23+84+24+2+59 = 236 tracks
```

**Much better than FCFS. Problem: Starvation** — far requests may never be served if nearby requests keep arriving.

---

## 3. SCAN (Elevator Algorithm)

> Head moves in one direction, servicing all requests along the way, then **reverses** at the end.

Like an elevator — goes up servicing floors, then comes back down.

```
Head at 53, moving toward higher tracks.
Requests: {14, 37, 65, 67, 98, 122, 124, 183}

Going UP (from 53):
  53 → 65 → 67 → 98 → 122 → 124 → 183 → 199 (end of disk)

Then reverses (going DOWN):
  199 → 37 → 14

Total: (199-53) + (199-14) = 146 + 185 = 331 tracks
```

**No starvation** — head will eventually reach every track. Middle tracks get better average service than outer tracks.

---

## 4. C-SCAN (Circular SCAN)

> Head moves in ONE direction only. When it reaches the end, it jumps back to the beginning **without servicing** on the return.

Provides more uniform wait times than SCAN.

```
Head at 53, moving toward higher tracks.
Requests: {14, 37, 65, 67, 98, 122, 124, 183}

Going UP:
  53 → 65 → 67 → 98 → 122 → 124 → 183 → 199 (end, no service on return)

Jump to 0 (no service):
  0 → 14 → 37

Total seek: (199-53) + (199-0) + 37 = 146 + 199 + 37 = 382 tracks
```

Jump from 199 to 0 is counted but is fast (just repositioning).

**More uniform** service distribution than SCAN — outer tracks not disadvantaged.

---

## 5. LOOK

> Like SCAN, but head only goes **as far as the last request** in each direction — does NOT go all the way to disk end.

```
Head at 53, moving UP.
Requests: {14, 37, 65, 67, 98, 122, 124, 183}

Going UP to last request (183, not 199):
  53 → 65 → 67 → 98 → 122 → 124 → 183

Reverse (going DOWN to first request = 14):
  183 → 37 → 14

Total: (183-53) + (183-14) = 130 + 169 = 299 tracks
```

**Better than SCAN** — avoids unnecessary travel to disk edges.

---

## 6. C-LOOK (Circular LOOK)

> Like C-SCAN but only goes to **last request** before jumping back to **first request** (not track 0).

```
Head at 53, moving UP.
Requests: {14, 37, 65, 67, 98, 122, 124, 183}

Going UP to last request (183):
  53 → 65 → 67 → 98 → 122 → 124 → 183

Jump to smallest request (14):
  14 → 37

Total: (183-53) + (183-14) + (37-14) = 130 + 169 + 23 = 322 tracks
```

---

## Comparison Table

| Algorithm | Total Movement | Starvation | Direction | Notes |
|-----------|:-------------:|:---------:|:---------:|-------|
| FCFS | 640 | No | Any | Simple, worst performance |
| SSTF | 236 | Yes | Greedy | Best for random loads, starvation risk |
| SCAN | 331 | No | Both | Good, unfair to outer tracks |
| C-SCAN | 382 | No | One way | More uniform service |
| LOOK | 299 | No | Both | Better than SCAN (no edge waste) |
| C-LOOK | 322 | No | One way | Best balance of uniform + efficient |

---

## When to Use Which

| Scenario | Best Algorithm |
|----------|---------------|
| Light, unpredictable load | FCFS (simple) |
| High performance, no starvation concern | SSTF |
| General purpose HDD | LOOK or C-LOOK |
| Large systems needing uniform service | C-SCAN / C-LOOK |
| SSD | None needed (no seek time — any order same speed) |

> Note: SSDs have no moving parts, so seek time is ~constant regardless of track. Disk scheduling only matters for HDDs.

---

## Interview Questions & Answers

**Q: What is disk scheduling and why is it needed?**
> Disk scheduling determines the order in which I/O requests to an HDD are serviced. Since HDD seek time (mechanical arm movement) is the dominant cost, reordering requests can dramatically reduce total movement and improve throughput. Not needed for SSDs (no mechanical parts).

**Q: What is the difference between SCAN and C-SCAN?**
> SCAN (elevator) moves in both directions, servicing requests going up then coming back down. C-SCAN moves in one direction only; when it reaches the end, it jumps back to the beginning without servicing on the return. C-SCAN provides more uniform wait times — requests near the far end don't wait twice as long as requests in the middle.

**Q: What is the difference between SCAN and LOOK?**
> SCAN moves all the way to the end of the disk (track 0 or max) before reversing. LOOK only goes as far as the last pending request in each direction before reversing. LOOK avoids unnecessary travel to disk edges, making it more efficient.

**Q: Why does SSTF cause starvation?**
> SSTF always picks the request closest to the current head position. If new requests keep arriving near the current position, requests at far tracks are perpetually skipped. SCAN-based algorithms avoid this by sweeping across the disk in one direction, guaranteeing every track is eventually reached.

**Q: Why doesn't disk scheduling matter for SSDs?**
> SSDs use flash memory with no moving parts. Access time is uniform (~0.1ms) regardless of the "track" location. There is no seek or rotational latency. The OS still queues I/O requests for SSDs but can use simpler ordering (or even reorder for wear leveling) without needing complex seek-optimization algorithms.

---

*Back to [Topic 5 Index](../05-IO-Systems/README.md)*
