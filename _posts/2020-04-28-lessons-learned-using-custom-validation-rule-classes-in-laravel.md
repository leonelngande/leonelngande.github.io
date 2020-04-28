---
layout: post
hidden: false
title: Lessons Learned Using Custom Validation Rule Classes in Laravel
tags:
  - Laravel
  - PHP
---
* Avoid using together with Validator::extend in a service provider, call   the class directly when declaring the validation rule.
* It's not required to return values for all placeholders in the validation message here, Laravel is able to still pickup the default ones (:attribute, :input, etc) even if you don't pass them in. So we only pass in a value for the :carrier placeholder which is custom and added by us.
* You can declare custom placeholders in language file validation messages and pass values to use in filling them from the rule class's message() method.
* You can conveniently format the :attribute placeholder to display as is (`:attribute`), to display in uppercase (`:ATTRIBUTE`), or just make the first letter uppercase (:Attribute). [More here](https://laracasts.com/discuss/channels/general-discussion/how-can-i-manipulate-the-attribute-name-on-validation?reply=387749)