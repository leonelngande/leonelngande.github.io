---
layout: post
hidden: false
title: "Laravel: Test replacement value is passed to validation string"
author: Leonel Elimpe
tags:
  - Laravel
  - Unit Testing
last_modified_at: 2020-09-13 06:34:47
---
Below is the validation language line we want to test, I added it to the array in `resources/lang/en/validation.php`.

```php
<?php

return [

    // ...
    
    'empty_unless' => 'The :attribute must be empty unless :other is :value.',
    
    // ...

];
```

We want to make sure the replacements values passed in for `:other` and `:value` are included in the validation message. Below are the tests.

```php
<?php

namespace Tests\Unit;

use Tests\TestCase;

class ValidationLanguageLinesTest extends TestCase
{
    // ...

    /** @test */
    public function a_replacement_for__other__can_be_passed_to_the_empty_unless_validation_message()
    {
        /**
         * @see vendor/laravel/framework/src/Illuminate/Foundation/helpers.php at trans()
         */
        $englishTranslation = app('translator')->get(
            'validation.empty_unless',
            ['other' => 'tentative value for OTHER'],
            'en',
            false
        );

        self::assertStringContainsString(
            'tentative value for OTHER',
            $englishTranslation
        );

        $frenchTranslation = app('translator')->get(
            'validation.empty_unless',
            ['other' => 'valeur provisoire pour OTHER'],
            'fr',
            false
        );

        self::assertStringContainsString(
            'valeur provisoire pour OTHER',
            $frenchTranslation
        );
    }

    /** @test */
    public function a_replacement_for__value__can_be_passed_to_the_empty_unless_validation_message()
    {
        $englishTranslation = app('translator')->get(
            'validation.empty_unless',
            ['value' => 'tentative value for VALUE'],
            'en',
            false
        );

        self::assertStringContainsString(
            'tentative value for VALUE',
            $englishTranslation
        );

        $frenchTranslation = app('translator')->get(
            'validation.empty_unless',
            ['value' => 'valeur provisoire pour VALUE'],
            'fr',
            false
        );

        self::assertStringContainsString(
            'valeur provisoire pour VALUE',
            $frenchTranslation
        );
    }
    
    // ...
}
```

If you're wondering about `app('translator')->get()` , I found it by looking at the `trans()` function in `vendor/laravel/framework/src/Illuminate/Foundation/helpers.php`. Here what it looks like at a high level.

```php
<?php

    /**
     * Get the translation for the given key.
     *
     * @param  string  $key
     * @param  array  $replace
     * @param  string|null  $locale
     * @param  bool  $fallback
     * @return string|array
     */
    public function get($key, array $replace = [], $locale = null, $fallback = true)
    {
      // ...
    }
```

Passing in `false` as the fourth parameter, `$fallback`, tells the function not to use a fallback language file if present, which is the behaviour we want in the test. We want it to look for the translation string in the specified translation, `en` or `fr`, and fail if it doesn't find it. 

And there you have it! If you think this approach can be improved in any way, please leave a comment below, will be happy to hear it.