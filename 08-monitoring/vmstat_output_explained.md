# Understanding `vmstat` Output

`vmstat` (Virtual Memory Statistics) reports system performance information collected from the Linux kernel. It shows information about processes, memory, paging, disk I/O, interrupts, context switches, and CPU usage.

A basic command is:

```bash
vmstat
```

For continuous monitoring:

```bash
vmstat 1
```

This updates the output every 1 second.

> **Important:** The first line of `vmstat 1` is an average since boot on many Linux systems, while the following lines represent activity during each interval. For real-time troubleshooting, focus mainly on the repeated lines.

```text
procs	  memory	                    swap	io	    system	cpu
r	b	  swpd	free	  buff	cache  si  so	bi	bo	in	cs	us	sy	id	wa	st
2	0	  1024	812340	98220	1543200	0	 0	12	45	210	340	18	4	  76	2	  0
5	2	  1024	790110	98220	1543400	0	 0	0	1820	245	410	30	12	10	48	0
$ vmstat 2  — each row is one 2-second snapshot, oldest first
```

---


## procs — who's competing for the CPU
 
| Column | Meaning |
|---|---|
| `r` | Processes/threads currently runnable (running or waiting for a CPU) — the instantaneous version of load average. |
| `b` | Processes stuck in uninterruptible sleep, usually waiting on disk I/O. |
 
**DevOps read**: `r` consistently above your core count → CPU-bound. `b` consistently above 0 → disk/IO-bound. Fast way to tell CPU vs. disk problems apart.

---

## memory — what RAM is doing
 
| Column | Meaning |
|---|---|
| `swpd` | Memory currently swapped out to disk. |
| `free` | RAM completely unused. |
| `buff` | RAM holding "bookkeeping" information about the disk itself — not the content of your files, but details like which file is stored where, how big it is, when it was last changed, and how the disk is organized into pieces. The computer needs to check this bookkeeping constantly just to find and access anything on disk, so it keeps a copy in fast RAM instead of re-reading it from the slow disk every time. |
| `cache` | RAM holding an actual copy of file contents you've recently read — the real data inside the files themselves, not information about them. |
 
**Important**: `buff`/`cache` is not wasted memory — Linux uses spare RAM for this deliberately because it's instant to reuse, and hands it back to a process the moment it's actually needed. Low `free` with high `cache` is healthy.
 
To make the difference concrete: say you run `ls` on a big directory, then a moment later open and read a large log file. The directory/filesystem bookkeeping involved in that `ls` (which blocks belong to which files, where they live on disk) gets held in `buff`. The actual text content of the log file you read gets held in `cache`. Both exist so that if you (or another process) ask for the same thing again soon, Linux can hand it back instantly from RAM instead of going back to the slow disk to fetch it again. `buff` tends to be a small, fairly stable number; `cache` is usually the much bigger one and grows as you read more files, shrinking automatically whenever a process needs that RAM for something else.
 
**DevOps read**: don't panic over low `free`; check `cache` first. The real warning sign is `swpd` climbing steadily — real, actively-used memory being forced out to slow disk because RAM is genuinely insufficient.

---

## 3. `swap` — Virtual Memory Paging Activity

Linux gives each process its own **virtual memory space**. Memory is divided into small units called **pages**, and Linux manages where those pages are stored.

Pages are normally kept in physical RAM, but Linux can move some pages between RAM and swap space on disk.

### `si` — Swap In

`si` shows how much data is read **from swap into physical RAM per second**.

```text
si = swap → RAM
```

### `so` — Swap Out

`so` shows how much data is moved **from physical RAM to swap per second**.

```text
so = RAM → swap
```

The values are rates, not the amount of swap currently being used.

### What to look for

Occasional `si` or `so` activity is not necessarily a problem.

Continuous, high `si` and `so` can indicate **memory pressure** — the system does not have enough readily available RAM for its workload and is repeatedly moving pages between RAM and disk.

Heavy swapping can make a system significantly slower.

---

## 4. `io` — Disk I/O Activity

This section shows data being read from and written to storage devices such as HDDs, SSDs, or virtual disks.

The values are shown as blocks per second.

### `bi` — Blocks In

`bi` shows how much data is being read **from storage into memory per second**.

```text
storage → memory
```

### `bo` — Blocks Out

`bo` shows how much data is being written **from memory to storage per second**.

```text
memory → storage
```

A **physical storage device** is an actual HDD or SSD.

A **virtual storage device** is a storage device presented by software, such as a virtual disk used by a virtual machine.

| Column | Meaning |
|---|---|
| `bi` | Blocks read in from disk per second. |
| `bo` | Blocks written out to disk per second. |
 
**DevOps read**: sustained high bi/bo alongside high b (from the procs section) confirms disk is your bottleneck, not CPU or memory.

Example above: row 2 shows bo jump from 45 to 1820 — a big burst of disk writes just happened (maybe a log flush, a database commit, something writing a lot of data).


### `buff` vs `bi`/`bo`

These are related to storage activity, but they measure different things:

```text
buff → how much RAM is being used for buffers
bi   → storage read activity
bo   → storage write activity
```

