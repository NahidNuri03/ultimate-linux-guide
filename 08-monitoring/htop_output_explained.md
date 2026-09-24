# `htop` --- Understanding the Output

`htop` is an interactive process viewer for Linux. It shows what is
happening on the system in real time, especially **CPU usage, memory
usage, load, and running processes**.

A typical `htop` screen has three main parts:

``` text
┌──────────────────────────────────────────────┐
│ CPU / Memory / Swap meters                   │
│ Tasks / Load average / Uptime                │
├──────────────────────────────────────────────┤
│ PID USER PRI NI VIRT RES SHR S CPU% MEM% ...│
│ ... processes ...                             │
├──────────────────────────────────────────────┤
│ F1 Help  F2 Setup  F3 Search ...             │
└──────────────────────────────────────────────┘
```

The exact columns and meters can vary depending on your `htop` version
and configuration.

------------------------------------------------------------------------

## 1. CPU Meters

At the top, `htop` normally shows one meter for each logical CPU.

For example, on a system with 4 logical CPUs:

``` text
CPU0 [|||||                         15.0%]
CPU1 [||||||||                     25.0%]
CPU2 [||||                         10.0%]
CPU3 [||||||||||||                 40.0%]
```

These meters show how busy each logical CPU currently is.

### Physical cores vs logical CPUs

A CPU core can sometimes run more than one hardware thread. Linux
normally presents each hardware thread as a **logical CPU**.

So a machine advertised as:

``` text
4 cores / 8 threads
```

will normally appear to Linux as:

``` text
8 logical CPUs
```

`htop` can therefore show 8 CPU meters.


------------------------------------------------------------------------

# 3. Memory (`Mem`) Meter

The memory meter shows how physical RAM is being used.

For example:

``` text
Mem [||||||||||||||||          60.0%  4.8G/8.0G]
```

The exact appearance varies by version/theme.


### Do not assume that all "used" RAM is permanently unavailable

Linux deliberately uses otherwise-unused RAM for things such as the page
cache.

For example:

``` text
8 GB RAM
├── 3 GB → applications
├── 2 GB → kernel
├── 2 GB → filesystem cache
└── 1 GB → readily available
```

The exact accounting depends on the tool and version.

The important idea is:

> Linux tries to make useful use of RAM. A large amount of memory being
> used does not automatically mean there is a memory problem.

------------------------------------------------------------------------

# 4. Swap (`Swp`) Meter

The swap meter shows the use of **swap space**.

Swap is storage space that Linux can use to hold memory pages that are
not currently kept in physical RAM.

Conceptually:

``` text
Physical RAM
      ↕
    Swap
   (disk)
```

Swap is much slower than RAM because it uses storage.

### Example

If:

``` text
Swp 500M / 4G
```

this means roughly:

> 500 MB of the available swap space is currently in use.

Swap being used does **not automatically mean the system is in
trouble**. Linux may move less-active memory pages to swap even when
there is still some RAM available.

What is more concerning is **heavy ongoing swap activity**, especially
when the system becomes slow.

------------------------------------------------------------------------

# 5. Tasks

Near the top, `htop` commonly shows something similar to:

``` text
Tasks: 120, 350 thr; 2 running
```

The exact wording varies by version.

These numbers describe the tasks/threads that `htop` is tracking.

Common information includes:

-   total tasks/processes
-   number of threads
-   number currently running

### Process vs thread

A **process** is a running program with its own process resources and
address space.

A **thread** is an execution unit within a process.

For example:

``` text
Firefox process
├── thread 1
├── thread 2
├── thread 3
└── ...
```

A multithreaded application can therefore contribute many threads while
still being one process.

Because Linux schedules threads, the number of runnable tasks is not
necessarily the same as the number of processes.

------------------------------------------------------------------------

# 6. Load Average

`htop` normally shows:

``` text
Load average: 0.50 1.20 1.00
```

These are the:

``` text
1-minute   5-minute   15-minute
```

load averages.

Load average represents the average number of tasks that were **running
or waiting for CPU, plus tasks in certain uninterruptible states**.


