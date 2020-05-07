---
layout: post
hidden: false
title: 'Laravel: Exclude a Dynamic Route From CSRF Protection'
tags:
  - Laravel
  - CSRF
author: Leonel Elimpe
---
Let's say you have the following dynamic web route and would like to [exclude it](https://laravel.com/docs/6.x/csrf#csrf-excluding-uris) from [csrf protection](https://laravel.com/docs/6.x/csrf) in your Laravel application:

```php
Route::post('access-points/{access_point}/airtime-requests', 'AccessPointAirtimeRequestController');
```

In your `app/Http/Middleware/VerifyCsrfToken.php` file, add the following to the `$except` array to exclude the above route:

```php
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array
     */
    protected $except = [
        'access-points/*/airtime-requests', // <--- ADD THIS
    ];

```

This has worked for me, hope it does for you too 🙂.