---
layout: post
hidden: false
title: Laravel Middleware to Set App Locale to Passed In Parameter Value
author: Leonel Elimpe
tags:
  - Laravel
---
## Introduction

This is a middleware that takes care of setting the app locale to the passed in locale string (e.g `en`, `fr`). 

My use case for this is an app that serves primarily as an API server, and incoming requests do not contain data on the preferred locale. Locale is determined by detecting the language of the text in a request parameter, e.g the text in `$request->description`.

## Creating the middleware

Let's create the middleware with:

```
php artisan make:middleware SetLocale
```

Then open the file (`\App\Http\Middleware\SetLocale`) and replace the content with:

```php
<?php

namespace App\Http\Middleware;

use Closure;

class SetLocale
{
    /**
     * Handle an incoming request.
     *
     * @param \Illuminate\Http\Request $request
     * @param \Closure $next
     * @param string $lang The language key to use in setting the app locale, e.g 'en', 'fr'
     * @return mixed
     */
    public function handle($request, Closure $next, string $lang)
    {
        app()->setLocale($lang);

        return $next($request);
    }
}
```

Note that the `handle()` method, which usually only takes a `$request` and a `$next` closure, has a third parameter, which is our passed in parameter, `$lang`. To learn more about creating parameterized middleware, read [this](https://mattstauffer.com/blog/passing-parameters-to-middleware-in-laravel-5.1/) post by Matt Stauffer, as well as the Laravel [docs](https://laravel.com/docs/6.x/middleware#middleware-parameters).

Next up let's [register](https://laravel.com/docs/6.x/middleware#registering-middleware) the middleware in the `\App\Http\Kernel` class.

> NOTE: If you've never used middleware before, you need to ensure that this middleware is registered in the HTTP Kernel as a `routeMiddleware`—there's no way you could pass parameters to a universal middleware.

```php
<?php

namespace App\Http;

use Illuminate\Foundation\Http\Kernel as HttpKernel;

class Kernel extends HttpKernel
{
    // ...

    protected $routeMiddleware = [
        // ...
      
        'setLocale' => \App\Http\Middleware\SetLocale::class,
    ];

    // ...
}
```

<br>

## Usage Examples

You may assign the middleware to a route:

```php
<?php

Route::post('reviews', 'CreateReviewController')->middleware('setLocale:fr');
```

You may equally use it within a controller:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class CreateReviewController extends Controller
{
    public function __construct()
    {
      //
    }

    /**
     * Handle the incoming request.
     *
     * @return \Response
     */
    public function __invoke(Request $request)
    {
        $lang = 'fr'; // You may add logic to dynamically get the lang key here
        $this->middleware("setLocale:$lang");
        
        // Other controller logic and response follow ..
    }

}

```

## Tests

To test the middleware, let's create a unit test:

```
php artisan make:test SetLocaleMiddlewareTest --unit
```

Then flesh it out as below:

```php
<?php

namespace Tests\Unit;

use App\Http\Kernel;
use App\Http\Middleware\SetLocale;
use Illuminate\Http\Request;
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

    /** @test */
    public function it_correctly_sets_the_app_locale_to_the_passed_in_locale_string()
    {
        $request = new Request;

        $middleware = new SetLocale;

        $middleware->handle($request, function ($req) {

            $this->assertEquals('fr', app()->getLocale());

        }, 'fr');
    }
}
```

For more on Laravel middleware testing, have a look at [this](https://semaphoreci.com/community/tutorials/testing-middleware-in-laravel-with-phpunit) blog post by Christopher Vundi.