## How to interpret load on a 4-logical-CPU system

Suppose you have 4 logical CPUs.

### Load around 1

``` text
load average: 1.0
```

Roughly one task on average is demanding CPU or otherwise contributing
to load.

There is generally plenty of CPU capacity.

### Load around 4

``` text
load average: 4.0
```

The load is roughly equivalent to keeping all 4 logical CPUs busy.

### Load above 4

``` text
load average: 8.0
```

There are more tasks competing for CPU or otherwise contributing to load
than the 4 CPUs can execute simultaneously.

This can indicate CPU pressure, but **load average alone does not prove
a CPU problem**. I/O waits and other uninterruptible tasks can also
increase load.


------------------------------------------------------------------------

# 7. Uptime

`htop` can display something such as:

``` text
Uptime: 2 days, 4:31:12
```

This is how long the Linux system has been running since the last boot.

It is useful when interpreting other information.

For example, if a machine has:

``` text
Uptime: 12 days
```

and a process has been running for several days, that process may have
been active for most of the current boot.

------------------------------------------------------------------------

# 8. Process List

The largest section of `htop` is the process list.

A typical list might look like:

``` text
  PID USER      PRI  NI   VIRT   RES   SHR S  CPU% MEM%   TIME+  Command
 1234 nahid      20   0  500M  120M   20M R  45.0  1.5   2:31.20 python3 script.py
 1450 root       20   0  300M   50M   15M S   2.0  0.6   0:20.10 nginx
```

The columns are explained below.

------------------------------------------------------------------------

# 9. `PID` --- Process ID

**PID** means **Process ID**.

It is the number Linux assigns to a process.

Example:

``` text
PID
1234
```

means that this process has PID `1234`.

You can use the PID with other commands:

``` bash
kill 1234
```

or:

``` bash
ps -p 1234
```

The PID is one of the most useful pieces of information when
investigating a particular process.

------------------------------------------------------------------------

# 10. `USER`

This shows the user who owns the process.

Example:

``` text
USER
nahid
root
```

A process owned by `root` has privileges associated with the root user,
while a process owned by a normal user generally has that user's
permissions.

------------------------------------------------------------------------

# 11. `PRI` --- Priority

`PRI` shows the process's scheduling priority as presented by `htop`.

The Linux scheduler uses priority information when deciding which
runnable task should receive CPU time.

For ordinary processes, you will commonly see values around:

``` text
20
```

The exact value and meaning can depend on the scheduling policy.

You normally don't need to change `PRI` directly when learning basic
process management.

------------------------------------------------------------------------

# 12. `NI` --- Nice Value

`NI` means **nice value**.

It influences the scheduling priority of normal processes.

The normal nice value is:

``` text
0
```

A higher nice value generally means:

> Give this process less favorable CPU scheduling priority.

For example:

``` text
NI = 10
```

means the process has been made "nicer" to other processes and normally
receives less CPU priority.

A negative nice value gives a process more favorable priority, subject
to the user's permissions.

You can view or change nice values with tools such as:

``` bash
nice
renice
```

------------------------------------------------------------------------

# 13. `VIRT` --- Virtual Memory Size

`VIRT` shows the **virtual memory size associated with the process**.

It can include several kinds of virtual address space, including:

-   memory actually resident in RAM
-   memory mapped from files
-   shared libraries
-   allocated but not currently resident memory
-   other mapped address space

Therefore:

> **`VIRT` is not the amount of physical RAM the process is using.**

A process can have a large `VIRT` value while using relatively little
physical RAM.

------------------------------------------------------------------------

# `htop`: VIRT, RES, and SHR

These three columns are often confusing because they describe **different ways of looking at a process's memory**.

The key is to understand what Linux means by **virtual memory**, **physical RAM**, and **shared memory**.

---

## 1. `VIRT` — Virtual Memory

When you start a program, Linux gives that program a **virtual memory space**.

You can think of it as a memory map that the program is allowed to use:

