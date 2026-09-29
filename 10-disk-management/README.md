# Disk and Storage Management in Linux

## Introduction to Disk and Storage Management
Managing disks and storage efficiently is crucial for system performance and stability. Linux provides various commands to monitor, partition, format, mount, and manage disk storage.

## Index of Commands Covered

### Viewing Disk Information
- `lsblk` – Display block devices
- `fdisk -l` – List disk partitions
- `blkid` – Show UUIDs of devices
- `df -h` – Check disk space usage
- `du -sh /path` – Show size of a directory

### Partition Management
- `fdisk /dev/sdX` – Create and manage partitions
- `parted /dev/sdX` – Alternative to `fdisk` for GPT disks
- `mkfs.ext4 /dev/sdX1` – Format a partition as ext4
- `mkfs.xfs /dev/sdX1` – Format a partition as XFS

### Mounting and Unmounting
- `mount /dev/sdX1 /mnt` – Mount a partition
- `umount /mnt` – Unmount a partition
- `mount -o remount,rw /mnt` – Remount a partition as read-write

### Logical Volume Management (LVM)
- `pvcreate /dev/sdX` – Create a physical volume
- `vgcreate vg_name /dev/sdX` – Create a volume group
- `lvcreate -L 10G -n lv_name vg_name` – Create a logical volume
- `mkfs.ext4 /dev/vg_name/lv_name` – Format an LVM partition
- `mount /dev/vg_name/lv_name /mnt` – Mount an LVM partition

### Swap Management
- `mkswap /dev/sdX` – Create a swap partition
- `swapon /dev/sdX` – Enable swap space
- `swapoff /dev/sdX` – Disable swap space

## Viewing Disk Information
### Using `lsblk`
List all block devices:
```bash
lsblk
```
### Using `fdisk`
View partition details:
```bash
fdisk -l
```
### Using `df`
Check available disk space:
```bash
df -h
```
### Using `du`
Find the size of a directory:
```bash
du -sh /var/log
```

## Partition Management
### Creating a Partition with `fdisk`
```bash
fdisk /dev/sdX
```
Follow the interactive prompts to create a partition.

### Formatting a Partition
Format as ext4:
```bash
mkfs.ext4 /dev/sdX1
```
Format as XFS:
```bash
mkfs.xfs /dev/sdX1
```

## Mounting and Unmounting
### Mount a Partition
```bash
mount /dev/sdX1 /mnt
```
### Unmount a Partition
```bash
umount /mnt
```
### Remount a Partition
```bash
mount -o remount,rw /mnt
```

## LVM Management
### Create a Physical Volume
```bash
pvcreate /dev/sdX
```
### Create a Volume Group
```bash
vgcreate vg_name /dev/sdX
```
### Create a Logical Volume
```bash
lvcreate -L 10G -n lv_name vg_name
```
### Format and Mount the Logical Volume
```bash
mkfs.ext4 /dev/vg_name/lv_name
mount /dev/vg_name/lv_name /mnt
```

## Swap Management
### Create a Swap Partition
```bash
mkswap /dev/sdX
```
### Enable Swap
```bash
swapon /dev/sdX
```
### Disable Swap
```bash
swapoff /dev/sdX
```

## Additional Notes - When to Use fdisk, mount, or Both
### Check Available Disks
Before creating or mounting anything, always check what block devices exist:
```bash
lsblk
```
### Example output:
|NAME | MAJ:MIN| RM| SIZE |RO | TYPE| MOUNTPOINT|
|-----|---------|--|------|---|-----|-----------|
|sda   |  8:0   | 0 | 100G | 0 | disk|           |
|├─sda1|   8:1  | 0 |  96G | 0 | part| /         |
|└─sda2|   8:2  | 0 |  4G  | 0 | part| [SWAP]    |
|sdb   |   8:16 | 0 |  20G | 0 | disk|           |

`sda` → existing disk (already partitioned)

`sdb` → new disk, no partitions yet

