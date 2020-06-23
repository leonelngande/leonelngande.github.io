---
layout: post
hidden: false
title: Format Dates for Humans with Carbon in PHP
author: Leonel Elimpe
tags:
  - PHP
  - Laravel
last_modified_at: 2020-06-23 11:53:20
---
## Introduction

I've spent like the last 30 minutes searching the web for how to convert dates to human-readable strings with Carbon in PHP/Laravel. Here's a compilation of a couple of useful Carbon methods for my future self.

<br>

## Formatting Examples

```php
<?php

$date = Carbon::now();

echo $date->toDateString();
// 2020-06-22

echo $date->toDateTimeString();
// 2020-06-22 19:45:23

echo $date->toFormattedDateString();
// Jun 22, 2020

echo $date->toTimeString();
// 19:45:23

echo $date->toDayDateTimeString();
// Mon, Jun 22, 2020 7:45 PM

echo Carbon::now()->subDays(5)->diffForHumans();
// 5 days ago

echo Carbon::now()->subDays(24)->diffForHumans();
// 3 weeks ago

echo Carbon::now()->subMonth()->diffForHumans();
// 1 month ago

echo Carbon::create('2020')->longRelativeDiffForHumans('2018');
// 5 months 3 weeks 19 hours 40 minutes 54 seconds ago
```

<br>

## Further Reading

There's plenty more, have a look at the links below.

* [Carbon documentation](https://carbon.nesbot.com/docs/#api-humandiff)
* [Easier Date/Time in Laravel and PHP with Carbon - Scotch.io](https://scotch.io/tutorials/easier-datetime-in-laravel-and-php-with-carbon)