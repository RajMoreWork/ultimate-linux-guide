# Linux User Management — Production Practical Guide

Theory minimum rakhenge. Focus rahega left side command, right side command ka use, kyon use karna hai aur production mein kya dhyan rakhna hai.

Example ke liye `appuser` username use karenge. Commands ko test VM/lab mein practice karo; production mein execute karne se pehle impact verify karo.

## 1. User management ki important files

| Command / File            | Kyon use karna hai                                                                           |
| ------------------------- | -------------------------------------------------------------------------------------------- |
| `cat /etc/passwd`         | Existing users aur unki UID, primary GID, home directory, shell dekhne ke liye.              |
| `cat /etc/shadow`         | Password hashes aur password-aging status inspect karne ke liye. Access restricted hota hai. |
| `cat /etc/group`          | Existing groups aur group membership dekhne ke liye.                                         |
| `sudo cat /etc/gshadow`   | Restricted group-admin information dekhne ke liye.                                           |
| `cut -d: -f1 /etc/passwd` | Sirf usernames ki list nikalne ke liye.                                                      |
| `cut -d: -f1 /etc/group`  | Sirf group names ki list nikalne ke liye.                                                    |

Production tip: Password hashes ko bhi sensitive credentials samjho. `/etc/shadow` ka access sirf authorized administrators tak limited rakho. Modern environments mein users LDAP/SSSD jaise central identity systems se bhi aa sakte hain, isliye local files hamesha complete user list nahi hoti.

## 2. Create users

```
sudo useradd appuser
```

Basic user create

Account create karta hai. Home directory distribution defaults par depend karti hai; `-m` use na karne par home automatically banega, ye assume mat karo.

```
sudo useradd -m appuser
```

Home directory ke saath

`/home/appuser` create karne ke liye. Normal human users ke liye useful.

```
sudo useradd -m -s /bin/bash appuser
```

Home + login shell

Bash ko default shell banata hai. Server access policy ke hisaab se shell select karo.

```
sudo adduser appuser
```

Interactive creation

Debian/Ubuntu systems par password aur additional details poochta hai.

```
id appuser
getent passwd appuser
ls -ld /home/appuser
```

Verify user creation

UID, groups, account record aur home directory check karo. `getent` central identity sources ko bhi query kar sakta hai.

Production rule: Application/service account ko human login account ki tarah configure mat karo. Jahan interactive login ki zaroorat nahi hai, wahan suitable non-login shell aur minimum permissions use karo.

## 3. Password management and expiry

```
sudo passwd appuser
```

Password set/reset

Authorized admin user ka password set ya change kar sakta hai. Password terminal ke prompt par enter karo; command mein plaintext password mat likho.

```
sudo chage -M 90 appuser
sudo chage -l appuser
```

Password expiry policy

Maximum password age 90 days set karta hai aur doosri command policy verify karti hai. Isse 90-day rotation enforce karne ki configuration hoti hai; policy exceptions aur warning/grace settings separately review karo.

```
sudo passwd -l appuser
sudo passwd -S appuser
```

Password lock

Local password authentication ko lock karta hai. Ye har existing SSH key ya doosre authentication method ko automatically disable nahi karta.

```
sudo passwd -u appuser
```

Password unlock

Password lock remove karne ke liye. Unlock karne se pehle confirm karo ki account authorized hai aur password state valid hai.

```
sudo cat /etc/shadow
```

Password hash inspect

Stored password hash aur aging fields dekhe ja sakte hain, agar authorized access hai. Hash ko directly decrypt karke original password nikalna normally possible nahi hota; weak passwords guess/crack kiye ja sakte hain, isliye hashes secret rakho.

Production mein yaad rakho: `passwd -l` aur `passwd -u` ko complete account disable/enable mechanism mat samjho. Access revoke karte waqt SSH keys, active sessions, sudo rights, API credentials aur central identity policy bhi check karni pad sakti hai.

## 4. Modify existing users

```
sudo usermod -l newuser olduser
```

Username change

Login name badalta hai. Home directory ka naam aur files automatically rename/move hon, ye assume mat karo.

```
sudo usermod -d /new/home -m appuser
```

Home directory migrate

Home path change karta hai aur `-m` existing content move karta hai. Destination, disk space, ownership, ACLs aur running processes pehle check karo.

```
cat /etc/shells
sudo usermod -s /bin/zsh appuser
```

Default shell change

Login shell update karta hai. Confirm karo ki selected shell installed hai aur allowed hai.

```
id appuser
getent passwd appuser
```