```text
Your program
┌──────────────────────────┐
│ Code                     │
│ Variables                │
│ Data                     │
│ Libraries                │
│ Other memory             │
└──────────────────────────┘
        Virtual memory
```

The addresses that the program uses are **virtual addresses**. The program does not directly deal with the physical RAM addresses inside your computer.

Linux manages the relationship between those virtual addresses and the actual physical RAM.

### What does `VIRT` show?

**`VIRT` is the total amount of virtual memory space associated with the process.**

This does **not** mean:

> "This much physical RAM is currently being used."

For example:

```text
VIRT = 2.0G
RES  = 300M
```

This does **not** mean the program is using 2 GB of physical RAM.

It means the process has about **2 GB of virtual memory space associated with it**, while about **300 MB of that memory is currently in physical RAM**.

Why can `VIRT` be much larger?

Because some of that virtual memory may correspond to:

- memory currently in RAM
- memory that has not actually been used yet
- shared libraries
- files that the program has made available through memory
- memory that could currently be in swap

So:

> **`VIRT` is about the process's virtual memory space, not its actual physical RAM consumption.**

---

# 2. `RES` — Resident Memory

This one is easier once you understand the word **resident**.

**Resident simply means "currently present in physical RAM."**

So:

> **`RES` tells you approximately how much physical RAM is currently occupied by the process.**

For example:

```text
VIRT = 2.0G
RES  = 300M
```

You can think of it like this:

```text
Process's virtual memory
        2.0 GB
           │
           │ Linux manages it
           ↓
   ┌─────────────────┐
   │ Physical RAM    │
   │                 │
   │ 300 MB present  │
   └─────────────────┘
```

The rest of the process's virtual memory does not necessarily need to be sitting in RAM at this exact moment.

So when you're looking at `htop` and asking:

> **"How much physical RAM is this process currently using?"**

`RES` is generally much more useful than `VIRT`.

---

# 3. `SHR` — Shared Memory

Some memory can be **used by more than one process at the same time**.

For example, suppose you have:

```text
Firefox
LibreOffice
Terminal
```

They may all use the same system library.

Instead of putting three identical copies of that library into RAM:

```text
Firefox      → copy of library
LibreOffice  → copy of library
Terminal     → copy of library
```

Linux can keep **one copy in RAM** and allow multiple processes to use it:

```text
Firefox ───────┐
               │
LibreOffice ───┼──→ One copy in RAM
               │
Terminal ──────┘
```

That is **shared memory**.

`SHR` is related to the amount of memory associated with the process that **can be shared with other processes**.

This is why you should not simply add the `RES` values of every process and assume:

> "That's exactly how much RAM all my processes are using."

Some of the memory is shared.

---

# Putting All Three Together

Suppose `htop` shows:

```text
VIRT    RES    SHR
2.0G    300M   80M
```

A useful interpretation is:

### `VIRT = 2.0G`

The process has about **2 GB of virtual memory space** associated with it.

### `RES = 300M`

About **300 MB of that memory is currently present in physical RAM**.

### `SHR = 80M`

About **80 MB is associated with memory that can be shared with other processes**.

---


------------------------------------------------------------------------

# 16. `S` --- Process State

The `S` column shows the current process state.

Common states include:

``` text
R = Running or runnable
S = Sleeping
D = Uninterruptible sleep
T = Stopped or traced
Z = Zombie
I = Idle kernel thread
```

### `R` --- Running / Runnable

`R` means the task is running or is runnable and waiting to run.

On a system with multiple CPUs, several tasks can be runnable at the
same time.

### `S` --- Sleeping

The process is waiting for something and is not currently using the CPU. idle, waiting for something (a timer, network data, user input).

This is normal for many processes.


### `D` --- Uninterruptible Sleep

The process is normally waiting for something such as I/O and cannot
simply be interrupted in the usual way.

A large number of processes stuck in `D` state can be a sign of an I/O
problem.

### `T` --- Stopped

The process has been stopped, for example by a job-control signal or
debugger.

### `Z` --- Zombie

