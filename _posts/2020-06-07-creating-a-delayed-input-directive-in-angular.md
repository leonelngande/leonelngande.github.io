---
layout: post
hidden: false
title: Creating a Delayed Input Directive in Angular
author: Leonel Elimpe
excerpt_separator: <!--more-->
last_modified_at: 2020-06-08T19:25:21.708Z
tags:
  - Angular
---
[Demo and source code](#demo)

[Introduction](#introduction)

<!--more-->

[Generating the directive](#generating-the-directive)

[Adding functionality](#adding-functionality)

[Usage example](#usage-example)

[Further reading](#further-reading)

[Special thanks](#thanks)

## <a name="demo"></a>Demo and source code

Here's a link to the [demo](https://leonelngande.github.io/ng-delayed-input-demo/), and source code on [Github](https://github.com/leonelngande/ng-delayed-input-demo).

![](https://www.leonelngande.com/images/uploads/delayed-input.gif)

## <a name="introduction"></a>Introduction

Imagine in your app there's a search input that triggers an http request on each keystroke as a user types in their query. As your userbase grows, search operations quickly become expensive due to the increased traffic to your server. 

To mitigate this, a directive can be created to enable us to emit a value from the search input only after a particular time span has passed without another keystroke from the user. It will delay new keystrokes but drop previous pending delayed keystrokes if a new one arrives from the search input. Let's dig in!

## <a name="generating-the-directive"></a>Generating the directive

We start by creating a new Angular project with the command:

`ng new delayed-input-demo`

Then create a module we'll register the directive in:

`ng g module delayed-input`

After which we create and [register](https://stackoverflow.com/a/40649834) the directive inside the above module with the command:

`ng generate directive delayed-input/delayed-input --export=true`

We've used the `--export=true` CLI option so the directive is automatically added to the exports array of `DelayedInputModule`, which should now look like this:

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { DelayedInputDirective } from './delayed-input.directive';


@NgModule({
  declarations: [DelayedInputDirective],
  imports: [
    CommonModule
  ],
  exports: [DelayedInputDirective]
})
export class DelayedInputModule { }
```

## <a name="adding-functionality"></a>Adding functionality

Now let's flesh out the directive, `delayed-input.directive.ts`, as below. Notice I've added numbered comments to important code lines which we'll be reviewing.

```typescript
import {Directive, ElementRef, EventEmitter, Input,
        OnDestroy, OnInit, Output} from '@angular/core';
import {fromEvent, Subject, timer} from 'rxjs';
import {debounce, distinctUntilChanged, takeUntil} from 'rxjs/operators';

@Directive({
  selector: '[appDelayedInput]'
})
export class DelayedInputDirective implements OnInit, OnDestroy {

  private destroy$ = new Subject<void>(); // 0️⃣

  @Input() delayTime = 500; // 1️⃣
  @Output() delayedInput = new EventEmitter<Event>();  // 2️⃣

  constructor(private elementRef: ElementRef<HTMLInputElement>) { // 3️⃣
  }

  ngOnInit() {
    fromEvent(this.elementRef.nativeElement, 'input') // 4️⃣
      .pipe(
        debounce(() => timer(this.delayTime)),  // 5️⃣
        distinctUntilChanged(
          null,
          (event: Event) => (event.target as HTMLInputElement).value
        ), // 6️⃣
        takeUntil(this.destroy$), // 7️⃣
      )
      .subscribe(e => this.delayedInput.emit(e)); // 8️⃣
  }

  ngOnDestroy() {
    this.destroy$.next(); // 9️⃣
  }

}
```

* 0️⃣: We declare and initialize `destroy$` as an RxJS `Subject`. Used with the `takeUntil` operator, it will help us unsubscribe from RxJS subscriptions when the directive is destroyed.



* 1️⃣: We declare `delayTime` and set it's value to `500` milliseconds. It represents the timeout duration in milliseconds for the window of time required to wait for emission silence before emitting the most recent source (`userInput$`) value. We'll use this together with RxJS's `debounce`  and `timer` operators to only emit a value from `userInput$` after 500 ms has passed without another emission from the subject. Notice we've decorated `delayTime` with `@Input()` so that a different value can be passed in when applying the directive.



* 2️⃣: We declare `delayedInput`, decorate it with `@Output()`, and make it an `EventEmitter`. We'll use it push out a stream of delayed user inputs.



* 3️⃣: We get a reference to the host `HTMLInputElement` via constructor injection.



* 4️⃣: Using the `fromEvent` RxJS operator, we listen for `input` events on the directive's host element (an `HTMLInputElement`). We access to the host element - `this.elementRef.nativeElement` - through the element reference injected in the constructor.



* 5️⃣: We apply a combination of the `debounce` and `timer` operators to enable us to emit a value from the source Observable only after a particular time span has passed without another source emission. It passes only the most  recent value from each burst of emissions, and has the effect of only emitting search queries after the user stops typing.  If wondering why we didn't use `debounceTime` instead, please read [this](https://stackoverflow.com/a/57306062/6924437).



* 6️⃣: We apply the `distinctUntilChanged` operator which only emits when the current value is different from the last. This way, search queries not different from the last are dropped and not emitted. Note without the `keySelector` function passed in as the second argument, the `distinctUntilChanged` will not behave as we might expect it to. It will evaluate on the value reference. The `Event`s that are emitted over `fromEvent` will always be a different reference so it won't do anything. Thus, we pass in a `keySelector` function that takes in the current value and returns a `key` for use in comparing the current value to the previous value. In our case, we'll be returning the text in the input box, `event.target.value`. Of course a compare function can be passed in instead as the first argument, which I tried but was not able to get it working as expected. If you're able to, please do let me know you did it.



* 7️⃣: We make use of the `takeUntil` operator which emits values emitted the source Observable until a `notifier` Observable (`destroy$`) emits a value.



* 8️⃣: We call `this.delayedInput.emit(e)` to emit the delayed event.



* 9️⃣: Last but not the least, we call `next()` on the `destroy$` Subject in `ngOnDestroy` to automatically unsubscribe the `fromEvent` subscription when the directive is destroyed.

## <a name="usage-example"></a>Usage example

To use the directive, we have to import `DelayedInputModule` in `AppModule`.

```typescript
@NgModule({
  // ...
  
  imports: [
    // ...
    
    DelayedInputModule,
  ],
  
  // ...
})
export class AppModule { }
```

Then update `AppComponent` to add a usage example. We start with `app.component.html`  replacing it's content as below.

```html
<input type="text"
       placeholder="Search.."
       appDelayedInput
       (delayedInput)="search($event)"
       [delayTime]="600">
```

Followed by `app.component.ts` as below.

```typescript
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {

  search($event: Event) {
	// Do something with the input value, maybe make an http request?
	
    /**
    * You need to explicitly tell TypeScript the type of the HTMLElement which is your target.
    * @see https://stackoverflow.com/a/42066698/6924437
    */
    console.log( ($event.target as HTMLInputElement).value );

  }

}
```



## <a name="further-reading"></a>Further reading

- [Angular - Attribute Directives](https://angular.io/guide/attribute-directives)

- [Subject - Learn RxJS](https://www.learnrxjs.io/learn-rxjs/subjects/subject)

- [fromEvent - Learn RxJS](https://www.learnrxjs.io/learn-rxjs/operators/creation/fromevent)

- [debounce - Learn RxJS](https://www.learnrxjs.io/learn-rxjs/operators/filtering/debounce)

- [timer - Learn RxJS](https://www.learnrxjs.io/learn-rxjs/operators/creation/timer)

- [distinctUntilChanged - Learn RxJS](https://www.learnrxjs.io/learn-rxjs/operators/filtering/distinctuntilchanged)

- [Angular - EventEmitter](https://angular.io/api/core/EventEmitter)

- [Angular - ElementRef](https://angular.io/api/core/ElementRef)

- [Angular - Input](https://angular.io/api/core/Input)

- [Angular - Output](https://angular.io/api/core/Output)



## <a name="thanks"></a>Special thanks to

- [Sam Vloeberghs](https://twitter.com/samvloeberghs)
- [Santosh Yadav](https://twitter.com/SantoshYadavDev)
- [Jan-Niklas Wortmann](https://twitter.com/niklas_wortmann)

for reviewing this post and providing valuable and much-appreciated feedback!
