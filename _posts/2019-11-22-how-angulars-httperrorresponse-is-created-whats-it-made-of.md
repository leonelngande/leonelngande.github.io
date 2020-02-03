---
layout: post
hidden: false
published: true
title: 'How Angular''s HttpErrorResponse is Created, What''s It Made Of?'
tags:
  - Angular
  - HttpErrorResponse
author: Leonel Elimpe
---
This is mostly pulled from [this article](https://www.tektutorialshub.com/angular/angular-http-error-handling/) I recently read on [Angular Http Error Handling](https://www.tektutorialshub.com/angular/angular-http-error-handling/), full credits to it's original author [TEKTUTORIALSHUB](https://www.tektutorialshub.com/).

The [HttpClient](https://angular.io/guide/http) captures the errors and wraps it in the generic [HttpErrorResponse](https://angular.io/api/common/http/HttpErrorResponse). The `error` property of the `HttpErrorResponse` contains the underlying `error` object. It also provides additional context about the state of HTTP layer when the error occurred.

The errors falls into two categories. The back end server may generate the error and send the **error response**. Or the client side code may fail to generate the request and throw the error ([ErrorEvent](https://developer.mozilla.org/en-US/docs/Web/API/ErrorEvent) objects).

The server might reject the request for various reasons. Whenever it does it will return the **error response** with the [HTTP Status Codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) such as Unauthorized (401), Forbidden (403), Not found (404), internal Server Error (500) etc. Then Angular assigns the **error response** to `error` property of the `HttpErrorResponse`.

The client side code can also generate the error. The error may be due a network error or an error while executing the HTTP request or an exception thrown in an RxJS operator. These errors produce JavaScript `ErrorEvent objects`. Then Angular assigns the `ErrorEvent object` to error property of the `HttpErrorResponse`.

In both the cases the generic `HttpErrorResponse` is returned, and we can inspect the error property to find out the type of the Error.