### When to use `fdisk`
Use `fdisk` when:
- The disk is brand new and has no partitions
- You want to create `/dev/sdb1`, `/dev/sdb2`, etc.

Inside `fdisk`:
1. Press `n` → create a new partition
2. Press `w` → write changes

Then confirm:
```bash
lsblk
```
### When to Use `mount`
Use `mount` when:
The partition already exists and is formatted
You just want to make it accessible
```bash
sudo mkdir /mnt/mydisk
sudo mount /dev/sdb1 /mnt/mydisk
```
Now your disk is available at `/mnt/mydisk`.

### When to Use fdisk + mount (Full Setup)
Use `fdisk + mkfs + mount` when:
The disk is completely new
You need to partition → format → mount it
```bash
# 1. Check available disks
lsblk
# 2. Create partition
sudo fdisk /dev/sdb
# 3. Format the partition
sudo mkfs.ext4 /dev/sdb1
# 4. Mount it
sudo mkdir /data
sudo mount /dev/sdb1 /data
```
### Quick Reference

| Use Case                     | Command(s)                  |
|-------------------------------|-----------------------------|
| View disks and partitions     | `lsblk`                     |
| Partition a new disk          | `fdisk`                     |
| Mount an existing partition   | `mount`                     |
| Full setup (new disk)         | `fdisk + mkfs + mount`      |


---
 
## Part 8: Disk and storage management
 
Managing disks well matters for both performance and stability — a disk that fills up, or a filesystem that's misconfigured, can crash services outright. This section covers viewing disk info, creating/formatting partitions, and mounting/unmounting them.
 
### Viewing disk information
 
`df -h` (check overall disk space usage) and `du -sh /path` (check the size of a specific folder) were already covered in detail in Part 4 above — the three below are the ones that show you the disks and partitions themselves, before you even get to how full they are.
 
#### `lsblk` — display block devices
 
**Real output:**
 
```text
$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   50G  0 disk
├─sda1        8:1    0   49G  0 part /
└─sda2        8:2    0    1G  0 part [SWAP]
sdb           8:16   0  200G  0 disk
└─sdb1        8:17   0  200G  0 part /var
```
 
**What's a "block device"?** Just the technical name for a storage device that data gets read/written to in fixed-size chunks ("blocks") — in practice, this means your actual disks and their partitions. `lsblk` shows you all of them, in a tree, so you can see at a glance which partitions belong to which physical disk.
 
**Column by column:**
 
- **`NAME`** — the device/partition name, shown as a tree: `sda` is a whole physical disk, and `sda1`/`sda2` (indented underneath, with a `├─`/`└─` connector) are partitions that live on that disk. Same naming pattern covered earlier under `df -h`.
- **`SIZE`** — how big that disk or partition is.
- **`RO`** — whether it's read-only (`1`) or writable (`0`).
- **`TYPE`** — `disk` for a whole physical disk, `part` for a partition on one.
- **`MOUNTPOINT`** — where that partition is currently attached in your folder structure (same "mounting" concept from Part 4) — blank means it isn't mounted anywhere right now, so it's essentially not accessible for normal file access. `[SWAP]` means it's not mounted as a folder at all, it's being used as swap space instead.
**DevOps read**: your fastest "what disks does this machine actually have, and how are they laid out?" check — especially useful before partitioning or formatting anything, so you don't accidentally target the wrong disk.
 
#### `fdisk -l` — list disk partitions
 
**Real output:**
 
```text
$ sudo fdisk -l
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
 
Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048 102758399 102756352   49G 83 Linux
/dev/sda2      102758400 104855551   2097152    1G 82 Linux swap
```
 
**Column by column:**
 
