---
title: "Drive Discovery — US Cyber Games (Forensics)"
description: "Disk forensics using FTK Imager to recover a deleted file and extract the flag."
pubDate: 2025-04-10
tags: ["forensics", "disk", "ftk-imager", "uscybergames"]
---

**Category:** Forensics
**Flag:** `SVBR{d3l373d_n07_f0r60773n_283029382}`

## Challenge

A disk image is provided. The flag is hidden in a deleted file.

## Process

1. Open the disk image in **FTK Imager**.
2. Browse the file system — use the "Orphan Files" or deleted files view to find files marked for deletion.
3. Recover the deleted file containing the flag.

## Flag

```
SVBR{d3l373d_n07_f0r60773n_283029382}
```

## Tools

- **FTK Imager** — free disk imaging and forensic analysis tool by AccessData.
- Alternatively: **Autopsy**, **Sleuth Kit (`fls`, `icat`)**, or **PhotoRec** for file carving.

## Takeaways

- Deleted files are recoverable until their sectors are overwritten — forensics relies on this.
- FTK Imager's "Export Files" feature preserves deleted file metadata.
- In CTF forensics, always check the recycle bin, deleted files, and unallocated space.
