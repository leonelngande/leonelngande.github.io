---
layout: post
hidden: false
title: "Laravel: Test validation message key is defined in all translation
  files, en, fr, etc"
author: Leonel Elimpe
tags: []
last_modified_at: 2020-09-13 06:52:41
---
Below is the validation language line we want to test, It's found in `resources/lang/en/validation.php` and `resources/lang/fr/validation.php` my app supports just two languages at this time, English and French.

```php
<?php

return [

    // ...
    
    'empty_unless' => 'The :attribute must be empty unless :other is :value.',
    
    // ...

];
```

Now it's easy to forget to define `empty_unless` in one of the translations files and add the proper translation. So I write tests like below so am always reminded there's a missing translation.

```php
<?php

namespace Tests\Unit;

use Tests\TestCase;

class ValidationLanguageLinesTest extends TestCase
{
    // ...

    /** @test */
    public function empty_unless_has_an_english_translation()
    {
        self::assertTrue(
            \Lang::hasForLocale('validation.empty_unless', 'en'),
            'English translation for `empty_unless` not found'
        );
    }

    /** @test */
    public function empty_unless_has_a_french_translation()
    {
        self::assertTrue(
            \Lang::hasForLocale('validation.empty_unless', 'fr'),
            'French translation for `empty_unless` not found'
        );
    }
    
    // ...
}
```

And that's it! Should I forget to add a translation, I will be reminded by a failing unit test. 

If you think this testing approach can be improved in any way, please leave a comment below, will be happy to hear it.