A zombie is a process that has finished executing but whose parent has
not yet collected its exit status.

A zombie does not continue executing like a normal process.

## DevOps read: 
mostly you're scanning for anomalies — lots of D states → disk/IO bottleneck; growing zombie count → a bug worth reporting; unexpectedly many R states at once → CPU contention, which load average will already have hinted at.

------------------------------------------------------------------------

# 17. `CPU%` --- CPU Usage

`CPU%` shows how much CPU time the process is currently consuming.

The exact interpretation depends on the `htop` version and
configuration, especially on multi-CPU systems.

For example:

``` text
CPU% = 100%
```

can represent one logical CPU being fully occupied.

On a system with 4 logical CPUs, a multithreaded process can potentially
use more than 100% if `htop` is configured to report CPU usage relative
to a single CPU.

For example:

``` text
CPU% = 250%
```

can mean the process is using roughly 2.5 logical CPUs' worth of CPU
time.

------------------------------------------------------------------------

# 18. `MEM%` --- Memory Percentage

`MEM%` shows the percentage of physical RAM associated with the process.

For example, on a machine with 8 GB RAM:

``` text
MEM% = 5%
```

means the process accounts for roughly 5% of the system's physical
memory according to `htop`'s memory accounting.

The exact calculation can vary with shared memory and `htop`
configuration.

------------------------------------------------------------------------

# 19. `TIME+` --- CPU Time Used

`TIME+` shows the accumulated CPU time used by the process since it
started.

For example:

``` text
TIME+
12:30.50
```

means the process has accumulated approximately 12 minutes and 30.5
seconds of CPU time.

This is **not necessarily the amount of real-world time the process has
been running**.

A process can exist for one hour but use only 30 seconds of CPU time if
it spends most of that hour sleeping.

------------------------------------------------------------------------

# 20. `Command` / `COMMAND`

This shows the command used to start the process.

For example:

``` text
python3 /home/nahid/script.py --port 8080
```

This is particularly useful when several processes have the same
executable name.

For example, you might have:

``` text
python3 server.py
python3 backup.py
python3 monitor.py
```

The command column lets you distinguish them.

------------------------------------------------------------------------

# 21. Threads in `htop`

Depending on configuration, `htop` can show individual threads.

You may see:

``` text
firefox
  ├─ thread
  ├─ thread
  └─ thread
```

or a process may appear as one entry while its threads are
hidden/grouped.

You can configure thread display from `htop`'s setup options.

This matters because Linux schedules **threads/tasks**, not just whole
processes.

------------------------------------------------------------------------

# 22. Sorting Processes

By default, `htop` commonly sorts processes by CPU usage, but this can
be changed.

Useful ways to sort include:

-   CPU usage
-   memory usage
-   PID
-   process state
-   user
-   priority
-   runtime

The exact keys available depend on your `htop` version.

A common workflow is:

``` text
Sort by CPU → find what is consuming the processor
Sort by memory → find what is consuming RAM
Sort by PID → find a particular process
```

------------------------------------------------------------------------

# 23. Searching for a Process

You can search for a process by name.

A common key is:

``` text
F3
```

or `/`, depending on the version/configuration.

For example, search for:

``` text
nginx
```

to find nginx-related processes.

------------------------------------------------------------------------

# 24. Killing a Process

`htop` allows you to select a process and send it a signal.

A common key is:

``` text
F9
```

This opens the signal menu.

Common signals include:

``` text
SIGTERM
SIGKILL
SIGSTOP
SIGCONT
```

### `SIGTERM`

Requests that the process terminate normally.

This should generally be preferred first because the program has an
opportunity to clean up.

### `SIGKILL`

Immediately terminates the process from the kernel's perspective.

Use it when a process does not respond to normal termination.

You should not automatically use `SIGKILL` just because a process is
consuming CPU or memory.

------------------------------------------------------------------------

# 25. `htop` vs `ps`

`htop` and `ps` both show process information, but they are useful in
different situations.

### `htop`

``` bash
htop
```

Good for:

