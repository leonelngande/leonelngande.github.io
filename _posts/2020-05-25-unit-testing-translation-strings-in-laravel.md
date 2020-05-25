---
layout: post
hidden: false
title: Unit Testing Translation Strings in Laravel
author: Leonel Elimpe
tags:
  - Laravel
  - Unit Testing
---
If your Laravel app uses multiple locales, it can get tedious keeping track of translations that are yet to be added to the appropriate translation files, e.g English translations in `resources/lang/en/validation.php` and French translations in `resources/lang/fr/validation.php`.

To automatically make these checks so I know when a translation is missing, I add unit tests.

It's been tricky however given I started out by trying to use the `trans()` and `__()` helpers to retrieve translation strings as below.

```php
<?php

// Using __() helper
$englishText = __('validation.is_tall', [], 'en');

$frenchText = __('validation.is_tall', [], 'fr');

// Using trans() helper
$englishText = trans('validation.is_tall', [], 'en');

$frenchText = __('validation.is_tall', [], 'fr');
```

The issue with using these helpers is that they're written to automatically fallback to a default translation if the requested translation is not found, and further, return the translation key if a default translation is not found.

Notice we're using a custom validation rule, `is_tall`, for which translation strings are to be added as follows.

In `resources/lang/en/validation.php` file:

```php
<?php

return [
  // ...
  
  'is_tall' => 'You must be tall',
  
  // ...
];
```

And in resources/lang/fr/validation.php file:

```php
<?php

return [
  // ...
  
  'is_tall' => 'Tu dois être grand',
  
  // ...
];
```

I later came across [this](https://laracasts.com/discuss/channels/laravel/determining-if-a-translation-in-specific-locale-exists?reply=111070) Laracasts answer by Bobby Bouwmann in which he mentions he made a [pull request](https://github.com/laravel/framework/commit/3c4ec8112daee69bd50c5a5fa174642924a151ea) to the Laravel repository to address the above problem, and it was merged. He added the method `hasLocaleFor()`.

```php
<?php

    /**
     * Determine if a translation exists for a given locale.
     *
     * @param  string  $key
     * @param  string  $locale
     * @return bool
     */
    public function hasForLocale($key, $locale = null)
    {
        return $this->has($key, $locale, false);
    }
```

Which does exactly what we want, check if a translation exists for a given locale. We can now go ahead and write the unit tests as below.

```php
<?php

namespace Tests\Unit;

use Tests\TestCase;

class IsTallRuleTest extends TestCase
{
    /** @test */
    public function it_has_an_english_validation_message()
    {
        $this->assertTrue(
            \Lang::hasForLocale('validation.is_tall', 'en'),
            'English validation message not found'
        );
    }

    /** @test */
    public function it_has_a_french_validation_message()
    {
        $this->assertTrue(
            \Lang::hasForLocale('validation.is_tall', 'fr'),
            'French validation message not found'
        );
    }
}

```