---
layout: post
hidden: false
title: "Create a 10 Seconds PHP DateInterval "
author: Leonel Elimpe
tags:
  - PHP
  - DateInterval
last_modified_at: 2020-09-30 02:15:41
---
I wanted to test an interval was being respected which required me to add a seconds-based interval value to a [Carbon](https://github.com/briannesbitt/carbon) date object.

For example, if a record is created now, a new one can only be created in say 10 seconds. Here's how I did it thanks to [this comment](https://www.php.net/manual/en/class.dateinterval.php#102930):

```php
<?php

use Carbon\Carbon;

$interval = new DateInterval('PT10S');

$now = Carbon::now();

$nextValidCreationDate = $now->rawAdd($interval);

echo $now->toDateTimeString();
// "2020-09-30 12:42:41"
echo $nextValidCreationDate->toDateTimeString();
// "2020-09-30 12:42:51"

?>
```

Notice 10 seconds was added to `$now`. Also, notice the use of `PT10S` when created the DateInterval, instead of `P10S` which results in an error. It seems when creating time-only DateInterval objects, we have to use `T` too, instead of just `P`. When using the ISO 8601 duration format, T is the time designator that precedes the time components as explained [here](https://www.digi.com/resources/documentation/digidocs/90001437-13/reference/r_iso_8601_duration_format.htm).

* **P** is the duration designator (referred to as "period"), and is always placed at the beginning of the duration.
* **Y** is the year designator that follows the value for the number of years.
* **M** is the month designator that follows the value for the number of months.
* **W** is the week designator that follows the value for the number of weeks.
* **D** is the day designator that follows the value for the number of days.
* **T** is the time designator that precedes the time components.
* **H** is the hour designator that follows the value for the number of hours.
* **M** is the minute designator that follows the value for the number of minutes.
* **S** is the second designator that follows the value for the number of seconds.

For example:

> P3Y6M4DT12H30M5S

Represents a duration of three years, six months, four days, twelve hours, thirty minutes, and five seconds.

<br>

In conclusion, all date-only interval formats must start with `P`, e.g. `P3Y`, `P6M`, `P4D`, `P3Y6M4D`, and all time-only interval formats must start with `P` and `T`, e.g. `PT12H`, `PT30M`, `PT5S`, `PT12H30M5S`.