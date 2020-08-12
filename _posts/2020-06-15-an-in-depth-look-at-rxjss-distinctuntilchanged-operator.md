---
layout: post
hidden: false
title: An in-depth look at RxJS's distinctUntilChanged Operator
author: Leonel Elimpe
excerpt: In this post, we'll have an in-depth look at RxJS's `distinctUntilChanged` operator, it's signature and what it does, it's parameters `compare` and `keySelector`, and typical use cases for each of them and both of them.
image:
  path: /images/uploads/distinctuntilchanged-icon-cover.png
  thumbnail: /images/uploads/distinctuntilchanged-icon-cover.png
tags:
  - RxJS
  - distinctUntilChanged
  - Featured
---
* [Introduction](introduction)
* [What is distinctUntilChanged](definition)
  * [It's function signature(s)](signature)
  * [The compare function](compare)
  * [The keySelector function](keyselector)
* [Usage Examples](usage)
  * [With value streams e.g. numbers, strings](usage-value-streams)
  * [With value streams and the compare function](usage-value-streams-compare)
  * [With object streams](usage-object-streams)
  * [With object streams and the compare function](usage-object-streams-compare)
  * [With object streams and the keySelector function](usage-object-streams-keyselector)
  * [With object streams + the compare and keySelector functions](usage-object-streams-compare-keyselector)
* [Conclusion](conclusion)
* [Further reading](further-reading)
* [Special thanks to](special-thanks)



<br/>


## <a name="introduction"></a>Introduction

In this post, we'll have an in-depth look at RxJS's `distinctUntilChanged` operator, it's signature and what it does, it's parameters `compare` and `keySelector`, and typical use cases for each of them and both of them.



<br/>

## <a name="definition"></a>What is distinctUntilChanged?

It is an RxJS operator which returns an Observable that emits items from the source Observable with distinct values. It only emits when the current emitted value is different from the last emitted value.

From the official documentation, it

> Returns an Observable that emits all items emitted by the source Observable that are distinct by comparison from the previous item.
>
> 
>
> If a comparator function is provided, then it will be called for each item to test for whether or not that value should be emitted.
>
> The comparator function should return true if the values are the same, and false if they are different.
>
> 
>
> If a comparator function is not provided, an equality check is used by default.



<br/>

### <a name="signature"></a>It's function signature(s)

A function signature defines input and output of functions or methods, and can include parameters and their types.

Two signatures are presented in the RxJS official docs for `distinctUntilChanged`. 

1. 
    ```typescript
    distinctUntilChanged<T>(compare?: (x: T, y: T) => boolean): Observable<T>;
    ```
    When this signature is used, `distinctUntilChanged` can be passed an optional `compare` function. Here, the generic `T` represents the **type** of the value `distinctUntilChanged` is operating on. 

2. 
    ```typescript
    distinctUntilChanged<T, K>(compare: (x: K, y: K) => boolean, keySelector: (x: T) => K): Observable<T>;
    ```
    When this signature is used, `distinctUntilChanged` can be passed an optional `compare` function as well as an optional `keySelector` function. Here, the generic `T` is still the **type** of the Observable while `K` is the **return type** of the `keySelector` function. 

Let's have a deeper look at the `compare` and `keySelector` functions.



<br/>

### <a name="compare"></a>The `compare` function

It's an optional comparison function called to test if an item is distinct from the previous item in the source. It is called with two parameters - the current item from the source and the previous item from the source . It should return a `boolean`.

If a compare function is not provided, `distinctUntilChanged` defaults to it's built in `compare` function which makes use of an **equality check**.

```typescript
  private compare(x: any, y: any): boolean {
    return x === y;
  }
```

Below are a few use cases for a custom `compare` function.

- You need to write custom comparison logic, e.g. converting all strings to lowercase before comparing them.
- You are working with objects and would like to compare a specific key in the objects



<br/>

### <a name="keyselector"></a>The `keySelector` function

It's an optional function to select which value you want to check as distinct, a function to compute the comparison key for each element. 

If one is passed to `distinctUntilChanged`, it'll be called for every value emitted from the source and it's result used in making comparisons in the `compare` function. It is called with one parameter - the current item from the source.

`keySelector` is what you need when working with objects and would like to select a specific key for use in comparisons.



<br/>

## <a name="usage"></a>Usage Examples

### <a name="usage-value-streams"></a>With value streams e.g numbers, strings

```typescript
import { of } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

of(1, 1, 2, 2, 2, 1, 1, 2, 3, 3, 4).pipe(
    distinctUntilChanged(),
  )
  .subscribe(x => console.log(x)); // 1, 2, 1, 2, 3, 4
```

[Stackblitz](https://stackblitz.com/edit/distinctuntilchanged-with-value-streams)



<br/>

### <a name="usage-value-streams-compare"></a>With value streams and the `compare` function

```typescript
import { of } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

/**
 * Create a stream from a mix of upper and lowercase characters
 */
of('a', 'A', 'b', 'c', 'D', 'd', 'e', 'f', 'G', 'g', 'h').pipe(
    distinctUntilChanged((curr, prev) => {
      /**
       * Two characters are the same if they match each other in lowercase
       */
      return curr.toLowerCase() === prev.toLowerCase();
    }),
  )
  .subscribe(x => console.log); // a b c D e f G h
```

[Stackblitz](https://stackblitz.com/edit/distinctuntilchanged-with-value-streams-plus-compare-function)



<br/>

### <a name="usage-object-streams"></a>With objects streams

Note that using distinctUntilChanged on an object stream will result in the comparison of object references and not the objects' property values.

```typescript
import { from } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

const sampleObject = { name: 'Test' };

// Objects must be same reference
const source$ = from([sampleObject, sampleObject, sampleObject]);

// only emit distinct objects, based on last emitted value
source$
  .pipe(
    distinctUntilChanged()
  )
  .subscribe(console.log);

// console output: 
// {name: 'Test'}
```

[Stackblitz](https://stackblitz.com/edit/distinctuntilchanged-with-object-streams)

Since all items in `source$` array reference the same object, `sampleObject`, only one item gets outputted to the console after applying `distinctUntilChanged`.

Should we change the reference of one of the objects in `source$` array, the output will change as follows.

```typescript
import { from } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

const sampleObject = { name: 'Test' };

// Change the reference of the second array item
const source$ = from([sampleObject, {...sampleObject}, sampleObject]);

// only emit distinct objects, based on last emitted value
source$
  .pipe(
    distinctUntilChanged()
  )
  .subscribe(console.log);

// console output: 
// {name: 'Test'}
// {name: 'Test'}
// {name: 'Test'}
```

[Stackblitz](https://stackblitz.com/edit/distinctuntilchanged-with-object-streams-2)

When the reference of the second item from the `source$` observable is changed, it results in all the items being outputted to the console. This is because the first item has a unique reference, the second item has a unique reference, and the third item doesn't have a unique reference, but it has a reference different from that of the second item.



<br/>

### <a name="usage-object-streams-compare"></a>With object streams and the `compare` function

```typescript
import { from } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

/**
 * Create a stream from an array of car objects.
 */
from([
  { name: 'Porsche', model: '911' },
  { name: 'Porsche', model: '911' },
  { name: 'Ferrari', model: 'F40' }
]).pipe(
  distinctUntilChanged((prevCar, nextCar) => {
      /**
       * Two cars are the same if they have the same name and model
       */
      return (prevCar.name === nextCar.name) 
      	&& (prevCar.model === nextCar.model);
  })
)
.subscribe(console.log);

// console output: 
// {name: "Porsche", model: "911"}
// {name: "Ferrari", model: "F40"}
```

[Stackblitz](https://stackblitz.com/edit/distinctuntilchanged-with-object-streams-plus-compare-function)



<br/>

### <a name="usage-object-streams-keyselector"></a>With object streams and the `keySelector` function

```typescript
import { from } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

/**
 * Create a stream from an array of car objects.
 */
from([
  { name: 'Porsche', model: '911' },
  { name: 'Porsche', model: '911' },
  { name: 'Ferrari', model: 'F40' }
]).pipe(
  /**
   * Use keySelectorFn to return car names for use in comparison
   */
  distinctUntilChanged(null, (car) => car.name)
)
.subscribe(console.log);

// console output: 
// {name: "Porsche", model: "911"}
// {name: "Ferrari", model: "F40"}
```

[Stackblitz](https://stackblitz.com/edit/distinctuntilchanged-with-object-streams-plus-keyselector)



<br/>

### <a name="usage-object-streams-compare-keyselector"></a>With object streams + the `compare` and `keySelector` functions

I've not been able to come up with or find a convincing enough code example for this use case, if you know/think of one, please let me know so I update the article. Here's a contrived example.

```typescript
import { from } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

/**
 * Create a stream from an array of car objects with
 * names in both lower and upper case.
 */
from([
  { name: 'Porsche', model: '911' },
  { name: 'PORSCHE', model: '911' },
  { name: 'Ferrari', model: 'F40' },
  { name: 'FERRARI', model: 'F40' },
]).pipe(
  distinctUntilChanged(
     /**
      * Convert car names returned by keySelectorFn to lower case
      * before doing a comparison.
      */
     (prevCarName, nextCarName) => {
       return prevCarName.toLowerCase() === nextCarName.toLowerCase();
     },
     /**
      * Use keySelectorFn to return car names for use in comparison
      *
      * We have to manually add types due to a Typescript bug.
      * @see https://github.com/ReactiveX/rxjs/issues/5484
      */
    (car: {name: string, model: string}) => car.name
  )
)
.subscribe(console.log);

// console output: 
// {name: "Porsche", model: "911"}
// {name: "Ferrari", model: "F40"}
```

[Stackblitz](https://stackblitz.com/edit/distinctuntilchanged-with-object-streams-compare-n-keyselector)



<br/>

## <a name="conclusion"></a>Conclusion

In this post, we've had an in-depth look at RxJS's `distinctUntilChanged` operator and it's two optional parameters, `compare` and a `keySelector`. We equally did an overview of it's function signature(s), as well as specific usage examples.

Did you like this post? Then you might also enjoy reading my recent post, [Creating a Delayed Input Directive in Angular](https://www.leonelngande.com/creating-a-delayed-input-directive-in-angular/) in which I apply `distinctUntilChanged` as discussed this post.



<br/>

## <a name="further-reading"></a>Further reading

- [distinctUntilChanged - Learn RxJS](https://www.learnrxjs.io/learn-rxjs/operators/filtering/distinctuntilchanged)
- [distinctUntilChanged source code](https://github.com/ReactiveX/rxjs/blob/master/src/internal/operators/distinctUntilChanged.ts)
- [Operators - RxJS](https://rxjs-dev.firebaseapp.com/guide/operators)



<br/>

## <a name="special-thanks"></a>Special thanks to

- [Jan-Niklas Wortmann](https://twitter.com/niklas_wortmann)

for reviewing this post and providing valuable and much-appreciated feedback!