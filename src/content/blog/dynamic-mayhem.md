---
title: "Dynamic Mayhem — CTF"
description: "Reversing a Java encryption scheme with a 16-bit random key — brute-forcing all 65536 possible keys to recover the flag."
pubDate: 2025-03-25
tags: ["reverse-engineering", "java", "crypto"]
---

**Category:** Reverse Engineering
**Flag:** `ictf{wHo_DOE$N7_1Ik3_r3v3R$e_3n9iN33r1Ng_brUV_1729_666}`

## Challenge

Two files provided:
- `CTFChallenge.java` — encrypts a flag from `flag.txt`
- `encrypted.txt` — hex-encoded output

## Encryption analysis

The Java program's encryption flow:

```
flag → bytes → P6O5I4(..., R7T8Y9(Q4W5E6(M1N2O3()), M1N2O3())) → hex string
```

Key functions:
- `M1N2O3()` — generates a **16-bit random number** (0–65535) using `SecureRandom`
- `Q4W5E6(int)` — generates a key-dependent byte array
- `R7T8Y9(byte[], int)` — XORs the byte array with the key
- `P6O5I4(...)` — XORs flag bytes with key bytes + applies bitwise/positional transforms
- `X1Y2Z3(...)` — formats output as hex

**Key insight:** The key is only 16 bits → only **65,536 possible values** → fully brute-forceable.

## Decryption approach

1. Read the encrypted hex from `encrypted.txt`
2. Split on the hex separator `00000A` (newline in hex) → 3 segments
3. For each segment, brute-force all 65,536 keys
4. Reverse `P6O5I4` logic for each key
5. Check for readable ASCII / flag format

Segment 0 yielded a base64 string:

```
aWN0Znt3SG9fRE9FJE43XzFJazNfcjN2M1IkZV8zbjlpTj...
```

Decoded:

```
ictf{wHo_DOE$N7_1Ik3_r3v3R$e_3n9iN33r1Ng_brUV_1729_666}
```

## Takeaways

- Obfuscated variable names (`M1N2O3`, `P6O5I4`) are a red herring — focus on what the code _does_.
- A 16-bit key space is trivially brute-forceable (~65K iterations).
- Splitting encrypted data on hex newlines (`00000A`) is a useful trick when data has multiple parts.
- Segment-by-segment encryption is a common CTF pattern.
