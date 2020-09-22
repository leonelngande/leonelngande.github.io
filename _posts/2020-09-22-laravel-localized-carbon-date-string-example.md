---
layout: post
hidden: false
title: "Laravel: Localized Carbon Date String Example"
author: Leonel Elimpe
tags:
  - Laravel
last_modified_at: 2020-09-22 05:56:44
---
I wanted to return a translated date string from a Carbon date object and after some research, used the below solution which worked for me. It is quite brief, do leave a comment if you've got a question.

```php
<?php

use Carbon\Carbon;


// Create a date object
$date = Carbon::now();

/**
 * Make sure the date translation below is in the app's locale.
 *
 * Format Examples: `D, M j, Y H:i:s` --> mer., oct. 21, 2020 15:11:07,
 *  and `D, M j, Y g:i A` --> mer., oct. 21, 2020 3:26 PM
 *
 * @see \Carbon\Traits\Converter::toDayDateTimeString()
 */
// Get the current app locale
$locale = app()->getLocale();

// Tell Carbon to use the current app locale
Carbon::setlocale($locale);

/**
 * Set the date format you'd like to use.
 * Here, I use two different formats depending on current app locale.
 *
 * For Example: `D, M j, Y H:i:s` outputs `mer., oct. 21, 2020 15:11:07`,
 *  and `D, M j, Y g:i A` outputs `mer., oct. 21, 2020 3:26 PM`
 * If you have a look at the below function 👇🏻 in the Carbon source code,
 * you'll see it uses the first format mentioned above.
 *
 * @see \Carbon\Traits\Converter::toDayDateTimeString()
 */
$format = $locale === 'fr' ? 'D, M j, Y H:i:s' : 'D, M j, Y g:i:s A';

// Use `translatedFormat()` to get a translated date string
$translatedDateString = $date->translatedFormat($format);

// Now do something with the translated date string 🙂 👋🏻
```



### Further Reading:

- [Stackoverflow - Convert AM/PM to 24 hours clock in Laravel input field](https://stackoverflow.com/a/51882771/6924437)
- [Laracasts - Use the localisation of Laravel for Carbon](https://laracasts.com/discuss/channels/laravel/use-the-localisation-of-laravel-for-carbon?reply=591412)