After-change verification

UID, group membership aur account record verify karo. Change ke baad application/service access bhi test karo.

## 5. Delete users safely

```
sudo userdel appuser
```

User account delete

Account remove karta hai; home directory aur files generally preserve ho sakti hain.

```
sudo userdel -r appuser
```

Account + home removal

User aur uski home/mail files remove karne ke liye. Irreversible data loss ho sakta hai.

```
id appuser
find / -xdev -uid 1005 -ls 2>/dev/null
```

Deletion se pehle investigation

`id` account existence check karta hai. `1005` ko actual UID se replace karo; `find` us filesystem par us UID ki owned files locate karta hai. Mounted filesystems aur application data separately inspect karo.

Production checklist: Account owner approval, backup/data retention, scheduled jobs, running processes, SSH access, service dependencies aur orphaned files check kiye bina `userdel -r` mat chalao.

## 6. Group management

```
sudo groupadd developers
```

Group create

Multiple users ko shared access dene ke liye, jaise project files ya deployment directory.

```
sudo usermod -aG developers appuser
```

Supplementary group add

User ko `developers` group mein add karta hai. `-a` zaroori hai; `-G` ko bina `-a` use karne se existing supplementary groups replace ho sakte hain.

```
groups appuser
id appuser
```

Membership verify

User ke primary aur supplementary groups inspect karo. Naye group changes existing login session mein turant reflect na hon.

```
sudo usermod -g developers appuser
```

Primary group change

User ka primary group badalta hai. Iska files ke group ownership aur application behavior par impact ho sakta hai.

```
getent group developers
cut -d: -f1 /etc/group
```

Group verification/list

Particular group ka record ya local group names dekhne ke liye. Central identity systems ke liye `getent` generally better hai.

## 7. Sudo access — production mein high-risk area

```
sudo usermod -aG sudo appuser
```

Debian/Ubuntu

User ko `sudo` group mein add karta hai, agar system ki sudoers policy is group ko authorize karti hai.

```
sudo usermod -aG wheel appuser
```

RHEL-based systems

User ko `wheel` group mein add karta hai, agar system policy group ko sudo rights deti hai.

```
sudo visudo
```

Sudo policy safely edit

Sudoers syntax validate karta hai. Direct text editor se `/etc/sudoers` edit karna avoid karo.

```
appuser ALL=(root) /usr/bin/systemctl status nginx
```

Specific command allow

`/etc/sudoers` ya `/etc/sudoers.d/` ki appropriate policy mein exact command ke liye permission ka example. Actual executable path `command -v systemctl` se verify karo.

```
sudo -l -U appuser
sudo -l
```

Sudo permissions verify

Admin specified user ki allowed commands inspect kar sakta hai; second command current user ke sudo privileges dikhati hai.

`NOPASSWD` caution: Ye configuration password prompt ke bina selected command run karne deti hai:

```
appuser ALL=(root) NOPASSWD: /usr/bin/systemctl status nginx
```

Sirf exact, carefully reviewed commands allow karo. Broad commands, writable scripts, shells ya unsafe arguments privilege escalation ka raasta bana sakte hain. Production mein full `ALL` access ya `NOPASSWD: ALL` tab tak mat do jab tak explicitly approved na ho.

## 8. Production incident mein kaam aane wala quick workflow

Example: New employee ko application server access dena hai.

```
getent passwd appuser
id appuser
```

1\. Existing account check

Duplicate account create karne ya galat user modify karne se bachata hai.

```
sudo useradd -m -s /bin/bash appuser
sudo passwd appuser
```

2\. Approved account create

Sirf tab run karo jab account absent ho aur approval mil chuka ho.

```
sudo usermod -aG developers appuser
id appuser
```

3\. Minimum required access

Sirf approved project group do. Sudo access tabhi jab role ko zaroorat ho.

```
getent passwd appuser
sudo -l -U appuser
```

4\. Final verification

Account details aur effective sudo permissions check karke approved access test karo.

## Production ke 6 golden rules

1. Least privilege: Har user ko sirf required access do.
2. `usermod -aG`: Existing groups preserve karne ke liye `-a` mat bhoolo.
3. Password security: Plaintext passwords commands, scripts ya logs mein mat rakho.
4. Sudo safety: `visudo` use karo aur exact commands authorize karo.
5. Deletion safety: `userdel -r` se pehle data, backups aur dependencies check karo.
6. Verification: Har important change ke baad `id`, `getent`, `groups` aur `sudo -l` jaise commands se result verify karo.