-   watching processes interactively
-   observing CPU and memory usage
-   sorting processes
-   finding resource-heavy processes
-   sending signals
-   investigating a system in real time

### `ps`

``` bash
ps aux
```

Good for:

-   taking a snapshot
-   scripting
-   piping output into other commands
-   using on systems without `htop`

For example:

``` bash
ps aux | grep nginx
```

------------------------------------------------------------------------

# 26. `htop` vs `vmstat`

These tools overlap, but they answer different questions.

### `htop`

Focuses heavily on:

> **Which processes are using the system's resources?**

It shows individual processes and system meters.

### `vmstat`

Focuses more on:

> **What is the system doing overall?**

It shows system-wide information about:

-   runnable/blocked tasks
-   memory
-   swap activity
-   I/O
-   interrupts
-   context switches
-   CPU usage

For example:

``` text
htop:
python3       CPU 80%
nginx         CPU  2%
```

tells you **which processes** are using CPU.

Whereas:

``` text
vmstat:
r = 5
si = 0
so = 0
bi = 20
bo = 10
```

tells you about **overall system activity**.

------------------------------------------------------------------------

# 27. A Practical `htop` Troubleshooting Workflow

When a Linux machine feels slow, a useful sequence is:

## Step 1 --- Look at CPU meters

Ask:

> Are the CPUs heavily busy?

If yes, continue investigating CPU usage.

## Step 2 --- Check load average

Compare the load average with the number of logical CPUs.

For example:

``` text
4 logical CPUs
load average: 8.0
```

This indicates significant system demand, but you still need to
determine whether the cause is CPU work or blocked I/O.

## Step 3 --- Sort by CPU

Find the processes using the most CPU.

For example:

``` text
python3    180%
firefox     40%
```

A multithreaded process can exceed 100%.

## Step 4 --- Check memory

Look at the `Mem` meter and process `MEM%`.

If RAM is under heavy pressure, continue investigating memory.

## Step 5 --- Check swap

If swap is heavily used, that does not automatically prove a problem.

Look for evidence of **active swapping** and poor performance.

`vmstat` is particularly useful for checking `si` and `so`.

## Step 6 --- Investigate I/O if necessary

If CPU usage is not high but the system is slow, I/O may be involved.

Check:

``` bash
vmstat 1
```

and examine:

``` text
r
si
so
bi
bo
wa
```

This can help distinguish CPU pressure, memory pressure, and
storage-related delays.

------------------------------------------------------------------------

# 28. Important Values to Remember

For day-to-day Linux monitoring, these are especially useful:

  `htop` item    Meaning
  -------------- -------------------------------------------------------
  CPU meter      How busy each logical CPU is
  `Mem`          Physical RAM usage
  `Swp`          Swap space currently in use
  Tasks          Processes/threads and runnable tasks
  Load average   Average system load over 1, 5, and 15 minutes
  Uptime         Time since the last boot
  `PID`          Process ID
  `USER`         Process owner
  `PRI`          Scheduling priority
  `NI`           Nice value
  `VIRT`         Process virtual memory/address space
  `RES`          Process memory currently resident in RAM
  `SHR`          Memory associated with the process that can be shared
  `S`            Process state
  `CPU%`         Process CPU usage
  `MEM%`         Process memory percentage
  `TIME+`        Accumulated CPU time
  `Command`      Command used to start the process

------------------------------------------------------------------------

# 29. The Most Important Mental Picture

When reading `htop`, think from **system level → process level**:

``` text
                 htop
                   │
       ┌───────────┴───────────┐
       │                       │
   SYSTEM METERS          PROCESS LIST
       │                       │
       ├─ CPU                  ├─ PID
       ├─ Memory               ├─ USER
       ├─ Swap                 ├─ CPU%
       ├─ Load                 ├─ MEM%
       └─ Uptime               ├─ State
                               └─ Command
```

The top tells you:

> **"How is the whole Linux system doing?"**

The process list tells you:

> **"Which processes are responsible for the work?"**

That distinction is one of the most useful things to understand when
using `htop`.
