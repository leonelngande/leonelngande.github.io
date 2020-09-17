---
layout: post
hidden: false
title: Storing timespans in a MySQL database
author: Leonel Elimpe
tags:
  - MySQL
last_modified_at: 2020-09-17 05:30:05
---
I've been working on coupon/promotion codes functionality and one requirement is only allowing the re-use of a coupon code after a period of time has passed.

Now how to persist this `period of time` a.k.a `timespan` to the database? After some time Googling I came across [this Stackoverflow answer](https://stackoverflow.com/a/3867631/6924437) which presents the best solution for my use case:

> A MySQL [TIME](http://dev.mysql.com/doc/refman/5.1/en/time.html) type can store time spans from '-838:59:59' to '838:59:59' - if that's within your required range, and it would be useful to have MySQL natively present these spans in HH:MM:SS form, then use that.

The time range supported by the TIME type is well within what I need, which is 30 days which equals 720 hours.

I equally considered the following solutions before going with the above:

- [Laracasts - Store time intervals in database](https://laracasts.com/discuss/channels/laravel/store-time-intervals-in-database?reply=519170)
- [Stackoverflow - Best way to store time interval values in MySQL?](https://stackoverflow.com/a/1540166/6924437)
- [Stackoverflow - What is the way to store “Time only” values with php?](https://stackoverflow.com/a/4604417/6924437)

<br>

Regarding a user interface for inputting this time value, you can have a look at:

- [Stackoverflow - Timefield with more than 24 hours ( timer )](https://stackoverflow.com/a/40127877/6924437)