---

## system — kernel-level activity. This section shows how often the Linux kernel handles certain system-level events.
 
| Column | Meaning |
|---|---|
| `in` | Interrupts per second (hardware/software signals the CPU handles — network packets, timers, disk completions, etc). |
| `cs` | Context switches per second — how often the CPU swaps which thread it's running. |
 
**DevOps read**: very high `cs` = lots of small threads constantly interrupting each other instead of getting sustained CPU time — real overhead, and can itself be a source of slowdown.


---

## 6. `cpu` — CPU Time

This section shows how CPU time is being spent.

### `us` — User Time

`us` is the percentage of CPU time spent running **user-space programs**.

User space is where normal applications run, such as:

- Nginx
- Python
- Bash
- databases

### `sy` — System Time

`sy` is the percentage of CPU time spent running **kernel code** on behalf of the system.

For example, an application may request that Linux read a file. The application makes the request, but the kernel performs the low-level work.

### `id` — Idle Time

`id` is the percentage of CPU time when the CPU is idle and has no task to run.

A high `id` usually means the CPU has plenty of unused capacity.

### `wa` — I/O Wait

`wa` is the percentage of CPU time associated with waiting for I/O to complete.

I/O means input/output operations, such as reading from or writing to storage.

High `wa` can indicate that tasks are waiting on storage or another I/O operation.

However, `wa` alone does not prove that the disk is the problem. Check storage activity and application behavior as well.

### `st` — Steal Time

`st` is the percentage of CPU time that a virtual machine wanted to use but the **hypervisor** assigned that CPU time to another virtual machine.

This matters mainly inside virtual machines.

For example, if several virtual machines share the same physical host CPU, one VM may have to wait while another VM uses the physical CPU.

**Example above**: row 2 shows wa=48 — nearly half the CPU's time was spent idle-but-blocked on disk, which lines up with the bo=1820 write burst and b=2 from earlier. Same story, three different angles.

---

## 7. A Practical Example

Consider:

```text
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff   cache   si   so    bi    bo   in   cs   us sy id wa st
 8  2      0  12000    500  45000    0    0   800   200 1200 2500   70 10 15  5  0
```

A quick interpretation:

- `r = 8` → 8 tasks are runnable and competing for CPU time.
- `b = 2` → 2 tasks are blocked, likely waiting for a resource.
- `swpd = 0` → no swap space is currently being used.
- `si = 0`, `so = 0` → no active swap movement during this interval.
- `bi = 800` → storage is being read at 800 blocks/s.
- `bo = 200` → storage is being written at 200 blocks/s.
- `in = 1200` → about 1,200 interrupts/s.
- `cs = 2500` → about 2,500 context switches/s.
- `us = 70`, `sy = 10` → most CPU time is being spent running applications, with some spent in the kernel.
- `id = 15` → the CPU is idle for about 15% of the time.
- `wa = 5` → about 5% of CPU time is associated with I/O wait.
- `st = 0` → the VM is not currently losing CPU time to other VMs on the host.

The important point is to look at **patterns**, not one number by itself.

---

## What to check, organized by suspected problem
 
**CPU problem?** Look at: `r`, `us`, `sy`, `id`
A persistently high `r` together with low `id` points to CPU contention — more things want CPU time than the machine can hand out.
 
**Memory problem?** Look at: `free`, `swpd`, `si`, `so`
Some `swpd` in use isn't automatically alarming, but persistent, non-zero `si`/`so` is — that's active thrashing (memory being moved to and from disk in real time), which is a much stronger warning sign than just having a nonzero swap total sitting there.
 
**Storage/I/O problem?** Look at: `b`, `bi`, `bo`, `wa`
A high `b` or high `wa`, especially alongside real read/write activity in `bi`/`bo`, points to an I/O bottleneck — the disk can't keep up with what's being asked of it.
 
**Virtual machine CPU contention?** Look at: `st`
A consistently high `st` means your VM is ready to run but is waiting for the physical host to actually give it CPU time — usually because other VMs on the same host are using it. App-level tuning won't fix this; it's a capacity/neighbor issue at the infrastructure level.
 
### Quick summary by combination
 
- `b`, `wa`, `bi`/`bo` all rising together → **disk** bottleneck.
- `r` and `us` high, everything else calm → **CPU** bottleneck.
- `swpd`/`si`/`so` active → **memory** bottleneck.
- `st` consistently high → **noisy neighbor / host-level CPU contention** (cloud/VM only).
 
---

## 9. The Key Values to Remember

You do not need to memorize every column immediately.

For practical Linux/DevOps monitoring, start with:

```text
r     → runnable tasks
b     → blocked tasks
free  → unused RAM
swpd  → swap currently used
si    → swap → RAM
so    → RAM → swap
bi    → storage → memory
bo    → memory → storage
in    → interrupts
cs    → context switches
us    → user-space CPU time
sy    → kernel CPU time
id    → idle CPU time
wa    → I/O wait
st    → CPU time taken by other VMs
```


`vmstat` is most useful when you read several columns together and watch how they change over time.
