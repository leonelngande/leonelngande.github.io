---
layout: post
hidden: false
title: 'Deep Cloning Objects in Angular, Typescript, Javascript'
tags:
  - Angular
  - Deep Cloning Objects
  - Typescript
  - Javascript
---
Some of the techniques for cloning objects in Javascript are either using [object destructuring](https://javascript.info/destructuring-assignment) or a combination of [JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) and [JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse).

```typescript
const objectA = { foo: { bar: 'baz' } };  // initial value of objectA
const cloneOfObjectA = {...objectA}; // objectA destructured to create cloneOfObjectA

const objectB = { foo: { bar: 'baz' } };  // initial value of objectB
const cloneOfObjectB = JSON.parse(JSON.stringify(objectB)); // create a clone of objectB using JSON.parse() and JSON.stringify()
```

Each of the above methods has limitations if you're looking to deep clone your object.

Object destructuring with the spread operator (i.e. Object.assign) for objects `const a = {...b}` only makes shallow copies of those objects, meanwhile using the JSON methods will dump the prototype chain and any methods on the object (more [here](http://www.zsoltnagy.eu/cloning-objects-in-javascript/)).

I am currently taking the [Angular Architecture and Best Practices](https://www.pluralsight.com/courses/angular-architecture-best-practices) course on Pluralsight and [@DanWahlin](https://twitter.com/DanWahlin) (it's author) mentions a really cool and lightweight library for achieving just this: [clone](https://github.com/pvorb/clone).

It offers foolproof deep cloning of objects, arrays, numbers, strings, maps, sets, promises, etc. in JavaScript, has 0 dependencies and at the time of writing has over 607M downloads!

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

And a (quite trivial) usage example:

```typescript
    const objectA = { foo: { bar: 'baz' } };  // initial value of objectA
    constructor(private clonerService: ClonerService) {
        const cloneOfObjectA = this.clonerService.deepClone(this.objectA);
    }
```

Hopefully this helps in your deep cloning adventures, happy coding!
