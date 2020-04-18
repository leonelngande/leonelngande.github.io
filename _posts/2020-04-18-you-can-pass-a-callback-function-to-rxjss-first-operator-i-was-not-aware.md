---
layout: post
hidden: false
title: You can pass a callback function to RxJS's first() operator, I was not aware!
tags:
  - RxJS
  - Javascript
---
My typical use-case for RxJS first() operator is to emit only the first item of an observable stream. What if you want the first value based on a certain condition instead? 

I recently found out you can pass in an optional callback function (predicate) just for this.

> <b>[First](http://reactivex.io/documentation/operators/first.html)</b>\
> <i>emit only the first item (or the first item that meets some condition) emitted by an Observable</i>.

`predicate` is a function you write to evaluate an expression and return true or false. You can have a look at it's source [here](https://github.com/ReactiveX/rxjs/blob/master/src/internal/operators/first.ts).

```typescript
export function first<T, D = T>(
  predicate: (value: T, index: number, source: Observable<T>) => boolean,
  defaultValue?: D
): OperatorFunction<T, T | D>;
```

As an example, say we setup a listener for DOM click events.

`const clicks = fromEvent(document, 'click');` 

We later decide we'd like to receive only the first click event that happens on a DIV, here's how we'll do that:

```typescript
import { fromEvent } from 'rxjs';
import { first } from 'rxjs/operators';

const clicks = fromEvent(document, 'click');

const result = clicks.pipe(first(ev => ev.target.tagName === 'DIV'));

result.subscribe(x => console.log(x));
```

As seen above, we make use of the first() operator passing it a callback function that receives a click event, and returns true if the tag name is DIV, false otherwise.

Hopefully, like me, you've learned something new from this tiny blog post 🙂.