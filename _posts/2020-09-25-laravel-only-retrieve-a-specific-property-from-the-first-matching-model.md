---
layout: post
hidden: false
title: "Laravel: Only retrieve a specific property from the first matching model"
author: Leonel Elimpe
tags:
  - Laravel
last_modified_at: 2020-09-25 12:47:33
---
Using Laravel Eloquent, I wanted to only get a specific property - the `id` - from the first matching model of my query.

Here's how I did it:

```php
<?php

use App\Promotion;

$promotionId = Promotion::where('code', 'GOOD_DEAL')->pluck('id')->first();

echo $promotionId // 1
```

<br>

I tried out a good number of eloquent method combinations before arriving at `pluck() + first()`.

![nly retrieve a specific property from the first matching model of a query - try outs](/images/uploads/select-specific-property-from-query.jpg)

With respect to the above screenshot, `GOOD_DEAL_500` is a valid promotion code and `GOOD_DEAL_50` is not.

Of course, I could still just use `Promotion::where('code', 'GOOD_DEAL')->first()->id`, but this prints out:

```
PHP Notice:  Trying to get property 'id' of non-object in Psy Shell code on line 1
```

 if no promotion is found, I wanted to avoid this 🤷🏻‍♂️.