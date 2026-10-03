# Understanding the Folder Structure

## Basic Commands

```bash
ls -ltr
sudo su -
diif
```

---

## Explanation of System Directories

### Binaries

=> binaries – commnds = admin (sbin) → /usr/sbin, non-admin (bin) /usr/bin

### Symbolic Links (Less Significant)

| Directory | Description |
|-----------|-------------|
| `/sbin -> /usr/sbin` | System binaries for administrative commands generally run with Linux administrator privileges. (linked to `/usr/sbin`). |
| `/bin -> /usr/bin` | Essential user binaries user use in daily life (linked to `/usr/bin`). |
| `/lib -> /usr/lib` | Use by linux kernal not by user , Shared libraries and kernel modules (linked to `/usr/lib`). |

=> shortcut for => sbin for usr/sbin , bin for usr/lib , lib for usr/bin

---

## Important System Directories

| Directory | Description |
|-----------|-------------|
| `/boot` | Stores files needed for booting the system (not relevant in containers). |
| `/usr` | Mainly stores programs, libraries, and other files used by the system and users./usr/bin → user commands/programs
/usr/sbin → administrative commands
/usr/lib → libraries required by programs
/usr/share → shared documentation, configuration-like data, etc.. |
| `/var` | Stores logs, caches, and temporary files that change frequently. |
| `/etc` | Stores system configuration files. |

### `/var`

=> var for logs files

### `/etc`

=> `ls /etc/` = you have a lot of system configuraction file (in windows c:/)

- `/etc/passwd` = by using this we can change password any user of linux \
- `os-release` – informaction about the you OS

---

## User & Application-Specific Directories

| Directory | Description |
|-----------|-------------|
| `/home` | Default location for user home directories. |
| `/opt` | Used for installing optional third-party software. |
| `/srv` | Holds data for services like web servers (rarely used in containers). |
| `/root` | Home directory for the root user. |

### `/srv`

=> srv for server ex: web-server

### `/opt`

=> opt is very important folder in linux environment

- custom tool, executable things, any shall scripts we can place them in `/opt/custom tool`

### `/root`

=> /root = root ki home directory direct /root hi hai

---

## Temporary & Volatile Directories

=> temporory or voatile file or folder – proc,dev,sys,tmp

| Directory | Description |
|-----------|-------------|
| `/tmp` | Temporary files (cleared on reboot). |
| `/run` | Holds runtime data for processes. |
| `/proc` | Virtual filesystem for process and system information. |
| `/sys` | Virtual filesystem for hardware and kernel information. |
| `/dev` | Contains device files (e.g., `/dev/null`, `/dev/sda`). |

### `/run`

=> /run = basically stores the runtime data of the proccess.

---

## Mount Points

| Directory | Description |
|-----------|-------------|
| `/mnt` | Temporary mount point for external filesystems. |
| `/media` | Mount point for removable media (USB, CDs). |
| `/data` | Likely your **mounted volume** from Windows (`C:/ubuntu-data`). |

### `/data`

=> /data =

if I want to share data with other people we can put in this , I have data releted billing informaction like cloud cost

---

## `$PATH`

=> $PATH – jab hum koi bhi command chalate hai tab linux ko kese pata chalta hai ki yehi karna hai = ex. You can check which ls
