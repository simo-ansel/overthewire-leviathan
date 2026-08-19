# OverTheWire Leviathan — Walkthrough & Notes

[![Status](https://img.shields.io/badge/Status-Completed%20Levels_0%E2%86%927-brightgreen)](#)
[![System](https://img.shields.io/badge/OS-Ubuntu_22.04_LTS-lightgrey)](#)
[![Last Updated](https://img.shields.io/badge/Updated-2025--12--01-blue)](#)

> Practical walkthrough and technical notes for the OverTheWire Leviathan wargame.
> This repository documents the filesystem, binary-analysis and reverse-engineering techniques used to solve levels 0–7.
> Passwords and challenge credentials are redacted.

---

## Introduction

Leviathan is a Linux-based wargame focused on basic reverse engineering, binary analysis and privilege boundaries.

The challenge introduces:

- Hidden files and backups
- Binary inspection
- Static and dynamic analysis
- `strings`
- `ltrace`
- `gdb`
- `strcmp()` analysis
- `system()` and command construction
- Symbolic links
- File permissions
- Hard-coded values
- SUID-style privilege boundaries
- Basic x86 reverse engineering

All exercises were performed in an isolated Linux environment.

---

## Tools

### Remote Access

```text
ssh
```

### Filesystem and File Management

```text
ls
cd
pwd
mktemp
touch
ln
chmod
cat
```

### Text Processing

```text
grep
strings
tr
echo
printf
```

### Binary Analysis

```text
file
strings
ltrace
gdb
```

### Scripting and Conversion

```text
Python 3
printf
```

---

## Methodology

The general workflow used throughout Leviathan was:

1. Enumerate the current environment.
2. Identify executables, hidden files and other potentially relevant artefacts.
3. Inspect binaries statically.
4. Trace their runtime behaviour when necessary.
5. Determine how input reaches security-sensitive functions.
6. Identify permission or trust-boundary weaknesses.
7. Construct the minimal input required to exploit the weakness.
8. Verify the resulting privilege transition.

The challenge is particularly useful for developing a basic reverse-engineering workflow:

```text
enumeration
    ↓
static analysis
    ↓
dynamic analysis
    ↓
identify security boundary
    ↓
construct input
    ↓
verify behaviour
```

---

## Level 0 → 1

### Objective

Locate the password for `leviathan1`.

### Method

Inspect the home directory, including hidden files:

```bash
ls -la
```

A hidden `.backup` directory contains an archived browser bookmark file:

```bash
cd .backup
grep leviathan bookmarks.html
```

The relevant entry contains the password for the next level.

### Takeaway

Backups, configuration files and hidden directories can expose sensitive information even when the primary application interface does not.

During enumeration, do not restrict the search to visible files.

---

## Level 1 → 2

### Objective

Analyse the `check` binary and determine the password required to obtain a shell as `leviathan2`.

### Static Analysis

Inspect readable strings:

```bash
strings ./check
```

Among the output are security-relevant functions and strings such as:

```text
strcmp
setreuid
system
password:
Wrong password, Good Bye ...
```

### Dynamic Analysis

Trace library calls with:

```bash
ltrace ./check
```

The trace shows that the program reads user input with `getchar()` and compares it using `strcmp()`.

The comparison reveals the expected password:

```text
strcmp("input", "sex")
```

### Verification

```bash
./check
```

Enter the recovered password.

The binary opens a shell in the context of `leviathan2`.

The next password can then be read from:

```bash
cat /etc/leviathan_pass/leviathan2
```

### Takeaway

Combining static and dynamic analysis is often more efficient than attempting to fully reverse engineer a small binary.

`strings` reveals useful artefacts, while `ltrace` can expose the values passed to library functions at runtime.

---

## Level 2 → 3

### Objective

Use the `printfile` binary to access a protected file despite unsafe handling of filenames containing spaces.

### Method

Create a temporary working directory:

```bash
mktemp -d
```

Create a file containing a space:

```bash
touch "/tmp/tmp.XXXXXX/test file.txt"
```

Trace the program:

```bash
ltrace ./printfile "/tmp/tmp.XXXXXX/test file.txt"
```

The trace shows that the binary constructs a shell command similar to:

```text
/bin/cat <user-controlled-path>
```

Because the path contains a space, the resulting command is interpreted as multiple arguments.

A symbolic link can be used to control the first argument:

```bash
ln -s /etc/leviathan_pass/leviathan3 /tmp/tmp.XXXXXX/test
```

The vulnerable program can then be invoked using:

```bash
./printfile "/tmp/tmp.XXXXXX/test file.txt"
```

The `cat` invocation resolves `test` through the symlink and accesses the protected file.

### Takeaway

Constructing shell commands from user-controlled paths is unsafe.

Spaces, metacharacters and shell parsing can alter the meaning of an intended file operation. Symbolic links can further redirect filesystem operations to attacker-controlled targets.

---

## Level 3 → 4

### Objective

Determine the password checked by the `level3` binary.

### Method

Run the binary with an incorrect value:

```bash
./level3
```

Then trace the execution:

```bash
ltrace ./level3
```

The trace reveals a call to:

```text
strcmp()
```

and exposes the expected password:

```text
strcmp("input", "snlprintf\n")
```

The correct value can then be supplied to the program:

```bash
./level3
```

The binary opens a shell as `leviathan4`.

Read the next password:

```bash
cat /etc/leviathan_pass/leviathan4
```

### Takeaway

Runtime tracing can reveal sensitive values passed to library functions without requiring a complete static reverse engineering of the binary.

---

## Level 4 → 5

### Objective

Decode a binary representation printed by the `bin` executable.

### Method

Inspect the hidden directory:

```bash
ls -la
cd .trash
ls
```

Execute the program:

```bash
./bin
```

It prints the password as groups of binary digits.

The groups can be converted to ASCII using standard shell tools:

```bash
echo "00110000 01100100 01111001 ..." |
tr ' ' '\n' |
while read b; do
    printf "\\$(printf '%03o' "$((2#$b))")"
done
printf "\n"
```

The resulting ASCII string is the password for `leviathan5`.

### Takeaway

Security analysis frequently involves converting between different data representations.

Understanding binary, hexadecimal and ASCII representations is useful when inspecting binaries, network data and encoded output.

---

## Level 5 → 6

### Objective

Exploit a symbolic link to redirect a file operation performed by the `leviathan5` executable.

### Method

Create a symlink pointing to the protected password file:

```bash
ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
```

Run the provided executable:

```bash
./leviathan5
```

The program follows the symbolic link and reads the protected file.

### Takeaway

Symbolic links can redirect programs that operate on predictable paths.

Programs running with elevated privileges must not blindly trust filesystem paths that can be influenced by lower-privileged users.

---

## Level 6 → 7

### Objective

Reverse engineer `leviathan6` and recover the four-digit code required to obtain a shell as `leviathan7`.

### Initial Analysis

Inspect the binary:

```bash
ls
```

Run it without a valid code:

```bash
./leviathan6 1234
```

The program expects a four-digit argument.

### Static Analysis with GDB

Start GDB:

```bash
gdb --args ./leviathan6 0000
```

Disassemble `main`:

```gdb
disassemble main
```

The relevant instructions include:

```text
movl   $0x1bd3,-0xc(%ebp)
...
call   atoi@plt
...
cmp    %eax,-0xc(%ebp)
```

The binary stores the constant:

```text
0x1bd3
```

and compares it with the integer obtained from the command-line argument through `atoi()`.

Convert the constant to decimal:

```gdb
print/d 0x1bd3
```

Result:

```text
7123
```

### Verification

Run the binary with the recovered value:

```bash
./leviathan6 7123
```

The program opens a shell as `leviathan7`.

Retrieve the next password:

```bash
cat /etc/leviathan_pass/leviathan7
```

### Takeaway

The level demonstrates a basic reverse-engineering workflow:

- identify the input source;
- locate the comparison;
- recover hard-coded constants;
- determine the expected value;
- verify the result dynamically.

GDB provides both static disassembly and runtime inspection capabilities for analysing compiled programs.

---

# Techniques Demonstrated

Leviathan provides practical experience with:

- Linux filesystem enumeration
- Hidden files and backups
- Static binary analysis
- Dynamic library-call tracing
- `strings`
- `ltrace`
- GDB
- Assembly-level inspection
- `strcmp()` analysis
- `system()` command construction
- Symbolic-link exploitation
- File permission analysis
- Hard-coded credential discovery
- Basic privilege-boundary analysis

---

# Key Lessons

The challenge reinforces several important security principles:

- Sensitive data can remain exposed in backups and hidden files.
- Static and dynamic analysis complement each other.
- Runtime tracing can reveal values that are difficult to identify from normal execution.
- Programs should not construct shell commands directly from user-controlled paths.
- Symbolic links can redirect filesystem operations to unintended targets.
- Elevated programs must carefully validate both input and filesystem paths.
- Hard-coded secrets and authentication values can often be recovered through reverse engineering.
- Small binaries can frequently be understood by identifying their input, comparison and privilege-transition logic.

---

# Conclusion

Completing Leviathan 0–7 provides a practical introduction to Linux binary analysis and basic reverse engineering.

The challenge progresses from simple information disclosure to dynamic tracing, symbolic-link manipulation and assembly-level analysis with GDB.

The main workflow developed throughout the challenge is:

```text
Enumerate
    ↓
Identify executable or artefact
    ↓
Inspect statically
    ↓
Trace dynamically
    ↓
Identify input and trust boundaries
    ↓
Determine the security weakness
    ↓
Construct a controlled input
    ↓
Verify the resulting behaviour
```

These techniques form a useful foundation for further work in binary exploitation, reverse engineering and Linux security.

---

# Environment

The walkthrough was originally performed using:

```text
Operating System: Ubuntu 22.04 LTS
Shell: Bash
Architecture: x86_64
```

The challenge binaries and services are provided by OverTheWire.

---

# Disclaimer

This repository documents activity performed against the intentionally vulnerable OverTheWire Leviathan environment.

The techniques described are intended for educational purposes, CTFs, security laboratories and systems for which explicit authorization has been obtained.

Do not apply these techniques to systems without authorization.

---

# Credits

OverTheWire — Leviathan

https://overthewire.org/wargames/leviathan/

All original challenge content and infrastructure belong to OverTheWire and its respective authors.

This repository contains personal notes and walkthrough material created for educational purposes.

---

# Status

Leviathan 0 → 7: Completed
