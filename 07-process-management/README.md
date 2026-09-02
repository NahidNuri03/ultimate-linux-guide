# Process Management in Linux

## 🔄 Introduction to Process Management

A **process** is simply an active instance of a running program. Every time you run a command, open a text editor, or start a web server, Linux creates a process. 

To keep everything organized, the operating system assigns two core tracking properties to every process:
*   **PID (Process ID):** A unique identification number assigned to that specific running instance.
*   **PPID (Parent Process ID):** The identifier of the program that launched it. (For example, if you run `ls` inside a Bash terminal, Bash is the "parent" process, and `ls` is the "child" process).

---


### Viewing Processes
- `ps aux` – View all running processes
- `ps -u username` – View running processes for a specific user
- `ps -C processname` – Filters and shows running instances matching an exact process name. If you want to see who owns the process (User ID) and its parent process, add the -f flag. The time here means the amount of CPU time that the process has consumed. For example, if it's one second, it does not mean that the process has been running for one second. The process could have been running for three days but have only used one second of CPU time because it spends most of its time waiting.
```bash
ps -C nginx

 PID TTY          TIME CMD
1432 ?        00:00:05 nginx
1433 ?        00:00:12 nginx
```
```bash
ps -o pid,%cpu,%mem -C chrome

 PID %CPU %MEM
4102  0.1  1.4
4109  5.2  3.8
4115  0.0  0.9
4520 22.4  4.2
```
%cpu: Displays the percentage of CPU (processor) power that specific process is actively using.
%mem: Displays the percentage of your computer's RAM (memory) that specific process is hoarding.
--sort=-%mem (The minus sign - tells the system to sort in descending order, showing the biggest memory hogs first)
- `pgrep processname` – Searches for a process by name and return its PID.  only returns raw numbers. To make it print the process name next to the numbers, use the -l flag
```bash
pgrep -l ssh
1105 sshd
2931 ssh-agent
``` 
- `pidof processname` – Find the PID of a running program. It prints just the raw numbers, separated by spaces, on a single line, usually in reverse chronological order
```bash
pidof chrome 
4115 4109 4102 
```  

### 🛑 Controlling & Terminating Processes
*   `kill PID` – Sends a friendly request to a process asking it to close gracefully (**SIGTERM**).
*   `pkill processname` – Targets and requests a process to close using its name instead of a number.
*   `kill -9 PID` – Forces a stubborn or frozen process to shut down immediately (**SIGKILL**).
*   `pkill -9 processname` – Forcefully wipes out all running instances matching a specific name.
*   `kill -STOP PID` – Pauses/freezes a running process in its tracks without closing it.
*   `kill -CONT PID` – Wakes up and resumes a paused process right where it left off.
- `renice -n 10 -p PID` – Lower priority of a process
- `renice -n -5 -p PID` – Increase priority of a process (requires root)


### Linux Jobs — Summary

A **Process** is an individual program execution tracked globally by the operating system using a unique **PID**. In contrast, a **Job** is a command or pipeline managed by your current terminal shell.

A single job can contain **one or multiple processes** linked together.

---

### 📋 Core Concept Cheat Sheet

* **`jobs`:** Lists the jobs managed by your current terminal shell session, along with their current state (`Running`, `Stopped`, etc.).
* **`[1]`, `[2]`:** Job reference numbers assigned by the shell. You can target a specific job by prefixing its number with `%` (e.g., `%1` targets job 1).
* **`+` and `-`:** Indicators shown in `jobs` output. `+` marks the current/default job, while `-` marks the previous job.
* **`Ctrl + Z`:** Suspends the active foreground job.
* **`fg %1`:** Brings job 1 to the foreground.
* **`bg %1`:** Resumes a stopped job in the background.
* **`kill PID`:** Sends a signal to one individual process using its PID.
* **`kill %1`:** Sends a signal to the process group associated with job 1.
* **`kill -STOP PID`:** Suspends a single process. `jobs` only shows it as a job if it belongs to a job managed by the current shell.
* **`ps` vs `jobs`:** `ps` displays information about processes; `jobs` displays the jobs managed by your current shell.

---

### 🎯 Example: One Job With Multiple Processes

When you use pipes (`|`), you can connect multiple commands together. The shell treats the entire pipeline as **one job**, while each command in the pipeline normally runs as a separate process.

For example:

```bash
cat application.log | grep "ERROR" | sort &
```

The `&` sends the **entire pipeline to the background**.

The shell treats it as one job containing three processes:

