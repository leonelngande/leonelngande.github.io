---
layout: post
hidden: false
title: Format and Display a Javascript Date Object as Date and Time
tags:
  - Javascript
  - Date and Time Formatting
  - Javascript Date Object
---
Say we have the following Javascript date object and would like to display it as date and time.

```javascript
const date = new Date();
```

Here's how to using methods available on the date object itself:

```javascript
const dateTimeString = date.toLocaleDateString() + ' ' + date.toLocaleTimeString();
```

The above will output `"5/10/2019 11:49:44 AM"` (time of writing of this article).

Full code:

```javascript
const date = new Date();

const dateTimeString = date.toLocaleDateString() + ' ' + date.toLocaleTimeString();
```



Happy coding!
