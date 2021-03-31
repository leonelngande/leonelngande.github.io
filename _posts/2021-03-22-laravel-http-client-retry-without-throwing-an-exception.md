---
layout: post
hidden: false
title: Laravel HTTP Client Retry Without Throwing an Exception
author: Leonel Elimpe
image:
  path: /images/uploads/laravel-http-retry.svg
  thumbnail: /images/uploads/laravel-http-retry.svg
tags:
  - Laravel
last_modified_at: 2021-03-22 11:31:08
---
I used Laravel's [HTTP Client](https://laravel.com/docs/7.x/http-client) `retry()` feature for the first time yesterday and was surprised it throws an exception on failure of all retries instead of returning the failure `Illuminate\Http\Client\Response` object.

If you make a normal request like:

```php
$response = Http::post(...);
```

You will always be returned an `Illuminate\Http\Client\Response` object, even if a server error or timeout occurs.

However, if you would like you request [retried](https://laravel.com/docs/7.x/http-client#retries) `3` times as below,

```php
$response = Http::retry(3, 100)->post(...);
```

an exception will be thrown if the 3 attempts fail, or a timeout occurs. But I just want the response object returned, and after inspecting the exception thrown after the retries, I noticed the response object is attached to it. So I wrote this:

```php
$response = rescue(function () {

    return Http::retry(3, 100)->post(...);

}, function ($e) {
    
    return $e->response;

});
```

With the above [rescue()](https://laravel.com/docs/7.x/helpers#method-rescue) wrapper around the HTTP request, I always receive a response object and can safely go ahead to verify its status.

```php
$response->successful() : bool;
$response->failed() : bool;
$response->serverError() : bool;
$response->clientError() : bool;
```

Hope this was helpful ;-).