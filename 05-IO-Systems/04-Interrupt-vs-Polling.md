#  Interrupt vs Polling

---

## The Core Problem

The CPU and I/O devices operate at very different speeds.  
The CPU needs to know when a device is done with its operation.  
**How does it find out?**

Two approaches: **Polling** and **Interrupts**.

---

## Polling (Busy-Waiting)

> **The CPU repeatedly checks (polls) the device status register to see if it's done.**

```
CPU starts I/O operation on device
    │
    ▼
CPU enters polling loop:
┌─────────────────────────────────────┐
│  while (device.status != DONE):     │ ← checking over and over
│      /* do nothing, just spin */    │ ← wasting CPU cycles
└─────────────────────────────────────┘
    │
    ▼
Device done! Exit loop.
CPU reads data from device.
```

### Timeline

```
───────────────────────────────────────────────────→ time
CPU:    [start I/O][POLL][POLL][POLL][POLL][POLL][process result]
Device:            [════════════ working ═════]
                                              ↑
                              CPU only stops polling when device done
```

### Characteristics

| Property | Detail |
|----------|--------|
| CPU during I/O | Busy-waiting (wasted cycles) |
| Latency |  Very low — detects completion immediately |
| CPU efficiency |  Poor — monopolizes CPU for one I/O |
| Complexity |  Simple to implement |
| Best for | Very fast devices, short waits, embedded systems |

---

## Interrupt-Driven I/O

> **The CPU starts the I/O and goes on to do other work.  
> When the device completes, it sends a hardware INTERRUPT signal to the CPU.**

```
CPU starts I/O operation on device
    │
    ▼
CPU runs OTHER processes while device works
    │         (useful work happening!)
    │
    [INTERRUPT SIGNAL from device] ──────────────────────┐
    │                                                     │
    ▼                                                     │
CPU receives interrupt:                                   │
  1. Saves current context (PC, registers)               │
  2. Jumps to Interrupt Service Routine (ISR)            │◀────────┘
  3. ISR reads data from device
  4. ISR signals waiting process (moves to READY)
  5. Restores context, returns to previous work
```

### Timeline

```
───────────────────────────────────────────────────→ time
CPU:    [start I/O][─────── OTHER WORK ──────────][ISR][resume work]
Device:            [════════════ working ═════════]↑
                                                   │
                                             Device fires interrupt
```

### Characteristics

| Property | Detail |
|----------|--------|
| CPU during I/O | Does other useful work  |
| Latency | Slightly higher (interrupt handling overhead ~microseconds) |
| CPU efficiency |  Excellent |
| Complexity |  More complex (ISR, interrupt controller, context save) |
| Best for | Slow devices (disk, network, keyboard) |

---

## How Interrupts Work (Hardware Level)

```
Interrupt Controller (PIC / APIC):
  - Each device has an Interrupt Request (IRQ) line
  - Keyboard → IRQ 1
  - Disk     → IRQ 14/15
  - Network  → assigned IRQ

When device is done:
  Device → asserts IRQ line → Interrupt Controller
  Interrupt Controller → signals CPU
  CPU → finishes current instruction → checks interrupt flag
  CPU → saves state → jumps to ISR (via Interrupt Vector Table)
  ISR → handles I/O → sets EOI (End of Interrupt) → return
```

### Interrupt Vector Table (IVT)

```
IVT: array of function pointers (ISR addresses)
  IVT[0]  → divide-by-zero handler
  IVT[1]  → keyboard ISR
  IVT[14] → disk ISR
  IVT[32+] → software interrupt handlers (system calls!)
```

---

## DMA — Direct Memory Access (Bonus — Very Important!)

Even with interrupts, if the CPU has to copy data byte-by-byte from device to RAM, it's inefficient.

**DMA Controller** handles the data transfer autonomously:

```
Without DMA:
  Device ready → interrupt → CPU copies 1MB byte by byte → slow!

With DMA:
  CPU tells DMA: "Copy 1MB from device to address 0x8000"
  CPU goes back to work
  DMA copies data directly (RAM ↔ Device), no CPU involvement
  DMA fires ONE interrupt when done: "transfer complete"
  CPU handles result
```

| | Without DMA | With DMA |
|-|------------|---------|
| CPU involvement | Every byte | Only setup + completion |
| CPU efficiency |  Poor |  Excellent |
| Used for | Simple/old devices | Modern disks, NICs, GPUs |

---

## Polling vs Interrupt — Full Comparison

| Feature | Polling | Interrupt-driven |
|---------|---------|-----------------|
| **CPU during I/O** | Wasted (busy-wait) | Doing other work  |
| **Response latency** |  Very low (immediate) | Slightly higher (ISR overhead) |
| **CPU efficiency** |  Poor |  Excellent |
| **Complexity** |  Simple |  More complex |
| **Device speed** | Fast devices (short wait) | Slow devices (long wait) |
| **Multiple devices** |  Hard (CPU stuck on one) |  Easy (each fires own IRQ) |
| **Examples** | Embedded controllers, game controllers | Keyboard, disk, NIC, USB |

---

## When to Use Which?

```
Device response time vs CPU cycle time:

  Device VERY fast (μs):   Polling might be better
                            (interrupt overhead > polling wait)

  Device SLOW (ms+):        Interrupts always better
                            (CPU can do thousands of operations while waiting)

Rule of thumb:
  Disk/Network/USB → Interrupts
  Ultra-fast NVMe SSD (in some kernel drivers) → Polling can be faster!
  (Linux has both modes in its NVMe driver)
```

---

##  Interview Questions & Answers

**Q: What is the difference between interrupt-driven I/O and polling?**
> Polling: the CPU loops, continuously checking if the device is ready — simple but wastes CPU cycles. Interrupt-driven: the CPU starts the I/O and does other work; the device signals the CPU via a hardware interrupt when done — efficient but more complex. Interrupts are preferred for slow devices (disk, keyboard, network).

**Q: Which is more CPU-efficient — interrupts or polling?**
> Interrupt-driven I/O is more CPU-efficient for slow devices because the CPU does useful work while waiting. Polling wastes CPU cycles spinning in a loop doing nothing. However, for very fast devices, polling can sometimes be faster because interrupt overhead (context save, ISR, return) can exceed the device's response time.

**Q: What is an ISR (Interrupt Service Routine)?**
> An ISR (also called an interrupt handler) is a special function registered to handle a specific interrupt. When a device fires its IRQ, the CPU saves its current context and jumps to the ISR via the Interrupt Vector Table. The ISR handles the event (reads data, signals a waiting process) and returns.

**Q: What is DMA and why is it useful?**
> DMA (Direct Memory Access) allows a dedicated DMA controller to transfer data directly between a device and RAM without CPU involvement. The CPU only initiates the transfer and receives one interrupt at completion. This is critical for high-bandwidth devices (disks, GPUs, NICs) — without DMA, the CPU would waste time copying every byte.

---

*← [Spooling](./03-Spooling.md) | Back to [Topic 5 Index](./README.md)*
