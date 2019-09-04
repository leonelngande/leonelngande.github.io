---
layout: post
hidden: false
title: 'Deep Cloning Objects in Angular, Typescript, Javascript'
tags: []
---
Traditional methods for deep cloning objects in Javascript are either using [object destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) or a combination of [JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) and [JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse).

```typescript
const objectA = { foo: { bar: 'baz' } };  // initial value of objectA
const cloneOfObjectA = {...objectA}; // objectA destructured to create cloneOfObjectA

const objectB = { foo: { bar: 'baz' } };  // initial value of objectB
const cloneOfObjectB = JSON.parse(JSON.stringify(objectB)); // create a clone of objectB using JSON.parse() and JSON.stringify()
```
d

I am currently taking the [Angular Architecture and Best Practices](https://www.pluralsight.com/courses/angular-architecture-best-practices) course on Pluralsight and [@DanWahlin](https://twitter.com/DanWahlin) (it's author) mentions a really cool and lightweight library for achieving just this: [clone](https://github.com/pvorb/clone).

It's a single javascript file, no dependencies and at the time of writing has over 607M downloads!

To install it, simple run `npm install clone`, just follow the [readme](https://github.com/pvorb/clone) for more usage notes 🙂.

Here's a wrapper service for the library when using with Angular and Typescript:

```typescript
import {Injectable} from '@angular/core';
import * as clone from 'clone';

@Injectable({providedIn: 'root'})
export class ClonerService {

    deepClone<T>(value): T {
        return clone<T>(value);
    }

}
```

And it's usage:



Happy coding!
