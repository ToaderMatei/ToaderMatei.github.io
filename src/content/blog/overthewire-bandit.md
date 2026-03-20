---
title: "OverTheWire: Bandit — Levels 0-10"
description: "Solutions for the first 10 levels of OverTheWire Bandit, a beginner-friendly wargame focused on Linux fundamentals."
pubDate: 2025-01-10
tags: ["linux", "misc", "overthewire"]
---

OverTheWire Bandit is a wargame that teaches the basics of Linux, file systems, and shell usage. Here are my solutions for levels 0–10.

## Level 0

Connect via SSH and read the readme file.

```
The password you are looking for is: ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If
```

## Level 1

The file is named `-`, which conflicts with shell argument parsing. Use `./` to refer to it explicitly.

```bash
cat ./-
# 263JGJPfgU6LtdEvgfWU1XP5yac29mFx
```

## Level 2

Filename contains spaces — quote it.

```bash
cat "spaces in this filename"
# MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx
```

## Level 3

File is hidden (starts with `.`) inside `inhere/`.

```bash
cd inhere
find .
cat ./...Hiding-From-You
# 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
```

## Level 4

Only one file is human-readable. Check each with `file` or just try them.

```bash
cd inhere
cat ./-file07
# 4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
```

## Level 5

Find a file that is 1033 bytes, readable, and not executable.

```bash
find -type f -size 1033c -readable ! -executable
# ./maybehere07/.file2
cat ./maybehere07/.file2
# HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
```

## Level 6

File is owned by user `bandit7`, group `bandit6`, and is 33 bytes. Search the whole filesystem, redirect errors.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
# morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj
```

## Level 7

Password is in `data.txt` next to the word "millionth".

```bash
grep "millionth" data.txt
# millionth    dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
```

## Level 8

Password is the only line that appears once in `data.txt`.

```bash
sort data.txt | uniq -u
# 4CKMh1JI91bUIZZPXDqGanal4xvAg0JM
```

## Level 9

Password is in `data.txt` as a human-readable string preceded by `==`.

```bash
strings data.txt | grep "=="
# FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey
```

## Level 10

Data is base64 encoded.

```bash
base64 -d data.txt
# The password is dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr
```
