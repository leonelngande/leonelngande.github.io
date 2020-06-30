---
layout: post
hidden: false
title: How to Set Nullable Types in PHP >= 7.1.x
author: Leonel Elimpe
excerpt: To set a function parameter or return type as nullable, add a question mark in front of the type declaration.
tags:
  - PHP
last_modified_at: 2020-06-30 01:28:05
---
To set a function parameter or return type as nullable, add a question mark in front of the type declaration.

```php
<?php

function testReturn(): ?string
{
    return 'elePHPant';
}

var_dump(testReturn()); // "elePHPant"

function testReturn(): ?string
{
    return null;
}

var_dump(testReturn()); // NULL

function test(?string $name)
{
    var_dump($name);
}

test('elePHPant'); // "elePHPant"
test(null); // NULL
test(); // Uncaught Error: Too few arguments to function test(), 0 passed in...
```

If like me you're confused as to why running the last example - `test()` - throws an error, it's because the function's `$name` parameter being optional does not mean you should not pass in anything at all. From what I've understood, you either pass in a string or null. 

Below is quick way to do this without having to pass in `null` all the time.

```php
<?php

function test(string $name = null)
{
    var_dump($name);
}

test(); // NULL
```



<br>

## Further reading

- [https://www.php.net/manual/en/migration71.new-features.php](https://www.php.net/manual/en/migration71.new-features.php)