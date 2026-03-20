---
title: "Time-gone — Chronos Security CTF 2025"
description: "Network forensics: decoding a hidden message encoded in NTP Reference ID fields across server response packets."
pubDate: 2025-04-05
tags: ["forensics", "network", "pcap", "scapy", "chronos"]
---

**Category:** Forensics / Network
**Flag:** `CSCTF{sn3@ky_p@cK3Ts}`

## Challenge

A PCAP file containing NTP traffic. Someone encoded a secret message in the **Reference ID** field of NTP server responses.

## NTP header layout (bytes, UDP payload offsets)

| Bytes | Field |
|-------|-------|
| 0 | LI / VN / Mode |
| 1 | Stratum |
| 2 | Poll |
| 3 | Precision |
| 4–7 | Root Delay |
| 8–11 | Root Dispersion |
| **12–15** | **Reference ID** ← encoded data here |
| 16–23 | Reference Timestamp |
| 24–31 | Originate Timestamp |
| 32–39 | Receive Timestamp |
| 40–47 | Transmit Timestamp |

The Reference ID is normally a 4-byte identifier (stratum 1: ASCII clock name; stratum >1: IPv4 address). It's free-form enough to be used as a covert channel.

## Extraction with Scapy

```python
from scapy.all import rdpcap, UDP
import sys

pcap_file = sys.argv[1] if len(sys.argv) > 1 else "ntp_payload.pcap"
pkts = rdpcap(pcap_file)

refid_bytes = []
for p in pkts:
    if UDP in p and bytes(p[UDP].payload):
        if p[UDP].sport == 123 or p[UDP].dport == 123:
            payload = bytes(p[UDP].payload)
            if len(payload) >= 16:
                mode = payload[0] & 0x7
                if mode == 4:  # server responses only
                    refid = payload[12:16]
                    refid_bytes.append(refid)

# Take first byte of each Reference ID in sequence
message = bytes([r[0] for r in refid_bytes]).decode('ascii', errors='replace')
print(f"Collected {len(refid_bytes)} reference IDs")
print(f"Message: {message}")
```

## Alternative — tshark one-liner

```bash
tshark -r ntp_payload.pcap -Y "udp.port == 123 && ntp" \
  -T fields -e ntp.refid
```

## Result

The extracted first bytes produce:

```
GLCSCTF{sn3@ky_p@cK3Ts}GL
```

Strip the `GL` delimiters:

```
CSCTF{sn3@ky_p@cK3Ts}
```

## Takeaways

- Only extract **server responses** (`mode == 4`) — client packets have meaningless Reference IDs.
- Many CTFs add delimiter bytes (here `GL`) around the flag — strip them.
- NTP Reference ID is a classic covert channel because it's visible in plaintext and rarely monitored.
- If characters are encoded in a different byte position (byte 1, 2, or 3), adjust the extraction index.
