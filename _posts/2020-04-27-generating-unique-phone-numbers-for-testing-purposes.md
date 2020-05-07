---
layout: post
hidden: false
title: Generating Unique Phone Numbers for Testing Purposes
tags:
  - Testing
  - PHP
author: Leonel Elimpe
---
* [Background](#background)
* [PHP Example](#in-php)

## <a name="background"></a>Background

I've decided to document this primarily for future reference.

In an app am working on, I have a database table `access_points` for the model `AccessPoint`. This table has a column phone_number which has to be unique for each access point model. This has been a pain point during testing with respect to generating unique phone numbers to a certain degree of accuracy.

The approach demonstrated below has worked best so far.

## <a name="in-php"></a>PHP Example

```php
<?php

use App\AccessPoint;
use Faker\Generator as Faker;
use Illuminate\Database\Eloquent\Factory;

/** @var Factory $factory */
$factory->define(AccessPoint::class, function (Faker $faker) {

    /**
     * We use anonymous function variable assignment here to assign a function
     * to the variable $randomNumber. What the function does is generate an n-digit number.
     * See below Stackoverflow answer.
     * @see https://stackoverflow.com/a/13169091/6924437
     *
     * @param $length
     * @return string
     *
     * @see https://www.php.net/manual/en/functions.anonymous.php
     */
    $randomNumber = function($length)
    {
        $result = '';

        for($i = 0; $i < $length; $i++) {
            $result .= random_int(0, 9);
        }

        return $result;
    };
  
    /**
     * Set the initial mobile number as a random Cameroonian number.
     * You can use a number from any network operator in your country here.
     */
    $mobileNumber = '+237671231833';
    /**
     * Try generating a unique number by replacing the last 4 digits of the example phone number with 4 randomly
     * generated digits.
     */
    $mobileNumberUnique = substr($mobileNumber, 0, -4) . $randomNumber(4);

    return [
        'phone_number' => $mobileNumberUnique,
    ];
});
```

If you use Laravel you'll notice this is a model factory. You can of course move the logic into a function as necessary for your use case.

I also had to go through the trouble of using an anonymous function variable for I don't have to declare a function that'll be 'hanging' in my codebase. Might clean up the post later, but if you need any clarification, please leave a comment.