```text
📦 Shell Job [%1]
   ├── 🆔 Process 1: cat application.log (Reads the file)
   ├── 🆔 Process 2: grep "ERROR"        (Filters lines)
   └── 🆔 Process 3: sort                (Organizes results)
```

Because these commands usually finish very quickly, you may **not have enough time to see them in the `Running` state**.

Immediately running:

```bash
jobs
```

might show:

```text
[1]+  Running    cat application.log | grep "ERROR" | sort &
```

but it could just as easily show:

```text
[1]+  Done       cat application.log | grep "ERROR" | sort
```

or the job may already have disappeared from the active job list because it has finished.

---

### 💤 A Better Example: `sleep` With Pipes

To clearly see a job containing multiple processes, use commands that stay alive for a while:

```bash
sleep 100 | sleep 200 | sleep 300 &
```

This is **one job** containing three processes:

```text
📦 Shell Job [%1]
   ├── Process 1: sleep 100
   ├── Process 2: sleep 200
   └── Process 3: sleep 300
```

Now, if you immediately run:

```bash
jobs
```

you have plenty of time to see:

```text
[1]+  Running    sleep 100 | sleep 200 | sleep 300 &
```

The job remains active because the `sleep` processes take time to finish.

---

### How to Manage the Job

You can manage the entire job using its job number:

```bash
fg %1
```

Brings the job to the foreground.

```bash
bg %1
```

Resumes a stopped job in the background.

```bash
kill %1
```

Sends a termination signal to the job's process group.

If you need fine-grained control, you can instead target an individual process using its PID.


### Monitoring System Processes
- `top` – Interactive process viewer
- `htop` – User-friendly process viewer (requires installation)
- `nice -n 10 command` – Run a command with a specific priority
- `renice -n -5 -p PID` – Change priority of an existing process

 # What Problem Do `nice` and `renice` Solve?

Imagine your computer has several processes that all want to use the CPU:

```text
Process A  ─┐
Process B  ─┼──> CPU
Process C  ─┤
Process D  ─┘
```

The CPU cannot necessarily run all of them at exactly the same instant, so the Linux scheduler decides which process gets CPU time. 

This is where the **nice value** matters.

---

### What Does "Less CPU Preferred" Actually Mean?

Suppose you have two CPU-intensive processes:
* **Process A** → nice value `0`
* **Process B** → nice value `10`

Both are constantly asking: *"Give me CPU time! I have work to do."*

Generally, the scheduler gives **Process A** more favorable scheduling priority than **Process B** because **A** has the lower nice value.

#### Conceptually (CPU Time Preference):
```text
Process A (nice 0)  ███████████████
Process B (nice 10) ███████
```

So when I say **"less CPU preferred,"** I mean: When processes compete for CPU time, the process with a higher nice value is generally treated as less important by the scheduler and may receive less CPU time.

---

### The Nice Scale

The scale is generally:
* **`-20`** → Highest scheduling priority
* **`0`** → Normal
* **`19`** → Lowest scheduling priority

#### Starting a New Program: `nice`
```bash
nice -n 10 command
```
* **Meaning:** Start this process with a lower scheduling priority than normal.

```bash
sudo nice -n -10 command
```
* **Meaning:** Start this process with a higher scheduling priority than normal.

Because increasing a process's priority can negatively affect other processes, ordinary users are generally restricted from setting negative nice values.

---

### Modifying a Running Program: `renice`

Less commonly used interactively, but still important. It is useful when a process is already running, and you realize you need to change its priority on the fly.

#### Example Scenario:
```text
You start a large job
        ↓
It starts consuming lots of CPU
        ↓
You don't want to stop and restart it
        ↓
Use renice
```


### Daemon Process Management
- `systemctl list-units --type=service` – List all system daemons
- `systemctl start service-name` – Start a daemon/service
- `systemctl stop service-name` – Stop a daemon/service
- `systemctl enable service-name` – Enable a service at startup

⚙️ Daemon & Background Service Management (systemctl)
What is a Daemon?

A daemon (pronounced "demon") is a background process that provides a service, usually without direct interaction from a logged-in user.

Daemons commonly run in the background and wait for events or requests that they need to handle.

Examples:

sshd      → accepts SSH connections
nginx     → serves web requests
cron      → runs scheduled tasks
systemd   → manages system services

In modern Linux distributions, many background services are managed by systemd. You interact with systemd using the systemctl command.

🚦 The Two Dimensions of Service Management

When managing a service with systemctl, there are two separate concepts:

Active State (Right Now) — whether the service is currently running.
Startup State (After Boot) — whether the service is configured to start automatically when the system boots.

These are independent of each other.

