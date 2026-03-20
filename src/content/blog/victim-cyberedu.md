---
title: "Victim — CyberEdu CTF"
description: "Network forensics: extracting an FTP-transferred encrypted ZIP and recovering credentials from plaintext FTP traffic."
pubDate: 2025-03-15
tags: ["forensics", "network", "wireshark", "cyberedu"]
---

**Category:** Network / Forensics
**Flag:** `ECSC{AC0DFD65CA16813A6AD68C4BA55F8C607496D93E2408EE0B5EF6F1B9ACCE0BC9}`

## Challenge

Analyze a provided `.pcap` file to extract the flag.

## Step 1 — Inspect the traffic

Open `file.pcap` in Wireshark. FTP traffic between two private IPs is visible, indicating a file transfer session.

## Step 2 — Find the transferred file

Filter for data packets from the server:

```
(ip.src == 10.1.10.71 or ip.dst == 10.1.10.71) and data
```

Several files are transferred — one of them is `log.zip`.

## Step 3 — Extract `log.zip`

Use Wireshark's **Follow TCP Stream** on the FTP data connection and save the raw stream as `log.zip`.

## Step 4 — Attempt extraction

```bash
unzip log.zip
# skipping: Flag.txt    unsupported compression method 99
```

Compression method 99 means AES encryption — need a password.

## Step 5 — Find the password

FTP transmits credentials in plaintext. Search the pcap:

```bash
strings file.pcap | grep PASS
# PASS VADPRDqid4TaB0r5a2B0n9wLp
```

## Step 6 — Extract the flag

```bash
7z x log.zip
# Enter password: VADPRDqid4TaB0r5a2B0n9wLp
```

`Flag.txt` contains:

```
ECSC{AC0DFD65CA16813A6AD68C4BA55F8C607496D93E2408EE0B5EF6F1B9ACCE0BC9}
```

## Takeaways

- FTP is plaintext — credentials and files are visible in any packet capture.
- `unzip` doesn't support AES (method 99); use `7z` instead.
- Always check for credential reuse between protocols in the same pcap.
