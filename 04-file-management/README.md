Bilkul\! Hum Linux File Management ko production/practical-focused tarike se karenge. Sirf utni theory jitni command chalane aur real production work samajhne ke liye zaroori hai.

## 1. File & Directory Management

| Command | Kyon use karna hai — practical use |
| --- | --- |
| `ls` | Current folder mein files aur directories dekhne ke liye. |
| `cd /var/log` | Kisi specific directory mein jaane ke liye, jaise production logs. |
| `pwd` | Abhi kis directory mein ho, confirm karne ke liye. |
| `mkdir new_folder` | Naya folder banane ke liye, jaise deployment ya backup folder. |
| `rmdir empty_folder` | Khali directory delete karne ke liye. |
| `rm file.txt` | Unwanted file delete karne ke liye. Production mein pehle filename verify karo. |
| `rm -r folder` | Folder aur uske andar ka data delete karne ke liye. Galat path se data loss ho sakta hai. |
| `cp file1.txt file2.txt` | File ki copy banane ke liye, jaise config ka backup. |
| `cp -r dir1 dir2` | Puri directory ko copy karne ke liye. |
| `mv old_name new_name` | File rename ya file ko doosri location par move karne ke liye. |

### Practical commands

bash
```
pwd
ls -l
cd /var/log
ls
```

- `ls -l` files ki permissions, owner, size aur modification time dikhata hai.
- `/var/log` mein application aur system logs mil sakte hain.
- Production mein delete karne se pehle `pwd` aur `ls` se location verify karo.

Practice ke liye:

bash
```
mkdir linux_practice
cd linux_practice
touch app.log
cp app.log app_backup.log
mv app.log application.log
ls -l
```

`touch` yahan practice file banane ke liye hai; ise bhi yaad rakho.

## 2. File Viewing & Editing

| Command | Kyon use karna hai — practical use |
| --- | --- |
| `cat file.txt` | Chhoti config files ya files ka poora content dekhne ke liye. |
| `tac file.txt` | File ko last line se first line tak dekhne ke liye. |
| `less file.txt` | Badi log files ko scroll karke padhne ke liye. `q` se exit. |
| `more file.txt` | File ko page-by-page padhne ke liye. |
| `head -n 10 file.txt` | File ki pehli 10 lines dekhne ke liye. |
| `tail -n 10 file.txt` | File ki aakhri 10 lines dekhne ke liye. |
| `nano file.txt` | Config ya text file ko simple editor mein edit karne ke liye. |
| `vi file.txt` | Server par available powerful editor se file edit karne ke liye. |
| `echo 'Hello' > file.txt` | File mein text likhne ke liye; purana content overwrite ho jayega. |
| `echo 'Hello' >> file.txt` | File ke end mein text add karne ke liye; purana content rahega. |

### Practical: Production logs check karna

bash
```
cat app.log
less app.log
head -n 10 app.log
tail -n 10 app.log
tail -f app.log
```

- `cat`: poori file ek saath dekhne ke liye.
- `less`: badi file ko safely browse karne ke liye.
- `head`: file ki shuruaat check karne ke liye.
- `tail`: latest log entries check karne ke liye.
- `tail -f`: naye log entries live monitor karne ke liye. `Ctrl+C` se stop.

### Practical: File mein likhna

bash
```
echo 'Application started' > app.log
echo 'Database connected' >> app.log
cat app.log
```

Expected output:

text
```
Application started
Database connected
```

Production rule: `>` existing content mita sakta hai, jabki `>>` content append karta hai. Live production logs par bina samjhe `>` use mat karna.

## 3. Sabse important production practice

| Kaam | Command |
| --- | --- |
| Current location verify karna | `pwd` |
| Files aur permissions dekhna | `ls -l` |
| Large log inspect karna | `less app.log` |
| Latest errors dekhne ki shuruaat | `tail -n 10 app.log` |
| Live logs monitor karna | `tail -f app.log` |
| Config ka backup banana | `cp app.conf app.conf.bak` |
| File mein change se pehle content check karna | `cat app.conf` |
| File mein append karna | `echo 'new entry' >> app.log` |

Yaad rakho: production mein pehle inspect, phir modify, aur delete karne se pehle path double-check.

Aage tumhare liye  Production file-management practical lab ya  Command-based interview practice kar sakta hoon.
