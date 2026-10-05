Bilkul bro. Is baar hum **sirf commands yaad nahi karenge** — ek **enterprise Linux server scenario** bana ke har command practically use karenge.

Flow रहेगा:

```text
Basic
 ↓
Process identify
 ↓
Process control
 ↓
Background/foreground
 ↓
Signals
 ↓
Priority
 ↓
top/htop
 ↓
Services/daemons
 ↓
Production troubleshooting
 ↓
Advanced scenarios
```

**Safety:** `kill -9`, `pkill`, `systemctl stop` hum pehle **lab processes/services** par karenge, random production process par nahi.

---

# LEVEL 0 — Understand your current server

### 1. Kernel/process basics

```bash
ps
```

Then:

```bash
ps aux
```

Then:

```bash
ps -ef
```

Compare:

```bash
ps aux | head
ps -ef | head
```

### 2. Process tree

```bash
pstree
```

If unavailable:

```bash
ps --forest
```

**Goal:** PID + PPID + user + command identify karna.

---

# LEVEL 1 — Find processes

Hum ek safe lab process create karenge:

```bash
sleep 1000 &
```

Ab:

```bash
jobs
```

PID find:

```bash
pgrep sleep
```

Then:

```bash
pidof sleep
```

Then:

```bash
ps -C sleep
```

Then:

```bash
ps aux | grep sleep
```

### Difference practically dekho

```text
pgrep sleep    → PID
pidof sleep    → PID
ps -C sleep    → detailed process information
ps aux         → all processes
```

---

# LEVEL 2 — User ke processes

Pehle current user:

```bash
whoami
```

Then:

```bash
ps -u $(whoami)
```

Ya:

```bash
ps -u raj
```

Enterprise scenario:

> "Raj ke processes kya chal rahe hain?"

```bash
ps -u raj
```

---

# LEVEL 3 — Process details ⭐

`pgrep` se PID lo:

```bash
pgrep sleep
```

Suppose PID:

```text
2456
```

Then:

```bash
ps -p 2456 -f
```

Aur:

```bash
ps -p 2456 -o pid,ppid,user,%cpu,%mem,stat,ni,cmd
```

Ab tum actual process ka:

```text
PID
PPID
USER
CPU
MEMORY
STATE
NICE
COMMAND
```

dekh rahe ho.

---

# LEVEL 4 — Parent/Child Process ⭐

Run:

```bash
sleep 1000 &
```

PID:

```bash
pgrep sleep
```

Then:

```bash
ps -ef | grep sleep
```

PPID dekho.

Then:

```bash
ps -p PID -o pid,ppid,cmd
```

`PID` ko actual number se replace karna.

**Enterprise troubleshooting mein PPID bahut important hai**, because process kis parent/service se create hua hai ye identify karna padta hai.

---

# LEVEL 5 — Graceful process termination

Lab process:

```bash
sleep 1000 &
```

PID:

```bash
pgrep sleep
```

Normal terminate:

```bash
kill PID
```

Check:

```bash
pgrep sleep
```

Agar output nahi aaya → process terminate.

### Important

```bash
kill PID
```

normally **SIGTERM (15)** bhejta hai.

Ye process ko cleanup karne ka chance deta hai.

---

# LEVEL 6 — Force Kill

Again:

```bash
sleep 1000 &
```

Then:

```bash
pgrep sleep
```

Force:

```bash
kill -9 PID
```

Check:

```bash
pgrep sleep
```

Difference:

```text
kill PID
   ↓
SIGTERM
   ↓
"please gracefully exit"

kill -9 PID
   ↓
SIGKILL
   ↓
"immediately terminate"
```

**Production mein `kill -9` first choice nahi hoti.**

---

# LEVEL 7 — pkill

Multiple processes create:

```bash
sleep 1000 &
sleep 1000 &
sleep 1000 &
```

Check:

```bash
pgrep sleep
```

Now:

```bash
pkill sleep
```

Check:

```bash
pgrep sleep
```

All matching `sleep` processes gone.

Then practice:

```bash
pkill -9 sleep
```

But remember:

> `pkill` name ke basis par multiple processes affect kar sakta hai.

---

# LEVEL 8 — STOP / CONT ⭐

Start:

```bash
sleep 1000 &
```

PID:

```bash
pgrep sleep
```

Stop:

```bash
kill -STOP PID
```

Check:

```bash
ps -p PID -o pid,stat,cmd
```

`STAT` mein `T`/stopped state dekho.

Resume:

```bash
kill -CONT PID
```

Check again:

```bash
ps -p PID -o pid,stat,cmd
```

Flow:

```text
RUNNING
   ↓
SIGSTOP
   ↓
STOPPED
   ↓
SIGCONT
   ↓
RUNNING
```

---

# LEVEL 9 — Background / Foreground

Run:

```bash
sleep 300
```

Press:

```text
Ctrl + Z
```

Now:

```bash
jobs
```

Resume background:

```bash
bg %1
```

Check:

```bash
jobs
```

Bring foreground:

```bash
fg %1
```

Then:

```text
Ctrl + C
```

Process terminate.

---

# LEVEL 10 — Multiple Jobs ⭐

Practice:

```bash
sleep 300 &
sleep 400 &
sleep 500 &
```

Then:

```bash
jobs
```

You might see:

```text
[1] Running sleep 300
[2] Running sleep 400
[3] Running sleep 500
```

Bring specific job:

```bash
fg %2
```

Suspend:

```text
Ctrl + Z
```

Resume:

```bash
bg %2
```

This is **shell job control**, not the same thing as system-wide process management.

---

# LEVEL 11 — nice

Start a low-priority process:

```bash
nice -n 10 sleep 1000 &
```

Find it:

```bash
pgrep sleep
```

Check:

```bash
ps -p PID -o pid,ni,cmd
```

You should see:

```text
NI = 10
```

---

# LEVEL 12 — renice ⭐

Create:

```bash
sleep 1000 &
```

Find PID:

```bash
pgrep sleep
```

Change priority:

```bash
renice -n 10 -p PID
```

Check:

```bash
ps -p PID -o pid,ni,cmd
```

Then try:

```bash
sudo renice -n -5 -p PID
```

Check again.

Understand:

```text
NI -20  → highest priority
NI   0  → default
NI +19  → lowest priority
```

---

# LEVEL 13 — top ⭐⭐⭐

Run:

```bash
top
```

Now identify:

```text
PID
USER
PR
NI
%CPU
%MEM
COMMAND
```

Practice:

```text
k → kill process
r → change priority
P → CPU sorting
M → memory sorting
q → exit
```

**Real production scenario:**

> Server CPU = 100%. Which process is consuming CPU?

`top` → `P`

Then investigate that PID.

---

# LEVEL 14 — htop

Check:

```bash
htop
```

If unavailable:

```bash
sudo apt install htop
```

Practice finding:

* CPU-heavy process
* memory-heavy process
* PID
* user
* process tree

---

# LEVEL 15 — Services / Daemons ⭐⭐⭐

Now actual enterprise-style processes.

List services:

```bash
systemctl list-units --type=service
```

Check a service:

```bash
systemctl status ssh
```

Depending on your system it may be:

```bash
systemctl status sshd
```

Find service:

```bash
systemctl list-units --type=service | grep ssh
```

---

# LEVEL 16 — Start / Stop service

Use a **safe service** available on your machine.

First:

```bash
systemctl status SERVICE
```

Then:

```bash
sudo systemctl stop SERVICE
```

Check:

```bash
systemctl status SERVICE
```

Start:

```bash
sudo systemctl start SERVICE
```

Check:

```bash
systemctl status SERVICE
```

**Do not randomly stop critical services like networking, SSH, systemd services on your active machine.**

---

# LEVEL 17 — Enable / Disable ⭐

Check:

```bash
systemctl is-enabled SERVICE
```

Enable startup:

```bash
sudo systemctl enable SERVICE
```

Disable startup:

```bash
sudo systemctl disable SERVICE
```

Understand:

```text
start
 ↓
service runs NOW

enable
 ↓
service starts automatically at BOOT
```

These are **different things**.

---

# LEVEL 18 — Enterprise troubleshooting ⭐⭐⭐

Imagine:

> "Application is down."

First:

```bash
systemctl status app.service
```

Then:

```bash
systemctl is-active app.service
```

Then:

```bash
systemctl is-enabled app.service
```

Find process:

```bash
pgrep -a app
```

or:

```bash
ps aux | grep app
```

Then investigate PID:

```bash
ps -p PID -o pid,ppid,user,%cpu,%mem,stat,ni,cmd
```

Then logs:

```bash
journalctl -u app.service
```

Recent logs:

```bash
journalctl -u app.service -n 50
```

Follow live logs:

```bash
journalctl -u app.service -f
```

This is much closer to **real DevOps/SRE work** than simply practicing `kill`.

---

# LEVEL 19 — Process resource investigation ⭐⭐⭐

CPU:

```bash
ps aux --sort=-%cpu | head
```

Memory:

```bash
ps aux --sort=-%mem | head
```

Top CPU process:

```bash
ps -eo pid,user,%cpu,%mem,stat,cmd --sort=-%cpu | head
```

Top memory:

```bash
ps -eo pid,user,%cpu,%mem,stat,cmd --sort=-%mem | head
```

Now you can answer:

> "Which process is consuming the most CPU?"

and

> "Which process is consuming the most memory?"

---

# LEVEL 20 — Final Enterprise Challenge 🔥

Imagine you're the DevOps engineer on a production server.

### Incident:

```text
CPU = 95%
Application is slow
```

You must investigate **without immediately killing anything**.

Your workflow:

```text
top
 ↓
identify PID
 ↓
ps -p PID -f
 ↓
check CPU/memory
 ↓
check PPID
 ↓
identify service
 ↓
systemctl status service
 ↓
journalctl -u service
 ↓
decide:
   restart?
   priority?
   application issue?
   memory issue?
```

Then only if required:

```bash
kill PID
```

and **`kill -9` only as a last resort**.

---

## 🧠 Commands you will practice

Your complete checklist:

```text
ps
ps aux
ps -ef
ps -u
ps -C
pgrep
pidof
ps -p

kill
kill -9
pkill
pkill -9
kill -STOP
kill -CONT

&
jobs
Ctrl+Z
bg
fg

top
htop

nice
renice

systemctl list-units
systemctl status
systemctl start
systemctl stop
systemctl enable
systemctl disable
systemctl is-active
systemctl is-enabled

journalctl
journalctl -u
journalctl -f

ps --sort
```

**Hum isko ek saath theory nahi karenge.** Tum pehle **LEVEL 1** execute karo:

```bash
sleep 1000 &
pgrep sleep
pidof sleep
ps -C sleep
ps -p $(pgrep sleep) -f
```

**Jo output aaye woh paste karo.** Uske basis par main tumhe next practical challenge dunga.
