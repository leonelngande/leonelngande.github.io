---
layout: post
hidden: false
title: Pass Value to Replace a Custom Placeholder in a Laravel Validation Message
tags:
  - Laravel
  - PHP
---
Say you have the following validation message in the `custom` array of your language file - `resources/lang/xx/validation.php` - (or another location depending on your use case).

```php
<?php

return [
  
  // ...
  
  'custom' => [
      'email' => [
          'supported_email_provider' => ':provider not supported at this time.',
      ],
  ],
  
  // ...
];  
```

`supported_email_provider` is a custom [validation rule](https://laraveldaily.com/how-to-create-custom-validation-rules-laravel/) here and `:provider` is a [custom placeholder](https://laravel.com/docs/7.x/validation#custom-error-messages) within the validation message.

Properly passing in a value for :provider has got me searching the web the last couple hours, fortunately, I found [this Stack Overflow](https://stackoverflow.com/a/50535139/6924437) answer that provides a clean solution to this, check it out!

<https://stackoverflow.com/a/50535139/6924437>