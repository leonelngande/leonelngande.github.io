---
layout: post
hidden: false
title: >-
  Conditionally Add a Member to an Object in Javascript Using the Spread
  Operator
tags:
  - Spread Operator
  - Javascript
---
Say you have an object you'd like to conditionally add a member to, here's a neat little trick to do it with the spread operator.

```javascript
function createObject(someParam = null) {
    return {
        param1: 'value',
        param2: 'value',
        ...(someParam ? {someParam} : {} ),
    };
}
```

The `spread syntax` allows an iterable to expand in places where 0+ arguments are expected. For a more in-depth explanation have a look at [this](https://codeburst.io/javascript-es6-the-spread-syntax-f5c35525f754) article by [Brandon Morelli](https://codeburst.io/@bmorelli25).

The example is pretty contrived, but what we're doing here is checking if `someParam` has a value, then creating an object with it which is copied to the returned object using the spread operator. 

If `someParam` doesn't have a value, we simple copy an empty object to the returned object.

Notice that the spread part could still be written like this:

```javascript
...(someParam ? {someParam: someParam} : {} )
```
But we're making use of the ES6/ES2015 object shorthand property.

With it, if you want to define an object who's keys have the same name as the variables passed-in as properties, you can use the shorthand and simply pass the key name as we did above.
