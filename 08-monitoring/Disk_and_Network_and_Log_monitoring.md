

## Disk monitoring

### `df -h` — how full is each disk

Shows how much space is used/free on every mounted disk or partition, in human-readable units (GB/MB instead of raw bytes).

**Real output:**

```text
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   38G   10G  80% /
/dev/sda2       200G  120G   70G  64% /var
```

**Column by column:**

- **`Filesystem`** —  which actual disk/partition device this row is describing, e.g. `/dev/sda1`. Linux names storage devices following a pattern: `sd` means it's being handled through the standard storage driver used for SATA and SCSI-type drives — which covers both traditional spinning hard disks (HDDs) and most SSDs alike, since to Linux they're accessed through the same kind of interface. The name itself doesn't tell you whether it's an HDD or SSD; that's just not information this naming convention carries (you'd check that separately, e.g. with a command like `lsblk -d -o name,rota`, where `rota=1` means spinning HDD and `rota=0` means SSD/no moving parts). The letter after `sd` (`a`, `b`, `c`...) identifies *which* physical disk — `sda` is the first disk Linux found, `sdb` the second, and so on. The number after that (`sda1`, `sda2`...) identifies a *partition* — a subdivided section — on that physical disk; one physical disk can be split into multiple partitions, each acting like its own separate storage area. On cloud servers or systems with multiple drives you'll often see several of these rows, one per disk or per partition on a disk.
  **How is the letter actually decided?** It's assigned in the order the kernel *detects* each disk while the machine boots — not based on a fixed physical slot, and not something you configure directly. This means it can genuinely shift between reboots: adding or removing a disk, or the driver simply initializing devices in a slightly different order, can turn what used to be `sdb` into `sdc` next time. That's exactly why UUIDs (covered under `blkid` further down) are the safer thing to reference in config files instead of the device name.
  **`sd` isn't the only prefix you'll see** — the prefix itself tells you what kind of storage/connection it is: `sda`/`sdb`... covers SATA/SCSI-style devices, which includes traditional hard disks, most SSDs, *and* USB flash drives/external drives (all handled through the same driver, so the name alone can't tell them apart — `lsblk -o name,size,tran` and checking the `tran` column, e.g. `usb` vs `sata`, will). `nvme0n1`, `nvme0n2`... are NVMe SSDs, a newer/faster connection type common in modern machines (partitions look like `nvme0n1p1`, note the extra `p`). `vda`, `vdb`... are virtual disks on KVM/QEMU-based virtual machines. `xvda`, `xvdb`... are virtual disks specifically under Xen-based virtualization. `mmcblk0`... is SD card/eMMC storage, common on small devices like a Raspberry Pi. In practice you don't need to guess which applies — `lsblk` shows you the real device names on that specific machine directly.

- **`Size`** — the total capacity of that filesystem, i.e. how big that partition is in total, regardless of how much is currently used.

- **`Used`** — how much space on that filesystem is currently occupied by files.

- **`Avail`** — how much space is still free to write new data to on that filesystem.

- **`Use%`** — `Used` expressed as a percentage of `Size`, so you can tell how full it is at a glance without doing the math yourself.

- **`Mounted on`** — "mounting" is the process of attaching a disk (or a partition on a disk) to a specific folder path, so that folder becomes the doorway into that disk's storage. Every disk needs to be mounted somewhere before you can actually read or write files on it. `/` (called "root") is the mount point for the main disk holding your core operating system files — it's the very top of the whole folder structure, with everything else nested inside it. A separate row like `/dev/sda2` mounted at `/var` means: this is a genuinely different disk or partition from the one at `/`, and Linux has attached it at the `/var` folder specifically — so anything saved inside `/var` (commonly logs, and sometimes databases) is physically living on that separate disk, with its own separate capacity, completely independent of how full `/` is.

**DevOps read**: this is your "is a disk about to fill up" check. Watch `Use%` especially on `/` (root, where the OS lives) and `/var` (where logs, and sometimes databases, tend to accumulate). A disk hitting 100% can crash services outright — many programs can't even start or log errors once they can't write anything at all. Treat anything consistently above ~85-90% as worth investigating before it becomes an outage.

### `du -sh /path` — what's actually eating the space

Shows the total disk space used by one specific folder (and everything inside it), human-readable.

```text
$ du -sh /var/log
4.2G    /var/log
```

- `-s` = "summary" — give me one total, don't list every single file and subfolder individually.
- `-h` = human-readable size (GB/MB instead of raw bytes).

**DevOps read**: this is the natural follow-up once `df -h` tells you a disk is nearly full but doesn't tell you *why*. Run `du -sh /path/*` (with a wildcard) on a suspect directory's contents to see the size of each subfolder at once, and repeat deeper into whichever one turns out to be the largest, until you land on the actual culprit — often an old log file, an accumulated cache, or forgotten backup files.

### `iostat` — disk I/O statistics, per device

Similar in spirit to vmstat, but focused specifically on each individual disk device rather than the system as a whole.

**Real output:**

```text
$ iostat
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          12.40    0.00    3.10    8.20    0.00   76.30

Device            tps    kB_read/s    kB_wrtn/s   %util
sda              45.20       120.50       980.30   62.10
```

**Top block — `avg-cpu`, column by column:**

- **`%user`** — same idea as `us` in vmstat: percentage of time spent running normal application programs.
- **`%nice`** — like `%user`, but specifically for programs that were deliberately run at a lower priority (a "nice" value was set, meaning "please let others go first").
- **`%system`** — same idea as `sy` in vmstat: percentage of time the kernel itself spent doing work on behalf of programs.
- **`%iowait`** — same idea as `wa` in vmstat: percentage of time the CPU sat idle specifically because it was blocked waiting on disk I/O to finish.
- **`%steal`** — same idea as `st` in vmstat: on a cloud/VM, percentage of time your virtual machine wanted CPU time but a neighboring VM on the same physical host was using it instead.
- **`%idle`** — percentage of time the CPU had nothing to do at all.

**Bottom block — per-device stats, column by column:**

- **`Device`** — the name of the physical disk device this row describes, e.g. `sda` (see the naming explanation under `df -h` above — `sd` just means it's a SATA/SCSI-style drive, which could be a spinning HDD or an SSD; the name alone doesn't tell you which). Unlike `df -h`, `iostat` reports on the whole physical device (`sda`), not individual partitions (`sda1`, `sda2`) — it's showing you the disk's overall workload, not how one particular partition on it is being used. If you have multiple disks you'll see one row per device.

- **`tps`** — "transfers per second": how many individual read or write operations were sent to that disk each second. Higher numbers mean the disk is being asked to do more discrete operations, regardless of how much data each one moves.

- **`kB_read/s`** — kilobytes actually read off that disk per second — the real throughput of data coming from the disk.

- **`kB_wrtn/s`** — kilobytes actually written to that disk per second — the real throughput of data going to the disk.

- **`%util`** — the percentage of time that device was actually busy doing work, out of the total time measured. This is different from throughput (`kB_read/s`/`kB_wrtn/s`) — a disk can have high `%util` even with modest throughput numbers, if the operations themselves are slow (e.g. lots of small random reads/writes rather than a few big sequential ones).

**DevOps read**: `%util` is the headline number — when it's consistently near 100% on a specific disk, that disk is saturated and can't keep up with the requests being sent to it, making it the bottleneck. Cross-reference with `wa`/`%iowait` and `b`/`D`-state processes from vmstat and htop — if all of them are pointing the same direction, you've confirmed disk I/O, not CPU or memory, is the actual problem.


---
 
---
 
## Network monitoring
 
### `ifconfig` — show network interfaces (deprecated, use `ip a`)
 
**Real output:**
 
```text
$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.42  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::a00:27ff:fe4e:66a1  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:4e:66:a1  txqueuelen 1000  (Ethernet)
        RX packets 84213  bytes 98234012 (98.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 61042  bytes 15230441 (15.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
 
**Column by column:**
 
- **`eth0`** — the name of this network interface (your network card, physical or virtual). A machine can have several: `eth0` for a wired connection, `wlan0` for wireless, `lo` for the internal "loopback" interface a machine uses to talk to itself, and so on.
- **`flags`** — the current state of the interface: `UP` means it's turned on and active, `BROADCAST` means it can send to all devices on the local network at once, `RUNNING` means it has an actual live connection, `MULTICAST` means it can send to a specific group of devices at once.
- **`mtu`** — "maximum transmission unit": the largest single chunk of data (in bytes) this interface will send in one go before having to break it into pieces. 1500 is the typical default for Ethernet.
- **`inet`** — the IPv4 address assigned to this interface — this machine's actual address on the network.
- **`netmask`** — defines how much of that address represents "the network" versus "this specific device," which is how the machine figures out which other addresses are on the same local network versus needing to be reached through a router.
- **`broadcast`** — the special address used to send a message to every device on this local network at once.
- **`ether`** — the MAC address: a unique hardware identifier burned into the network card itself, separate from any IP address, and normally not something you need to think about day to day.
- **`RX packets` / `TX packets`** — how many packets (chunks of network data) have been received (`RX`) and transmitted (`TX`) since the interface came up, along with the total bytes.
- **`errors` / `dropped` / `overruns` / `collisions`** — counts of things going wrong at the network hardware level. These should normally sit at, or very close to, zero.
**Why it's deprecated**: `ifconfig` comes from an older Linux toolset (`net-tools`) that isn't actively maintained anymore and doesn't understand newer networking features. `ip a` (from the `iproute2` toolset) is its modern, actively maintained replacement and is what you should reach for instead — most current Linux systems no longer install `ifconfig` by default at all.
 
---
 
### `ip a` — show network interface details (modern replacement)
 
**Real output:**
 
```text
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4e:66:a1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.42/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86234sec preferred_lft 86234sec
```
 
**Column by column:**
 
- **`1:` / `2:`** — a numeric index Linux assigns to each interface internally.
- **`lo` / `eth0`** — the interface name, same idea as in `ifconfig`. `lo` is the loopback interface every machine has for talking to itself (used constantly by local services); `eth0` here is the real network connection.
- **`<UP,LOWER_UP,...>`** — a list of status flags describing this interface's current state, similar in idea to a light switch and the bulb it's connected to being two separate things.
  - `UP` = the switch itself is turned on — the OS has this interface enabled and ready to use.
  - `LOWER_UP` = there's actually a live physical connection underneath — the cable is plugged in and working, or the wifi has actually connected to a network.
  - **Example**: if someone unplugs the network cable, you'd still see `UP` (the OS setting hasn't changed) but `LOWER_UP` would disappear, immediately telling you "this interface is enabled but nothing is actually connected to it" — a very quick way to confirm a cable/connection problem versus a configuration problem.
  - Other flags you might see: `BROADCAST` and `MULTICAST` just mean this interface is capable of those kinds of sends (explained under `ifconfig` above); they're capabilities, not current activity.
- **`mtu`** — same meaning as in `ifconfig`: largest chunk of data sent in one piece.
- **`qdisc`** — the "queueing discipline" — the algorithm the kernel uses to manage and prioritize outgoing traffic on this interface. Not something you typically need to touch.
- **`state`** — `UP` or `DOWN`, a simplified summary of whether this interface is functioning.
- **`link/ether`** — the MAC hardware address, same idea as `ether` in `ifconfig`.
- **`inet 192.168.1.42/24`** — the IPv4 address, but written in CIDR notation: the `/24` is a compact way of expressing the netmask (here, equivalent to `255.255.255.0`) — it tells you how many bits of the address represent the network portion.
- **`brd`** — the broadcast address for this network, same idea as `broadcast` in `ifconfig`.
- **`scope global` / `scope host`** — how far this address is reachable: `global` means reachable from the wider network, `host` means only usable by the machine itself (like the loopback address).
- **`dynamic`** — this address was assigned automatically (typically via DHCP) rather than being manually, permanently configured.
- **`valid_lft` / `preferred_lft`** — "lifetime" values, relevant to DHCP-assigned addresses: how much longer this address remains valid before it needs to be renewed. `forever` (as seen on `lo`) means it never expires.
**DevOps read**: this is your go-to for confirming a machine's actual IP address, whether an interface is really up, and whether it's getting an address via DHCP or a static config — useful first step any time you're troubleshooting "why can't this machine reach the network."
 
---
 
### `netstat -tulnp` — active connections and listening ports (older tool)
 
**Two quick concepts first:**
 
- **A "socket"** is just the technical name for one specific endpoint of a network connection — think of it as one open "phone line": it's a combination of an IP address, a port number, and a protocol (TCP or UDP), that a program has opened up to either wait for connections or talk to something. When you see the word "socket," just read it as "one specific open network connection (or connection point)."
- **"Listening"** means a program has opened a socket and is sitting there waiting for someone else to connect to it — it hasn't connected to anyone yet, it's just ready and waiting. Example: when you start a web server, it "listens" on port 80 — meaning it opens that port and waits for browsers to connect to it. It only becomes an actual active connection once a browser reaches out and connects.
**Real output:**
 
```text
$ netstat -tulnp
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      612/sshd
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      1840/mysqld
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      2210/nginx
udp        0      0 0.0.0.0:68              0.0.0.0:*                           980/dhclient
```
 
**What "listening" means**: a program that's "listening" on a port has opened that port and is sitting there waiting for someone to connect to it — like a receptionist sitting by a phone extension, ready to pick up whenever it rings, but not yet in a call with anyone. It hasn't talked to any specific remote computer yet; it's just available and waiting. The moment someone actually connects, that becomes a separate, active connection — the receptionist is now on a call with one specific caller, while the extension itself might still take other incoming calls too.
 
**What the flags mean**: `-t` = show TCP connections, `-u` = show UDP connections, `-l` = only show ones that are *listening* (as just described, rather than active two-way conversations already in progress), `-n` = show raw IP addresses/port numbers instead of trying to resolve them to hostnames/service names (faster, and avoids confusing lookups), `-p` = show which process (PID and program name) owns each connection.
 
**Column by column:**
 
- **`Proto`** — which protocol this connection uses: `tcp` (a reliable, connection-based protocol, used for things like web traffic, SSH, databases) or `udp` (a faster, connectionless protocol with no guaranteed delivery, used for things like DNS lookups or streaming).
- **`Recv-Q` / `Send-Q`** — how much data is currently sitting in the receive/send queue for this connection, waiting to be processed. Should normally sit at 0; a persistently non-zero value can mean the program isn't reading/sending data fast enough.
- **`Local Address`** — *this machine's* own address and port for this connection — i.e. "me, and which door (port) of mine this row is about." In the example, `0.0.0.0:80` means "port 80, reachable through any of this machine's network interfaces," not tied to one specific IP.
- **`Foreign Address`** — *the other side's* address and port — i.e. "who I'm connected to, and which door of theirs." For a row in `LISTEN` state, there is no "other side" yet, so this just shows a placeholder (`0.0.0.0:*`), meaning "nobody specific yet — open to anyone."
- **`State`** — where this particular row sits in that receptionist analogy. `LISTEN` = the port is open and waiting, nobody connected yet (this is why `Foreign Address` is just a placeholder on these rows). `ESTABLISHED` = an actual two-way connection is live right now, and on that row `Foreign Address` would show the specific real address of whoever it's connected to.
  - **Example**: `tcp 0.0.0.0:80 ... LISTEN` means "nginx is open on port 80, waiting." A moment later, once a visitor's browser actually connects, you'd see a *second*, separate row appear: `tcp 192.168.1.42:80 203.0.113.9:51422 ... ESTABLISHED` — now showing this machine's real address on one side and that specific visitor's address on the other, because that's now a live, ongoing conversation rather than an open, waiting door.
- **`PID/Program name`** — which process, by ID and name, owns this port. This is how you answer "what is actually using port 80 on this machine?"
**DevOps read**: this tells you exactly which services are exposed and on which ports — essential for confirming a service actually started and is listening where you expect, or for tracking down what's occupying a port you need. Note this command typically needs `sudo` to see the `PID/Program name` column fully.
 
---
 
### `ss -tulnp` — modern replacement for `netstat`
 
**Real output:**
 
```text
$ ss -tulnp
Netid  State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port   Process
tcp    LISTEN  0       128      0.0.0.0:22             0.0.0.0:*          users:(("sshd",pid=612,fd=3))
tcp    LISTEN  0       80       127.0.0.1:3306          0.0.0.0:*          users:(("mysqld",pid=1840,fd=21))
tcp    LISTEN  0       511      0.0.0.0:80              0.0.0.0:*          users:(("nginx",pid=2210,fd=6))
```
 
**Column by column:** functionally the same information as `netstat -tulnp`, just laid out slightly differently and gathered directly from the kernel in a faster way:
 
- **`Netid`** — same idea as `Proto` in `netstat`: `tcp` or `udp`.
- **`State`** — same idea as `State` in `netstat` (`LISTEN`, `ESTABLISHED`, etc.), just moved to the second column here instead of near the end.
- **`Recv-Q` / `Send-Q`** — same meaning as in `netstat`.
- **`Local Address:Port`** — this machine's address and port for the connection.
- **`Peer Address:Port`** — the other side's address and port (same idea as `Foreign Address` in `netstat`).
- **`Process`** — which process owns this socket, shown with more detail than `netstat`, including `fd=`.
  - **What `fd` means**: a "file descriptor" is just a small number Linux hands a running program as a label/handle for something it has open — a file, or (as here) a network connection. Linux treats network connections the same way it treats open files internally, so they get numbered the same way. A single program juggling several open files and connections at once tells them apart using these numbers.
  - **Example**: `fd=6` on nginx's listening socket just means "this is the 6th thing nginx currently has open" — not a meaningful number on its own, but useful if you're digging deeper into that exact process (e.g. looking inside `/proc/2210/fd/` to see everything process 2210 currently has open) and need to match this connection to a specific entry there.
**DevOps read**: same flags, same use case as `netstat -tulnp` — checking what's listening on which ports and what owns it. `ss` is faster (it reads directly from the kernel rather than parsing `/proc` files the way `netstat` does) and is the tool actually maintained and recommended today; on many modern systems `netstat` isn't installed by default anymore.
 
---
 
### `ping hostname` — test network connectivity
 
**Real output:**
 
```text
$ ping google.com
PING google.com (142.250.72.14) 56(84) bytes of data.
64 bytes from lhr48s35-in-f14.1e100.net (142.250.72.14): icmp_seq=1 ttl=117 time=11.2 ms
64 bytes from lhr48s35-in-f14.1e100.net (142.250.72.14): icmp_seq=2 ttl=117 time=10.8 ms
64 bytes from lhr48s35-in-f14.1e100.net (142.250.72.14): icmp_seq=3 ttl=117 time=12.1 ms
 
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 10.8/11.36/12.1/0.542 ms
```
 
**Column by column:**
 
- **`64 bytes from ...`** — confirms a reply actually came back, from which address, and how big that reply packet was.
- **`icmp_seq`** — a sequence number, incrementing by one for each ping sent, so you can tell if any went missing (a gap in the sequence means a lost reply).
- **`ttl`** — "time to live": a countdown value that gets reduced by one at every network hop (router) the packet passes through, so the packet doesn't wander the internet forever if something goes wrong. A lower-than-expected `ttl` can hint the reply is coming from further away, or through a different path, than you'd expect.
- **`time`** — how long the round trip took, in milliseconds — how long it took for your ping to reach the target and the reply to come back to you. This is your at-a-glance latency reading.
- **`packets transmitted / received / packet loss`** — the summary at the end: how many pings were sent, how many actually got a reply, and what percentage were lost entirely.
- **`rtt min/avg/max/mdev`** — "round-trip time" statistics across all the pings: the fastest, average, and slowest response times, plus `mdev` (mean deviation) — roughly how much the response times varied from each other. A high `mdev` means the connection is inconsistent/jittery, even if the average looks fine.
**DevOps read**: the very first, simplest check — "can this machine reach that host at all, and how fast?" Any packet loss above 0%, or a high/unstable `mdev`, points to a flaky network path worth investigating further (often with `traceroute`).
 
---
 
### `traceroute hostname` — show the network path to a host
 
**Real output:**
 
```text
$ traceroute google.com
traceroute to google.com (142.250.72.14), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  1.203 ms  1.150 ms  1.099 ms
 2  10.10.0.1 (10.10.0.1)  5.421 ms  5.390 ms  5.310 ms
 3  * * *
 4  108.170.242.1 (108.170.242.1)  10.812 ms  10.701 ms  10.655 ms
 5  142.250.72.14 (142.250.72.14)  11.203 ms  11.150 ms  11.099 ms
```
 
**How the timing actually works — every measurement is a round trip back to *your* machine:**
 
Your machine sends a test message toward the destination, but tells it to stop and report back after just 1 stop along the way. Whoever it hits first sends a reply straight back to you. Then your machine tries again, this time letting the message travel 2 stops before reporting back — and that second stop replies straight back to you too. It keeps going one stop further each time, until the real destination itself finally answers.
 
The key thing to remember: every reply always comes straight back to **your own machine** — never passed along from one stop to the next. So the time shown for stop 5 is really "there and back" all the way to stop 5, not just the little bit of extra distance from stop 4 to stop 5.
 
**Column by column:**
 
- **Hop number (leftmost, `1`, `2`, `3`...)** — each row represents one router along the path, in the order your traffic actually passes through them to reach the target.
- **Hostname/IP address** — which device replied for that hop.
- **Three timing values** — for each stop, the "there and back" test above is repeated three separate times, since timing naturally varies a bit moment to moment. So the three numbers on one row are just three independent measurements of the same round trip, not three different things being measured.
- **`* * *`** — none of the three round trips for that hop got a reply at all — sometimes because that particular router is deliberately configured not to reply to this kind of probe (common, and not automatically a problem), sometimes because there's a genuine block or failure at that point.
**DevOps read**: this is how you find *where* along the path a connectivity or latency problem is actually happening, rather than just knowing *that* one exists (which `ping` tells you). A big jump in response time between two consecutive hops shows you exactly which link introduced the delay. Occasional `* * *` rows are normal and not automatically a red flag; but if everything times out from a certain hop onward and never resolves, that's where the path is actually broken.
 
---
 
### `nslookup domain` — DNS resolution details
 
**Real output:**
 
```text
$ nslookup google.com
Server:         127.0.0.53
Address:        127.0.0.53#53
 
Non-authoritative answer:
Name:   google.com
Address: 142.250.72.14
```
 
**Column by column:**
 
- **`Server` / `Address` (top)** — this is *not* the address of the domain you're looking up — it's the address of the DNS resolver your own machine is asking the question to (often a local resolver, or one provided by your network/ISP). `#53` is the port number, since DNS runs over port 53 by convention.
- **`Non-authoritative answer`** — means this answer came from a resolver's cache or a secondary source, not directly from the domain's own official ("authoritative") DNS server. This is completely normal for everyday lookups — going all the way to the authoritative source for every single request would be unnecessarily slow, so resolvers cache answers and hand out the cached copy.
- **`Name`** — the domain name you actually asked about.
- **`Address` (bottom)** — the actual IP address that domain name resolves to — this is the real answer to "what address does this hostname point to right now?"
**DevOps read**: your go-to when something can reach an IP directly but not by name (or vice versa) — this tells you whether DNS itself is the broken link, and what address a name is actually resolving to, which is invaluable when tracking down issues like a DNS record pointing to a stale/wrong server after a migration.


  ---
 
## Log monitoring
 
### `tail -f /var/log/syslog` — live monitoring of system logs
 
**Real output:**
 
```text
$ tail -f /var/log/syslog
Sep 27 10:12:01 web01 sshd[2231]: Accepted publickey for alex from 203.0.113.5 port 51422
Sep 27 10:12:04 web01 systemd[1]: Started Session 42 of user alex.
Sep 27 10:14:22 web01 nginx[2210]: 203.0.113.5 - - "GET /health HTTP/1.1" 200
Sep 27 10:15:01 web01 CRON[3312]: (root) CMD (/usr/local/bin/backup.sh)
```
 
**What's going on here:** `/var/log/syslog` is just a plain text file where many programs on the machine write a line every time something happens worth recording — a login, a service starting, a request coming in, an error, and so on. `tail` normally just shows you the last few lines of a file and stops. The `-f` flag means "follow" — instead of stopping, it keeps the terminal open and prints each new line the instant it gets written to the file, so you're watching events happen live, in real time, as they occur on the machine.
 
**DevOps read**: this is the simplest way to watch "what's happening on this machine right now" — especially useful while reproducing a problem (e.g. run `tail -f` in one window, then trigger the issue in another, and watch exactly what gets logged the moment it happens).
 
---
 
### `journalctl -f` — live system logs for systemd-based distros
 
**Real output:**
 
```text
$ journalctl -f
Sep 27 10:12:01 web01 sshd[2231]: Accepted publickey for alex from 203.0.113.5 port 51422
Sep 27 10:12:04 web01 systemd[1]: Started Session 42 of user alex.
Sep 27 10:15:10 web01 nginx.service: Reloading nginx configuration...
Sep 27 10:15:11 web01 nginx.service: Reload succeeded.
```
 
**What's going on here:** think of **systemd** as the manager program that runs on most modern Linux machines and is in charge of starting everything else up — when the machine boots, systemd is what actually starts your web server, your database, your networking, and so on, one by one, and keeps track of whether each one is still running, restarting it automatically if it crashes. Each thing systemd manages (like `nginx` or `sshd`) is called a "service."
 
Since systemd is the one starting and supervising all these services anyway, it also collects the log output from every single one of them in one central place, called the **journal** — instead of each service having to write its own separate plain text log file (like `syslog` does), they all funnel into this one shared, unified log that systemd itself manages. `journalctl` is simply the command you use to read that journal — and `-f` means the same "follow" idea as with `tail`: keep the window open and stream new entries live as they happen.
 
The practical upside of this centralized approach: because systemd already knows exactly which service produced each log line, `journalctl` can filter by service name directly (see below) — something plain `tail` on a single shared text file can't do nearly as easily.
 
**DevOps read**: on a systemd-based machine (which is most current Linux distros), this is usually your primary live log view instead of `tail -f /var/log/syslog` — useful extras include `journalctl -u nginx -f` to follow logs for just one specific service by name, which plain `tail` can't easily filter for you.
 
---
 
### `dmesg | tail` — view kernel logs
 
**Real output:**
 
```text
$ dmesg | tail
[   12.402841] eth0: link up, 1000 Mbps, full duplex
[  843.221093] usb 1-2: new high-speed USB device
[ 1204.552210] EXT4-fs (sda1): mounted filesystem with ordered data mode
[ 2011.093812] Out of memory: Killed process 4821 (node) total-vm:3145728kB
```
 
**What's going on here:** `dmesg` shows messages from the **kernel** itself — the core piece of software that directly manages the hardware (CPU, memory, disks, USB devices, network cards) underneath everything else running on the machine. These are lower-level than typical application logs — think hardware being detected, a disk being mounted, or (as in the last line) the kernel forcibly killing a process because the machine ran out of memory. The number in brackets is a timestamp: seconds since the machine booted up, not a calendar date/time. `dmesg` normally dumps the *entire* kernel log buffer at once, which can be long, so piping it through `tail` (using the `|` symbol, which feeds one command's output directly into another command as input) trims it down to just the most recent entries.
 
**DevOps read**: this is where you look for hardware-level or memory-level problems that wouldn't show up in a normal application log — the "Out of memory: Killed process" line above is a real, common example: it's the kernel's own record of forcibly terminating a process because the system ran out of RAM, which is often the actual root-cause explanation behind a service mysteriously dying with no error of its own.
 
---
 