| Command | What it controls | Plain-Language Explanation |
| :--- | :--- | :--- |
| systemctl start | Right now | Starts the service immediately. |
| systemctl stop | Right now | Stops the service immediately. |
| systemctl enable | After boot | Configures the service to start automatically during boot. |
| systemctl disable | After boot | Prevents the service from starting automatically during boot. |


For example:

```bash
sudo systemctl start nginx
```

starts Nginx right now, but does not configure it to start after a reboot.

On the other hand:

```bash
sudo systemctl enable nginx
```

configures Nginx to start automatically during boot, but does not necessarily start it immediately.

🔥 Starting/Stopping AND Enabling/Disabling Together

If you want to start a service immediately and also configure it to start automatically at boot, use:

```bash
sudo systemctl enable --now nginx
```

This is effectively:

Start nginx now
        +
Enable nginx for boot

Similarly:

```bash
sudo systemctl disable --now nginx
```

means:

Stop nginx now
        +
Disable automatic startup

These commands are very convenient because they perform both operations at once.

🛠️ Practical Example: Managing an Nginx Web Server

Imagine you have installed the Nginx web server.

1. Checking the Service

You can check the current state of Nginx with:

```bash
systemctl status nginx
```

You might see something like:

```text
● nginx.service
   Loaded: loaded (...)
   Active: active (running)
   Main PID: 1234
```

Important information includes:

Loaded → systemd knows about the service/unit.
Active → shows the current state of the service.
Main PID → the process ID of the main process associated with the service.

For example:

```text
Active: active (running)
```

means Nginx is currently running.

Whereas:

```text
Active: inactive (dead)
```

means Nginx is currently stopped.

And:

```text
Active: failed
```

means the service attempted to start but encountered an error.

2. Start Nginx Immediately
```bash
sudo systemctl start nginx
```

This starts Nginx right now.

It does not change whether Nginx starts automatically after a reboot.

You can verify:

```bash
systemctl status nginx
```

3. Stop Nginx Immediately
```bash
sudo systemctl stop nginx
```

This stops Nginx right now.

The software remains installed on the system.

It also does not change the boot configuration.

4. Restart Nginx

If Nginx is already running and you want to stop and start it again:

```bash
sudo systemctl restart nginx
```

This is commonly used after changing a configuration file.

For example:

```text
Edit configuration
        ↓
Test configuration
        ↓
Restart/reload Nginx
```

🔍 Troubleshooting Services

If a service isn't working, start with:

```bash
systemctl status nginx
```

If the service has failed, you can examine its logs using:

```bash
journalctl -u nginx
```

This displays log messages collected by systemd's journal for the Nginx service.

You can also view recent entries:

```bash
journalctl -u nginx -n 50
```

The -n 50 option shows the most recent 50 log entries.

To follow new log entries as they appear:

```bash
journalctl -u nginx -f
```

This is similar to using:

```bash
tail -f
```

for a regular log file.

# Managing systemd Units with systemctl

The `systemctl` utility is designed specifically to interact with the core **systemd** initialization and service manager. 

* **What it can manage:** It can start, stop, enable, disable, and manage **systemd units**. These are most commonly persistent background network services such as Nginx, SSH (`sshd`), and relational databases, including standard background system daemons and certain specialized one-shot provisioning scripts.
* **What it cannot manage:** It cannot normally monitor or control ordinary desktop applications, ad-hoc shell commands, or stand-alone runtime binaries such as Google Chrome, `sleep`, `ls`, or standard `python` scripts, unless you explicitly create a custom systemd service configuration file (`.service` unit file) for that specific program.

 

## Viewing Process Details


### Changing Process Priority
View process priorities:
```bash
top  # Look at the NI column
```
Change priority of a running process:
```bash
renice -n 10 -p PID  # Lower priority (positive values)
renice -n -5 -p PID  # Higher priority (negative values, root required)
```

### Running Processes in the Background
Run a command in the background:
```bash
command &
```
List background jobs:
```bash
jobs
```
Bring a job to the foreground:
```bash
fg %jobnumber
```
Send a running process to the background:
```bash
Ctrl + Z  # Suspend process
bg %jobnumber  # Resume in background
```

## Monitoring System Processes
### Using `top`
Interactive process viewer:
- Press `k` and enter a PID to kill a process.
- Press `r` to renice a process.
- Press `q` to quit.

### Using `htop`
A user-friendly alternative to `top`:
```bash
htop
```
Allows mouse-based interaction for process management.


## Conclusion
Process management is crucial for system performance and stability. By using tools like `ps`, `top`, `htop`, `kill`, and `nice`, you can efficiently control and monitor Linux processes.
