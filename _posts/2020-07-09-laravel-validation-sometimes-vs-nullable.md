---
layout: post
hidden: false
title: "Laravel Validation: sometimes vs nullable"
author: Leonel Elimpe
tags:
  - Laravel
last_modified_at: 2020-07-09 11:43:44
---
* **[sometimes](https://laravel.com/docs/6.x/validation#conditionally-adding-rules)**: Only apply the rest of the validation rules if the field shows up in the request.
* **[nullable](https://laravel.com/docs/6.x/validation#rule-nullable)**: If the field shows up in the request and is null (undefined, empty, has no value at all, etc), do not apply the rest of the validation rules.

<br>

## sometimes

`sometimes` adds the defined validation conditions to a given field if the field is **present** in the request.

It's therefore useful in situations where you wish to run validation checks against a field **only** if that field is present in the request. 

Take for instance the examples presented in [this](https://stackoverflow.com/a/35839650/6924437) answer by haakym on Stackoverflow (also presented below).

I use this rule when I have some javascript on a page that will *disable* a field, as when a field is disabled it won't show up in the request. If I simply said `required|email` the validator is always going to apply the rules whereas using the `sometimes` rule will only apply the validation if the field shows up in the request! Hope that makes sense.

Examples:

```
input: []
rules: ['email' => 'sometimes|required|email']
result: pass, the request is empty so sometimes won't apply any of the rules

input: ['email' => '']
rules: ['email' => 'sometimes|required|email']
result: fail, the field is present so the required rule fails!

input: []
rules: ['email' => 'required|email']
result: fail, the request is empty and we require the email field!
```

<br>

## nullable

By default, Laravel includes the `TrimStrings` and `ConvertEmptyStringsToNull` middleware in your application's global middleware stack. These middleware are listed in the stack by the `App/Http/Kernel` class.

* [`TrimStrings`](https://github.com/laravel/framework/blob/6.x/src/Illuminate/Foundation/Http/Middleware/TrimStrings.php) takes care of Stripping whitespace (or other characters) from the beginning and end of each field in the request.
* [`ConvertEmptyStringsToNull`](https://github.com/laravel/framework/blob/6.x/src/Illuminate/Foundation/Http/Middleware/ConvertEmptyStringsToNull.php) as it's name suggests checks each field and if a field is just an empty string, it assigns null to it. Say the field was `name=""`, it'll be transformed into `name=null`.

Because of this, you will often need to mark your "optional" request fields as `nullable` if you do not want the validator to consider the `null` values as invalid. For example:

```
    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);
```

In this example, we are specifying that the `publish_at` field may be either `null` or a valid date representation. If the `nullable` modifier is not added to the rule definition, the validator would consider `null` an invalid date.

<br>

## Further reading

* [Conditionally Adding Rules - Laravel Docs](https://laravel.com/docs/6.x/validation#conditionally-adding-rules)
* [nullable - Laravel Docs](https://laravel.com/docs/6.x/validation#rule-nullable)
* [Validator.php - Github](https://github.com/laravel/framework/blob/5.3/src/Illuminate/Validation/Validator.php#L281)