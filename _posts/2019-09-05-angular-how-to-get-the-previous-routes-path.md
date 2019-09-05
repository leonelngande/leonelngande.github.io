---
layout: post
hidden: false
title: 'Angular: How to get the Previous Route''s Path'
tags: []
---
So I've been looking around for current solutions to getting the previous url's path in Angular and came across [this issue](https://github.com/angular/angular/issues/11268) on Github.

Many working solutions are proposed in that discussion thread. The one that's worked best for me is [this](https://github.com/angular/angular/issues/11268#issuecomment-391990641), updated to work with RxJs 6+:

```typescript
  ngOnInit() {
    this.router.events
      .pipe(
        // Take only events triggered when a navigation starts
        filter(e => e instanceof NavigationStart ),
        /**
         * @param {number} bufferSize The maximum size of the buffer emitted. Our bufferSize is 2
         * @param {number} [startBufferEvery] Interval at which to start a new buffer. Our startBufferEvery is 1
         * For example if `startBufferEvery` is `2`, then a new buffer will be started
         * on every other value from the source. A new buffer is started at the
         * beginning of the source by default.
         * @return {Observable<T[]>} An Observable of arrays of buffered values.
         */
        bufferCount(2, 1),
      )
      .subscribe(e => {
        if (e !== null && e !== undefined) {
          console.log({originalPath: e[0]['url'].substr(1)});
        }
      });
  }
```

I've observed the following after using it for a while:

\- When I initially navigate to the url (as in enter address in browser and hit enter), nothing is logged to console - a result of `bufferCount(2, 1)`.

\- When I click on any link on the loaded page, the originalPath is still not logged to the console.
