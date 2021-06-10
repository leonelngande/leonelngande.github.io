---
layout: post
hidden: false
title: Query Records Created Between Two Dates in Laravel
author: Leonel Elimpe
tags: []
last_modified_at: 2021-06-10 04:56:22
---
I have been trying to query records created between the start of the current month and the current date and time. If the current time is `2021-06-10 16:11:04`, the query should return records created between `2021-05-01` and this time.

Initial attempts using `whereDate()` all gave inaccurate results.

```php
<?php

ModelName::query()
    ->whereDate(
        'created_at',
        '>=',
        (new Carbon('first day of last month'))->toDateString()
    )
    ->whereDate(
        'created_at',
        '<=',
        now()->subMonthNoOverflow()
    )
    ->get();
```

I later found out `whereDate()` converts the date to the format `'Y-m-d'`  so the time is not considered (see below).

```php
<?php

public function whereDate($column, $operator, $value = null, $boolean = 'and')
{
    // ...
    if ($value instanceof DateTimeInterface) {
        $value = $value->format('Y-m-d');
    }
    // ...
}
```

Using `whereBetween()` did the trick though! It worked exactly as wanted it to.

```php
<?php

ModelName::query()
    
    ->whereBetween(
        'created_at',
        [
            (new Carbon('first day of last month'))->toDateString(),
            now()->subMonthNoOverflow()
        ]
    )
    ->get();
```

## Further reading

* [How to query between two dates using Laravel and Eloquent? - Stackoverflow](https://stackoverflow.com/a/33361741/6924437)
* [MySQL SELECT WHERE datetime matches day (and not necessarily time) - Stackoverflow](https://stackoverflow.com/questions/14104304/mysql-select-where-datetime-matches-day-and-not-necessarily-time)
* [whereBetween - Laravel Docs](https://laravel.com/docs/8.x/queries#additional-where-clauses)
* [whereDate - Laravel Docs](https://laravel.com/docs/8.x/queries#additional-where-clauses)