- **`Device`** — the partition name.
- **`Boot`** — an asterisk (`*`) marks the partition the machine boots from.
- **`Start` / `End` / `Sectors`** — where exactly this partition begins and ends on the physical disk, measured in "sectors" (the disk's smallest addressable chunk, 512 bytes each here). Not something you typically need to calculate by hand — `fdisk` handles this for you when creating partitions.
- **`Size`** — the human-readable size, same idea as elsewhere.
- **`Id` / `Type`** — a code identifying what kind of partition this is meant to hold — `83`/`Linux` for a normal Linux filesystem, `82`/`Linux swap` for swap space.
**DevOps read**: similar purpose to `lsblk`, but with more low-level partition detail (exact start/end positions, partition type codes) — useful when you specifically need to inspect or plan partition layout, rather than just get a quick overview. Needs `sudo` since it reads raw disk information.
 
#### `blkid` — show UUIDs of devices
 
**Real output:**
 
```text
$ sudo blkid
/dev/sda1: UUID="a1b2c3d4-5678-90ab-cdef-1234567890ab" TYPE="ext4"
/dev/sda2: UUID="f0e1d2c3-b4a5-6789-0123-456789abcdef" TYPE="swap"
/dev/sdb1: UUID="12ab34cd-56ef-78gh-90ij-klmnopqrstuv" TYPE="xfs"
```
 
**What's a UUID, and why does it matter?** A UUID ("universally unique identifier") is a long, essentially-guaranteed-unique ID string permanently attached to a specific partition when it's formatted. The reason it matters: device names like `/dev/sda1` aren't actually guaranteed to stay the same forever — if you add or remove a disk, or the machine detects hardware in a different order on a reboot, what used to be `sda1` could become `sdb1` instead. A UUID never changes, so critical configuration (like which partition should be mounted at `/var` on every boot) is normally written using the UUID, not the device name, so it keeps working correctly no matter what device name that disk happens to get assigned this time.
 
**Column by column:**
 
- **Device name** (`/dev/sda1`) — which partition this row describes.
- **`UUID`** — that partition's permanent unique identifier.
- **`TYPE`** — the filesystem format that partition was set up with (see `mkfs` below) — `ext4`, `xfs`, `swap`, etc.
**DevOps read**: mainly used when configuring what mounts where at boot time (in a file called `/etc/fstab`) — using UUIDs there instead of device names is the safer, standard practice so mounts don't silently break after a hardware change or reboot.
 
---
 
### Partition management
 
A quick concept first: a **partition** is a way of splitting one physical disk into multiple separate sections, each of which behaves like its own independent storage area — you could, for example, split one 200GB disk into a 50GB partition and a 150GB partition, and use them completely separately.
 
#### `fdisk /dev/sdX` — create and manage partitions
 
Unlike the commands above, `fdisk /dev/sdX` (replace `sdX` with the real disk, e.g. `sda`) drops you into an **interactive** menu inside the terminal, rather than printing one result and exiting.
 
**Real output (interactive session):**
 
```text
$ sudo fdisk /dev/sdb
 
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
Select: p
Partition number (1-4, default 1): 1
First sector: [Enter for default]
Last sector: +50G
 
Created a new partition 1 of type 'Linux' and of size 50 GiB.
 
Command (m for help): w
The partition table has been altered.
```
 
**What's happening, step by step:**
- `n` — create a **n**ew partition.
- `p` — make it a "primary" partition (the standard, straightforward type; older disks allow up to 4 of these).
- Partition number, start sector, end sector — where on the disk this new partition should begin and end; pressing Enter accepts the sensible default, and `+50G` here means "make this partition 50GB, starting from the default position."
- `w` — **w**rite the changes to the disk. Nothing is actually changed until you type `w` — up until then, you can freely experiment and back out. This is deliberate, since partitioning mistakes can destroy data.
**DevOps read**: this only *creates the partition* — it doesn't put a usable filesystem on it yet, that's what `mkfs` (below) is for. `fdisk` works with the older, more traditional "MBR" partitioning style, which has some limitations (like a 2TB size limit per disk, and a max of 4 primary partitions) — for anything larger or more modern, see `parted` below.
 
**What about the other partition types besides `primary`?** When `fdisk` asked `p` for primary above, it was actually offering a choice — MBR-style disks support three types:
 
- **`primary`** — a normal, standalone partition. The one you'll use almost all the time.
- **`extended`** — not a real partition you store files on directly; it's a container that can hold multiple smaller partitions inside it.
- **`logical`** — a partition that lives inside an `extended` one.
**Why the extra complexity exists**: MBR-style disks hard-cap you at 4 primary partitions total, period. If you genuinely needed more than 4 separate partitions on one disk, the workaround was: use 3 slots as normal `primary` partitions, then turn the 4th slot into a single `extended` partition, and carve as many `logical` partitions as you want *inside* that one extended container — effectively sneaking past the 4-partition ceiling.
 
**Do people still use `extended`/`logical` today?** Rarely, by choice. This whole mechanism only exists to work around a limitation specific to the older MBR format. GPT (used by `parted`, covered next) doesn't have a 4-partition limit at all, so on GPT disks you just create as many `primary`-type partitions as you need directly, with no extended/logical dance required. You'll mostly only encounter `extended`/`logical` on older systems or disks still using MBR — worth recognizing if you see it, not something you need to reach for on a modern setup.
 
#### `parted /dev/sdX` — alternative to fdisk for GPT disks
 
**Real output:**
 
```text
$ sudo parted /dev/sdc
GNU Parted 3.4
Using /dev/sdc
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) print
Model: Virtual disk (scsi)
Disk /dev/sdc: 500GB
Partition Table: gpt
 
Number  Start   End    Size   File system  Name     Flags
 1      1049kB  500GB  500GB  ext4         primary
```
 
**What's GPT, and why does it matter?** GPT ("GUID Partition Table") is the newer, modern way disks record their partition layout, replacing the older MBR style `fdisk` traditionally used. GPT removes MBR's limitations — it supports disks larger than 2TB and far more than 4 partitions — which is why `parted` (which understands GPT) is generally preferred over `fdisk` for large modern disks, particularly anything over 2TB.
 
**What's happening, step by step:**
- `mklabel gpt` — initialize the disk with a GPT partition table (needed once, before creating partitions, on a fresh disk).
- `mkpart primary ext4 0% 100%` — create one partition, from the very start (`0%`) to the very end (`100%`) of the disk, i.e. use the whole disk as one partition, and mark it as intended to hold an `ext4` filesystem.
- `print` — show the current partition layout, similar in spirit to `fdisk -l`.
**More realistic example — splitting one disk into multiple partitions with specific sizes:**
 
```text
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 50%
(parted) mkpart primary xfs 50% 100%
(parted) print
Number  Start   End    Size   File system  Name     Flags
 1      1049kB  250GB  250GB  ext4         primary
 2      250GB   500GB  250GB  xfs          primary
```
 
Same `mkpart` command as before, just with different start/end points instead of `0% 100%` — this carves the same 500GB disk into two separate 250GB partitions instead of one, the first formatted as `ext4`, the second as `xfs`. You can also use exact sizes instead of percentages if you prefer, e.g. `mkpart primary ext4 0GB 100GB` for a precise 100GB partition starting from the beginning of the disk.
 
**DevOps read**: reach for `parted` instead of `fdisk` for large (2TB+) disks or when you specifically need GPT — otherwise the two tools accomplish a similar end goal (creating partitions) through different interfaces and underlying partition table formats.
 
#### `mkfs.ext4 /dev/sdX1` and `mkfs.xfs /dev/sdX1` — format a partition
 
A quick concept first: creating a partition just carves out a section of the disk — it doesn't make it usable yet. **Formatting** is the step where you lay down an actual **filesystem** on that partition: the internal structure that keeps track of which files exist, their names, sizes, and where their data physically sits — without a filesystem, a partition is just raw, unorganized space that nothing knows how to read or write files onto.
 
**Real output:**
 
```text
$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 13107200 4k blocks and 3276800 inodes
Allocating group tables: done
Writing inode tables: done
Writing superblocks and filesystem accounting information: done
```
 
```text
$ sudo mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=3276800 blks
data     =                       bsize=4096   blocks=13107200, imaxpct=25
log      =internal log           bsize=4096   blocks=6400, version=2
```
 
**`ext4` vs `xfs`, briefly**: both are just different filesystem formats, each with its own trade-offs — `ext4` is the long-standing, widely used default on most Linux distributions, generally simple and reliable; `xfs` tends to handle very large files and high-performance workloads especially well, and is the default on some enterprise distributions (like RHEL). For most everyday purposes either works fine; the choice mainly matters at larger scale or for specific performance needs.
 
**DevOps read**: this step is destructive — formatting erases whatever was on that partition before. Always double-check you're targeting the right device (`sdb1`, not `sda1`!) before running this; there's no undo.
 
#### What about swap? Do people explicitly set storage aside for it?
 
Yes — recall from earlier (`lsblk`/`blkid`/`fdisk -l`) that `sda2` in our examples showed `TYPE="swap"`. There are two common ways to set swap space aside:
 
- **A dedicated swap partition** — an entire partition, created the same way as any other (via `fdisk`/`parted`), but instead of running `mkfs.ext4` or `mkfs.xfs` on it, you run **`mkswap /dev/sdX2`** to format it specifically as swap, then activate it with **`swapon /dev/sdX2`**. This used to be the standard, default approach.
```text
  $ sudo mkswap /dev/sdb2
  Setting up swapspace version 1, size = 2 GiB
  $ sudo swapon /dev/sdb2
```
 
- **A swap file** — instead of a whole dedicated partition, you create a single regular file of a chosen size (living inside an existing partition, like `/`) and tell Linux to use that file as swap instead. This has become the more common approach on modern systems (including most cloud servers) because it's much easier to resize later — growing a swap file is just making a bigger file; growing a swap partition means repartitioning the disk, which is far more disruptive.
```text
  $ sudo fallocate -l 2G /swapfile
  $ sudo chmod 600 /swapfile
  $ sudo mkswap /swapfile
  $ sudo swapon /swapfile
```
 
**How much swap should you set aside?** There's no single universal rule, but a common rough guideline: on a machine with a modest amount of RAM (say, 2-8GB), swap roughly equal to RAM is reasonable; on machines with a lot of RAM (16GB+), a smaller fixed amount (a few GB) is often enough, since you mainly want swap as a safety cushion for occasional spikes, not as a primary memory substitute — relying on swap heavily as if it were real RAM will make things slow, as covered back in the vmstat section (`si`/`so` activity).
 
**Turning swap back off — `swapoff /dev/sdX`**
 
```text
$ sudo swapoff /dev/sdb2
```
 
No output on success, same as `mount`/`umount`. This deactivates that swap space — the system stops using it, and anything currently sitting in it gets moved back into real RAM first (assuming there's enough free RAM to hold it; if not, this command fails rather than lose data).
 
**DevOps read**: reach for this before removing a swap partition/disk entirely, or before resizing one — you can't safely delete or shrink swap space that's actively in use. Check `free -m` or `swapon --show` beforehand to see how much of that swap is actually occupied right now, so you have a sense of whether there's enough free RAM for `swapoff` to succeed cleanly.
 
#### Made a mistake? How to redo or reverse a partition/format
 
**If you haven't saved yet**: with `fdisk`, nothing is actually changed until you type `w` — so if you're still in the middle of a session and realize you made a mistake, just quit without saving (`q`) and nothing happened at all. `parted` behaves differently and more dangerously here — it applies each command the instant you type it, with no separate "write" step to back out of, so a mistake there is already done the moment you hit Enter.
 
**If you already created the wrong partition**: delete it and recreate it correctly.
 
```text
# in fdisk
Command (m for help): d
Partition number (1,2, default 2): 2
Command (m for help): n
...(create it again, correctly)...
Command (m for help): w
```
 
```text
# in parted
(parted) rm 1
(parted) mkpart primary ext4 0% 100%
```
 
**If you already formatted it with the wrong filesystem type**: you don't need to delete/recreate the partition at all — just run `mkfs` again on it with the correct type. Formatting simply overwrites whatever was there before:
 
```text
$ sudo mkfs.xfs /dev/sdb1
```
replaces whatever filesystem (e.g. `ext4`) was previously on `sdb1`, no deletion step needed first.
 
**Important — this isn't real "undo."** Deleting a partition or reformatting doesn't securely erase data instantly, but it does discard the filesystem's own record of where your files were, so for practical purposes whatever was there is gone unless you had a backup. True recovery means restoring from a backup taken beforehand — `fdisk`/`parted`/`mkfs` themselves don't offer any way to get old data back.
 
---
 
### Mounting and unmounting
 
Recall from Part 4: **mounting** attaches a disk/partition to a folder path so that folder becomes the doorway into that disk's storage. A freshly formatted partition still needs to be mounted before you can actually use it for files.
 
#### `mount /dev/sdX1 /mnt` — mount a partition
 
**Real output:**
 
```text
$ sudo mount /dev/sdb1 /mnt
$
```
 
There's normally no output at all on success — the blank line after the command is the whole result. The partition is now attached: anything you save inside `/mnt` is now physically being written to `/dev/sdb1`. You can confirm it worked with `df -h` or `lsblk`, which would now show `/mnt` listed as that partition's mount point.
 
**DevOps read**: mounting like this is temporary — it won't survive a reboot unless it's also added to `/etc/fstab` (the file listing which partitions should be automatically mounted where, every time the machine starts up), typically referencing the partition by its UUID (from `blkid` above) rather than its device name, for the reasons covered earlier.
 
#### `umount /mnt` — unmount a partition
 
**Real output:**
 
```text
$ sudo umount /mnt
$
```
 
Again, no output on success. This detaches the partition from that folder — the reverse of mounting. Note the command itself is spelled `umount`, not `unmount` (no "n").
 
**DevOps read**: you generally need to unmount a partition before physically removing the disk (e.g. a USB drive) or before certain maintenance operations — a mounted disk that gets yanked out can lead to data corruption, since some writes may not have fully completed yet. If `umount` fails with something like "target is busy," it usually means a program still has a file open on that partition — you'd need to close whatever's using it first.
 
#### `mount -o remount,rw /mnt` — remount a partition as read-write
 
**Real output:**
 
```text
$ sudo mount -o remount,rw /mnt
$
```
 
**What's happening here:** sometimes a partition gets mounted as **read-only** — meaning files can be read from it, but nothing can be written to it — either deliberately (for safety, e.g. a recovery/rescue disk) or because the filesystem detected an error and automatically protected itself by switching to read-only. This command re-mounts an already-mounted partition with different options, without having to fully unmount and remount it from scratch: `-o` means "options," and `remount,rw` means "keep it mounted where it already is, but switch it to read-write."
 
**DevOps read**: useful when you hit a "Read-only file system" error unexpectedly — but treat that error as a symptom worth investigating first, not just something to immediately override. If the filesystem switched itself to read-only automatically, it's often reacting to a detected disk error, and forcing it back to read-write without checking further (e.g. with a filesystem-check tool) can risk making things worse.
 
---
 
## Part 9: Logical Volume Management (LVM)
 
**The problem LVM solves**: a regular partition (everything in Part 8 above) is fixed in size and tied to one specific disk — if you need more space later, you're stuck repartitioning, which is disruptive and risky. LVM adds a flexible layer on top of your physical disks that lets you resize storage, combine multiple disks into one usable pool, and add more space later without the disruption a raw partition resize would involve.
 
LVM works in three layers, each built on top of the last — the commands below create them in order:
 
**1. `pvcreate /dev/sdX` — create a physical volume**
 
```text
$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
```
 
This takes a raw disk (or partition) and marks it as available for LVM to use — think of it as "register this disk as raw material LVM is allowed to draw from." It doesn't create any usable storage yet by itself.
 
**2. `vgcreate vg_name /dev/sdX` — create a volume group**
 
```text
$ sudo vgcreate vg_data /dev/sdc
  Volume group "vg_data" successfully created.
```
 
This pools one or more physical volumes together into a single combined storage pool, given a name of your choosing (`vg_data` here). This is the key trick that makes LVM powerful: you can add several separate physical disks into the same volume group, and from here on, they behave as one combined chunk of available space — you're no longer thinking in terms of "disk A" and "disk B" separately.
 
**3. `lvcreate -L 10G -n lv_name vg_name` — create a logical volume**
 
```text
$ sudo lvcreate -L 10G -n lv_app vg_data
  Logical volume "lv_app" created.
```
 
This is the part that actually behaves like a "partition" you can format and use — carved out of the volume group's combined pool, rather than tied to one specific physical disk. `-L 10G` sets its size (10GB here), `-n lv_app` names it. Critically, this size isn't locked in the way a real partition's is — it can be grown later (using a separate command, `lvextend`) as long as the volume group still has free space, without needing to repartition anything.
 
**4. `mkfs.ext4 /dev/vg_name/lv_name` — format an LVM partition**
 
```text
$ sudo mkfs.ext4 /dev/vg_data/lv_app
Creating filesystem with 2621440 4k blocks and 655360 inodes
```
 
Exact same idea and command as formatting a regular partition (Part 8) — the only difference is the path: `/dev/vg_data/lv_app` refers to your logical volume instead of a raw device like `/dev/sdb1`.
 
**5. `mount /dev/vg_name/lv_name /mnt` — mount an LVM partition**
 
```text
$ sudo mount /dev/vg_data/lv_app /mnt
```
 
Also identical in concept to mounting a regular partition (Part 8) — LVM logical volumes are mounted exactly the same way once formatted, the "logical" layer underneath is invisible to `mount` itself.
 
**Putting the layers together**: physical volume(s) → pooled into a volume group → carved into logical volume(s) → formatted → mounted. The flexibility comes from steps 1-3: you could add a second disk to `vg_data` later (`pvcreate` it, then `vgextend vg_data /dev/sdd`), and now `lv_app` has room to grow into that new space too, all without ever touching the original partitions directly.
 
**After adding that second disk — do you extend `lv_app`, or create a new logical volume?** This is a real choice, not something decided automatically, and it comes down to what you're actually trying to accomplish:
 
- **Extend the existing LV** when you want the *same* volume, same mount point, same data — just bigger, because it's running low on space. This is the far more common reason people add a disk to an existing volume group.
```text
  $ sudo lvextend -L +50G /dev/vg_data/lv_app
  $ sudo resize2fs /dev/vg_data/lv_app
```
  `lvextend` grows the logical volume itself; the filesystem sitting on top of it still needs to be told to grow into that new space too — `resize2fs` for ext4, or `xfs_growfs` for xfs.
 
- **Create a new logical volume** when you want a genuinely separate, independent chunk of storage — a different mount point, a different purpose (e.g. a fresh volume for a new database, separate from `lv_app`).
```text
  $ sudo lvcreate -L 50G -n lv_new_thing vg_data
```
 
The deciding question: is this more space for something that already exists, or a genuinely new, separate thing? Both draw from the same combined pool once the second disk is in the volume group — that's the whole point of pooling them together in the first place.
 
**DevOps read**: reach for LVM (instead of a plain partition) whenever you expect storage needs to grow over time, or want to combine multiple disks into one flexible pool — very common for database servers, application data volumes, or anywhere "we might need more space later" is a realistic expectation. For a small, fixed-purpose disk that will never need to grow, a plain partition is simpler and perfectly fine.
 
