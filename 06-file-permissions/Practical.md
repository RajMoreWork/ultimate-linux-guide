Bilkul. Agar tum **enterprise-level Linux permissions** practically seekhna chahte ho, to sirf commands yaad nahi karenge. Hum ek **mini-company Linux server lab** banayenge aur har step mein actual access-control problem solve karenge.

Tumhare pasted `06-file-permissions` topics ko base rakhenge, phir **ACLs, SGID, umask, sudo, service accounts, shared directories, troubleshooting** tak jayenge.

## Lab scenario

Hum maanenge company mein:

```text
DevOps Team      → devops
Developers       → developers
QA Team          → qa
```

Users:

```text
dev1
dev2
qa1
devops1
```

Aur company ke projects:

```text
/opt/company/
├── app/
├── qa/
└── devops/
```

---

# LEVEL 1 — Users & Groups

### Step 1 — Groups create karo

```bash
sudo groupadd developers
sudo groupadd qa
sudo groupadd devops
```

Check:

```bash
getent group developers
getent group qa
getent group devops
```

### Step 2 — Users create karo

```bash
sudo useradd -m dev1
sudo useradd -m dev2
sudo useradd -m qa1
sudo useradd -m devops1
```

### Step 3 — Users ko role groups mein add karo

```bash
sudo usermod -aG developers dev1
sudo usermod -aG developers dev2
sudo usermod -aG qa qa1
sudo usermod -aG devops devops1
```

Verify:

```bash
groups dev1
groups dev2
groups qa1
groups devops1
```

---

# LEVEL 2 — Basic Permission Model

Ek project directory:

```bash
sudo mkdir -p /opt/company/app
```

Ownership:

```bash
sudo chown root:developers /opt/company/app
```

Permission:

```bash
sudo chmod 770 /opt/company/app
```

Ab samjho:

```text
root       → owner
developers → group
770        → rwx rwx ---
```

Meaning:

```text
root        → full
developers  → full
others      → no access
```

Test:

```bash
sudo -u dev1 touch /opt/company/app/test.txt
```

Then:

```bash
sudo -u qa1 touch /opt/company/app/qa.txt
```

**Expected:** `dev1` successful, `qa1` permission denied.

Yahi actual Linux access-control practice hai.

---

# LEVEL 3 — Group Ownership

Ab:

```bash
ls -ld /opt/company/app
```

Tumhe roughly:

```text
root developers ...
```

dikhega.

Group change:

```bash
sudo chgrp qa /opt/company/app
```

Ya:

```bash
sudo chown :qa /opt/company/app
```

Verify:

```bash
ls -ld /opt/company/app
```

---

# LEVEL 4 — SGID ⭐ Enterprise Important

Ab important scenario:

**Developers ke shared directory mein jo bhi new file aaye, uska group automatically `developers` hona chahiye.**

```bash
sudo chown root:developers /opt/company/app
sudo chmod 2770 /opt/company/app
```

Notice:

```text
2770
^
SGID
```

Test:

```bash
sudo -u dev1 touch /opt/company/app/file1
```

Check:

```bash
ls -l /opt/company/app/file1
```

File ka group automatically:

```text
developers
```

hona chahiye.

**Ye real shared-team directories mein bahut important concept hai.**

---

# LEVEL 5 — umask

Check:

```bash
umask
```

Example:

```text
0022
```

Test:

```bash
touch testfile
mkdir testdir
ls -ld testfile testdir
```

Phir different umask test karo:

```bash
umask 0002
```

Aur:

```bash
touch test2
mkdir testdir2
ls -ld test2 testdir2
```

Yahan tum practically dekhoge ki **new files/directories ki default permissions kaise decide hoti hain.**

---

# LEVEL 6 — Sticky Bit

Enterprise-style shared directory:

```bash
sudo mkdir /opt/company/shared
sudo chmod 1777 /opt/company/shared
```

```text
1 → sticky bit
777 → everyone can access
```

Test users se files banao:

```bash
sudo -u dev1 touch /opt/company/shared/dev1.txt
sudo -u qa1 touch /opt/company/shared/qa1.txt
```

Ab `qa1` ko `dev1.txt` delete karne ki try karao.

```bash
sudo -u qa1 rm /opt/company/shared/dev1.txt
```

**Permission denied** aana chahiye.

---

# LEVEL 7 — SetUID

Check real Linux example:

```bash
ls -l /usr/bin/passwd
```

Tumhe `s` dikhega:

```text
-rwsr-xr-x
   ^
 SetUID
```

Phir hum khud ek controlled lab example banayenge.

---

# LEVEL 8 — ACL ⭐⭐⭐

Yahan se enterprise-level access control interesting hota hai.

Suppose:

```text
developers → full access
qa         → no access
```

But **ek particular QA employee `qa1` ko exception mein read access dena hai.**

Group structure change nahi karna.

ACL:

```bash
sudo setfacl -m u:qa1:r-- /opt/company/app/file.txt
```

Check:

```bash
getfacl /opt/company/app/file.txt
```

Ye concept **ACL** hai.

---

# LEVEL 9 — Real Company Scenario

Ab hum proper scenario banayenge:

```text
                    /opt/company
                         |
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
         app             qa           devops
          ↓              ↓              ↓
    developers           qa           devops
```

Requirements:

```text
Developers:
    app → read/write

QA:
    app → read-only

DevOps:
    app → full access

Others:
    no access
```

Phir hum dekhenge ki **traditional owner/group permissions alone kaha fail hoti hain** aur ACL kahan required hoti hai.

---

# LEVEL 10 — Permission Troubleshooting ⭐⭐⭐

Ye interview + real production ke liye extremely important hai.

Agar:

```bash
sudo -u dev1 cat /opt/company/app/config.txt
```

fail hua, tum systematically investigate karoge:

```bash
id dev1
groups dev1
ls -l /opt/company/app/config.txt
getfacl /opt/company/app/config.txt
namei -l /opt/company/app/config.txt
```

Especially:

```bash
namei -l /path/to/file
```

se directory-by-directory permission check kar sakte ho.

---

## Tumhara complete progression

```text
1. Users
      ↓
2. Groups
      ↓
3. chmod
      ↓
4. chown
      ↓
5. chgrp
      ↓
6. SGID
      ↓
7. umask
      ↓
8. Sticky Bit
      ↓
9. SetUID
      ↓
10. ACL
      ↓
11. Permission troubleshooting
      ↓
12. Enterprise shared-directory design
```

**Best approach:** ek-ek level practically karo. Main tumhe **command → expected output → tumhare output ka analysis → next challenge** ke format mein le jaunga.

### Start karo Level 1 se:

```bash
sudo groupadd developers
sudo groupadd qa
sudo groupadd devops

getent group developers
getent group qa
getent group devops
```

**Output paste karo.** Phir next step karenge — bina unnecessary theory ke.
