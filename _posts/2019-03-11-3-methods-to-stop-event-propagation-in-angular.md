---
layout: post
hidden: false
title: 3 Methods to Stop Event Propagation in Angular
tags: []
---
I think this is already support in three ways:



returning false causes preventDefault() to run. ie: <div (click)="doSomething(); false">

calling it manually: <button (click)="doSomething(); $event.stopPropagation()">

One can always implement the same directive in NG2 as:

@Directive({

  selector: \`click-stop-propagation\`

  events: 'stopClick($event)'

})

class ClickStopPropagation {

  stopClick(event:Event) {

\    event.preventDefault();

\    event. stopPropagation();

  }

}

Then use as:



<button (click)="doSomething()" click-stop-propagation>
