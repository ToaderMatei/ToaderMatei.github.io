---
title: "Requests — Chronos Security CTF 2025"
description: "Automating a multi-step API flow: bootstrap token, ticket creation, artifact upload with If-Match header, and flag retrieval."
pubDate: 2025-04-05
tags: ["web", "api", "python", "chronos"]
---

**Category:** Web / API
**Flag:** `CSCTF{0nly_c0UP13_r7que2T2}`

## Challenge

> "Follow our requests, please"

The challenge involves a multi-step API flow: fetch a bootstrap token → create a ticket → read requirements → build artifact + manifest → upload with an `If-Match` header → poll status → retrieve flag.

## Flow

```
/auth/token?flow=bootstrap  →  access_token
/tickets (POST)             →  ticket_id
/tickets/{id}/requirements  →  artifact spec + precondition_value
/upload/{id} (POST)         →  upload artifact + manifest
/tickets/{id}/status        →  poll until verified
/flag?ticket={id}           →  flag
```

## Key details

- The `If-Match` header must equal the `precondition_value` from requirements (SHA256 of the artifact).
- The manifest format: `ticket_id=...`, `artifact_sha256=...`, `artifact_size=...`
- Artifact content is provided as `bytes_exact` in the requirements response.

## Script

```python
import argparse, hashlib, json, time
from pathlib import Path
import requests

DEFAULT_HOST = "http://chals.2025.chronos-security.ro:33143"

def fetch_token(host):
    r = requests.get(f"{host}/auth/token?flow=bootstrap", timeout=8)
    return r.json().get("access_token")

def create_ticket(host, token):
    headers = {"Authorization": f"Bearer {token}"}
    payload = {
        "component": "requests",
        "summary": "Follow our requests, please",
        "request": "5b1028800b7eef6a4e67f8e2b6aa6db8"
    }
    return requests.post(f"{host}/tickets", headers=headers, json=payload).json()

def get_requirements(host, token, tid):
    headers = {"Authorization": f"Bearer {token}"}
    return requests.get(f"{host}/tickets/{tid}/requirements", headers=headers).json()

def build_files(req, tid):
    artifact_val = req.get("artifact", {}).get("bytes_exact", f"{tid}:b66c6b6bf2cb")
    Path("artifact.txt").write_text(artifact_val)
    artifact_bytes = Path("artifact.txt").read_bytes()
    sha256 = hashlib.sha256(artifact_bytes).hexdigest()
    manifest = f"ticket_id={tid}\nartifact_sha256={sha256}\nartifact_size={len(artifact_bytes)}\n"
    Path("manifest.txt").write_text(manifest)
    return Path("artifact.txt"), Path("manifest.txt"), sha256

def upload(host, token, tid, artifact, manifest, if_match):
    headers = {"Authorization": f"Bearer {token}", "If-Match": if_match}
    files = {
        "file": (artifact.name, artifact.open("rb"), "text/plain"),
        "manifest": (manifest.name, manifest.open("rb"), "text/plain"),
    }
    return requests.post(f"{host}/upload/{tid}", headers=headers, files=files)

host = DEFAULT_HOST
token = fetch_token(host)
tid = create_ticket(host, token).get("ticket_id")
req = get_requirements(host, token, tid)
artifact, manifest, sha256 = build_files(req, tid)
precondition = req["upload"]["precondition_value"]

upload(host, token, tid, artifact, manifest, precondition)

# Poll status
for _ in range(30):
    st = requests.get(f"{host}/tickets/{tid}/status",
                      headers={"Authorization": f"Bearer {token}"}).json()
    if st.get("verified"):
        break
    time.sleep(2)

# Get flag
r = requests.get(f"{host}/flag?ticket={tid}",
                 headers={"Authorization": f"Bearer {token}"})
print(r.text)
# {"flag":"CSCTF{0nly_c0UP13_r7que2T2}"}
```

## Takeaways

- Read the API carefully — every step depends on the previous response.
- The `If-Match` header is a standard HTTP precondition mechanism; here it's used as integrity verification.
- Automating multi-step CTF challenges with Python `requests` saves a lot of time vs doing it manually in Burp.
