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
        self::assertStringContainsString(
            'tentative value for OTHER',
            trans(
              'validation.empty_unless',
              ['other' => 'tentative value for OTHER'],
              'en'
            )
        );

        self::assertStringContainsString(
            'tentative value for OTHER',
            trans(
              'validation.empty_unless',
              ['other' => 'tentative value for OTHER'],
              'fr'
            )
        );
    }

    /** @test */
    public function a_replacement_for__value__can_be_passed_to_the_empty_unless_validation_message()
    {
        self::assertStringContainsString(
            'tentative value for VALUE',
            trans(
              'validation.empty_unless',
              ['other' => 'tentative value for VALUE'],
              'en'
            )
        );

        self::assertStringContainsString(
            'tentative value for VALUE',
            trans(
              'validation.empty_unless',
              ['other' => 'tentative value for VALUE'],
              'fr'
            )
        );
    }
    
    // ...
}
```

And there you have it! If you think this approach can be improved in any way, please leave a comment below, will be happy to hear it.