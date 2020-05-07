---
layout: post
hidden: false
title: >-
  Pass Typescript-like Typed Objects and Arrays to Your Laravel Functions by
  Leveraging Data Transfer Objects
tags:
  - Laravel
  - DTOs
  - PHP
  - Typescript
author: Leonel Elimpe
---
When working on a PHP codebase, one thing I really miss from the Typescript world is the ability to pass properly typed objects and array into functions.

Been searching around for a proper solution in PHP and remembered the [DTO](https://stackoverflow.com/a/1058186/6924437) concept today, which led me to Spatie's [data-transfer-object](https://github.com/spatie/data-transfer-object) package.

It perfectly embodies what I've been looking for, only issue being you can't just declare an object is PHP with braces like in Javascript 😁. So instead we create a new namespace in the project under which we keep the DTO files, e.g App\DataTransferObjects.

![](/images/uploads/data-transfer-object-folder.png)

For a use case, I have this service `App\Services\MobileMoneyPaymentService`. It has a method `requestPayment` which sends a POST payment request to a payment server. This method currently looks like this:

```php
<?php

namespace App\Services;

use App\AirtimeRequest;
use Zttp\Zttp;
use Zttp\ZttpResponse;

class MobileMoneyPaymentService
{

    /** @var string $paymentServerUrl */
    protected $paymentServerUrl;

    public function __construct()
    {
        $this->paymentServerUrl = env('PAYMENT_SERVER_URL');
    }

    /**
     * @param AirtimeRequest $airtimeRequest
     * @return ZttpResponse
     */
    public function requestPayment(AirtimeRequest $airtimeRequest): ZttpResponse
    {
        $payload = [
            'amount' => $airtimeRequest->amount,
            'phone' => $airtimeRequest->payer,
            'external_id' => '*****',
            'notification_url' => '*****',
            'reference' => '*****',
            'payer_message' => 'Top up',
            'payee_note' => 'Top up',
        ];

         return Zttp::post("$this->paymentServerUrl/mtn-mobile-money", $payload);
    }
}
```

I would like to refactor `requestPayment` method to receive a `RequestPaymentData` object instead, so let's go ahead and create the `\App\DataTransferObjects\RequestPaymentData` class.

Please note i've already installed the [spatie/data-transfer-object](https://github.com/spatie/data-transfer-object#installation) package via composer with `composer require spatie/data-transfer-object`.

```php
<?php


namespace App\DataTransferObjects;


use Spatie\DataTransferObject\DataTransferObject;

class RequestPaymentData extends DataTransferObject
{
    /** @var integer */
    public $amount;

    /** @var string */
    public $phone;

    /** @var integer */
    public $external_id;
    
    /** @var string */
    public $notification_url;
    
    /** @var string */
    public $reference;
    
    /** @var string */
    public $payer_message;
    
    /** @var string */
    public $payee_note;
    
}
```

Having created the class, we can now refactor the requestPayment method to accept it as follows.

```php
<?php

namespace App\Services;

use App\AirtimeRequest;
use Zttp\Zttp;
use Zttp\ZttpResponse;

class MobileMoneyPaymentService
{

    /** @var string $paymentServerUrl */
    protected $paymentServerUrl;

    public function __construct()
    {
        $this->paymentServerUrl = env('PAYMENT_SERVER_URL');
    }

    /**
     * @param RequestPaymentData $requestPaymentData
     * @return ZttpResponse
     */
    public function requestPayment(RequestPaymentData $requestPaymentData): ZttpResponse
    {
         return Zttp::post("$this->paymentServerUrl/mtn-mobile-money", $requestPaymentData->toArray());
    }
}
```

With this setup, am always sure all the data passed in is actually set and of the correct type. The package throws a `TypeError` if the value doesn't comply with the given type 🙂.

You can learn more by having a look at it's [Readme](https://github.com/spatie/data-transfer-object#data-transfer-objects-with-batteries-included), there's a lot more you can do with it!
