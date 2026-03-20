---
title: "The Fragmented Truth — CTF"
description: "Forensics: reassembling fragmented TCP streams from a PCAP to extract hidden PNG images containing the flag."
pubDate: 2025-03-28
tags: ["forensics", "pcap", "python", "scapy"]
---

**Category:** Forensics
**Flag:** `ictf{1n_7h3_s1l3nc3_0f_fr4gm3n75_w3_r3v34l}`

## Challenge

> "In this sea of chaos, can you uncover the signal, where ictf holds the key to rising above the noise in the transmission?"

A PCAP file is provided. PNG images are hidden inside TCP streams, fragmented across multiple packets.

## Step 1 — Inspect the PCAP

In Wireshark, several packets contain partial PNG headers:

```
\x89PNG\r\n\x1a\n ... IDAT ... IEND\xaeB`\x82
```

Images are being transmitted but split across TCP segments — simple raw scanning only finds one.

## Step 2 — Reassemble TCP streams with Scapy

```python
from scapy.all import *
import re
import os
from collections import defaultdict

packets = rdpcap("your_file.pcap")

# Group payloads by TCP stream
streams = defaultdict(bytes)
for pkt in packets:
    if pkt.haslayer(TCP) and pkt.haslayer(Raw):
        ip = pkt[IP]
        tcp = pkt[TCP]
        key = (ip.src, ip.dst, tcp.sport, tcp.dport)
        streams[key] += bytes(pkt[Raw])

# Extract PNG images from each stream
png_start = b'\x89PNG\r\n\x1a\n'
png_end   = b'IEND\xaeB`\x82'

os.makedirs("extracted_pngs", exist_ok=True)
img_count = 0

for stream_data in streams.values():
    for match in re.finditer(png_start + b'.*?' + png_end, stream_data, re.DOTALL):
        img_data = match.group()
        with open(f"extracted_pngs/image_{img_count}.png", "wb") as f:
            f.write(img_data)
        img_count += 1

print(f"Extracted {img_count} images")
```

## Step 3 — Find the flag

Open each extracted PNG. One image contains the flag visually encoded in it.

## Takeaways

- Fragmented data requires stream reassembly — Scapy's grouping by `(src, dst, sport, dport)` is effective.
- Simple raw byte scanning misses fragmented files; always check TCP streams.
- PNG magic bytes (`\x89PNG`) and trailer (`IEND\xaeB\x82`) make reliable extraction anchors.
