---
layout: post
hidden: false
title: Creating a Delayed Input Directive in Angular
author: Leonel Elimpe
tags:
  - Angular
---
[Demo and source code](#demo)

[Introduction](#introduction)

[Generating the directive](#generating-the-directive)

[Adding functionality](#adding-functionality)

[Usage example](#usage-example)

[Further reading](#further-reading)

## <a name="demo"></a>Demo and source code

Here's a link to the [demo](https://leonelngande.github.io/ng-delayed-input-demo/), and source code on [Github](https://github.com/leonelngande/ng-delayed-input-demo).

## <a name="introduction"></a>Introduction

Imagine in your app there's a search input which triggers an http request on each keystroke as a user types in their query. As your userbase grows, search operations quickly become expensive due to the increased traffic to your server. 

To mitigate this, a directive can be created to enable us emit a value from the search input only after a particular time span has passed without another keystroke from the user. It'll delay new keystrokes but drop previous pending delayed keystrokes if a new one arrives from the search input. Let's dig in!

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
import {Directive, EventEmitter, HostListener, Input, OnDestroy, OnInit, Output} from '@angular/core';
import {Subject, timer} from 'rxjs';
import {debounce, distinctUntilChanged, takeWhile} from 'rxjs/operators';

@Directive({
  selector: '[appDelayedInput]'
})
export class DelayedInputDirective implements OnInit, OnDestroy {

    private userInput$ = new Subject<InputEvent>(); // 0️⃣
    private alive = true; // 1️⃣
    
    @Input() dueTime = 500; // 2️⃣
    @Output() delayedInput = new EventEmitter<InputEvent>(); // 3️⃣

    constructor() { }

    ngOnInit() {
        this.userInput$ // 5️⃣
            .pipe(
                debounce(() => timer(this.dueTime)), // 6️⃣
                distinctUntilChanged(), // 7️⃣
                takeWhile(() => this.alive),
            )
            .subscribe(e => this.delayedInput.emit(e)); // 8️⃣
    }

    @HostListener('input', ['$event']) // 4️⃣
    inputEvent(event: InputEvent) {
        event.preventDefault();
        event.stopPropagation();
        this.userInput$.next(event);
    }

    ngOnDestroy() {
        this.alive = false; // 9️⃣
    }

}
```

* 0️⃣: We declare and initialize `userInput$` as an RxJS `Subject` such that it has no initial value or replay behaviour. It'll enable us continuously store values coming in through an input event listener (see 4️⃣) attached to the directive's host element.



* 1️⃣: We declare and initialize `alive` to true. It'll help us unsubscribe from RxJS subscriptions when the directive is destroyed.



* 2️⃣: We declare `dueTime` and set it's value to `500` milliseconds. It represents the timeout duration in milliseconds for the window of time required to wait for emission silence before emitting the most recent source (`userInput$`) value. We'll use this together with RxJS's `debounce`  and `timer` operators to only emit a value from `userInput$` after 500 ms has passed without another emission from the subject. Notice we've decorated `dueTime` with `@Input()` so that a different value can be passed in when applying the directive.



* 3️⃣: We declare `delayedInput`, decorate it with `@Output()`, and make it an `EventEmitter`. We'll use it push out a stream of delayed user inputs.



* 4️⃣: Using Angular's `@HostListener()` decorator, we listen for `input` events on the directive's host element (an `HTMLInputElement`). We also provide a handler method - `inputEvent()` - to run when that event occurs. Inside this method, we call `event.preventDefault()` to prevent triggering the event's default action, `event.stopPropagation()` to stop the event from further propagating/bubbling to parent elements, and `this.userInput$.next(event)` to feed a new value - the input event - to the `userInput$` Subject.
* 5️⃣: When the directive initializes, we begin listening for emissions from the `userInput$` Subject.



* 6️⃣: We apply a combination of the `debounce` and `timer` operators to enable us emit a value from the source Observable (`userInput$`) only after a particular time span has passed without another source emission. It passes only the most  recent value from each burst of emissions, and has the effect of only emitting search queries after the user stops typing.  If wondering why we didn't use `debounceTime` instead, please read [this](https://stackoverflow.com/a/57306062/6924437).



* 7️⃣: We apply the `distinctUntilChanged()` operator which only emits when the current value is different from the last. This way, search queries not different from the last are dropped and not emitted.



* 8️⃣: We call `this.delayedInput.emit(e)` to emit an event containing the value in `e`, an `InputEvent` from `userInput$`.



* 9️⃣: Last but not the least, we set `alive` to `false` in `ngOnDestroy()` to automatically unsubscribe the `userInput$` subscription when the directive is destroyed.

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
       [dueTime]="600">
```

Followed by `app.component.ts` as below.

```typescript
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {

  search($event: InputEvent) {
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

[Attribute Directives](https://angular.io/guide/attribute-directives)

[RxJS Subject](https://www.learnrxjs.io/learn-rxjs/subjects/subject)

[RxJS debounce](https://www.learnrxjs.io/learn-rxjs/operators/filtering/debounce)

[RxJS timer](https://www.learnrxjs.io/learn-rxjs/operators/creation/timer)

[RxJS distinctUntilChanged](https://www.learnrxjs.io/learn-rxjs/operators/filtering/distinctuntilchanged)

[RxJS takeWhile](https://www.learnrxjs.io/learn-rxjs/operators/filtering/takewhile)

[EventEmitter](https://angular.io/api/core/EventEmitter)

[@HostListener()](https://angular.io/api/core/HostListener)

[@Input()](https://angular.io/api/core/Input)

[@Output()](https://angular.io/api/core/Output)