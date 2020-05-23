---
layout: post
hidden: false
title: How to access \App\Http\Kernel class instance in Laravel
author: Leonel Elimpe
tags:
  - Laravel
---
To access an instance of your app's `App\Http\Kernel` class, you can use Laravel's built in [`resolve()`](https://laravel.com/docs/7.x/helpers#method-resolve) helper function:

```php
<?php

$appKernel = resolve(\App\Http\Kernel::class);
```

I use this in a unit test to make sure a middleware is registered in the `$routeMiddleware` array of `\App\Http\Kernel`.

```php
<?php

namespace Tests\Unit;

use App\Http\Kernel;
use App\Http\Middleware\SetLocale;
use Tests\TestCase;

class SetLocaleMiddlewareTest extends TestCase
{
    /** @test */
    public function it_is_correctly_registered_in_route_middleware_array_of_the_http_kernel()
    {
        $routeMiddleware = resolve(Kernel::class)->getRouteMiddleware();

        $this->assertArrayHasKey('setLocale', $routeMiddleware);

        // Assert it's pointing to the correct namespace
        $this->assertEquals(SetLocale::class, $routeMiddleware['setLocale']);
    }
}

```