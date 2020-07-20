---
layout: post
hidden: false
title: Ionic 4 Swipe Navigation Between Pages
author: Leonel Elimpe
tags:
  - Ionic
  - Angular
last_modified_at: 2020-07-20 02:47:59
---
Imagine in your app there's a page with a list of items, and clicking on an item navigates to and item details page. On the details page, you'd like to be able to swipe left to bring up the previous item, or right to bring up the next item. This post aims to provide an implementation for such functionality.

Having gone through a good number of resources online in search of a way to implement swipe navigation between pages in Ionic 4, I've decided to put together what I've learned.

<br>

The crux of the problem is - I think - the removal of `hammerjs` from Ionic 4, which caused some gesture event bindings previously available in Ionic 3 to be missing in Ionic 4. This means events like swipe can no longer be used like below.

```html
<div (swipe)="onSwipe($event)"></div>
```

You can of course go ahead and add `hammerjs` to your app which is the quickest solution, but there's no guarantee something else won't break, more [here](<>).

I therefore decided to implement a custom solution instead which consisted of the following steps:

1. Create a directive to listen for swipe events.
2. Apply this directive to a page we want to support swipe navigation.
3. On left swipe, resolve the previous route and navigate backward, and on right swipe, resolve the next route and navigate forward.
4. Bonus: disable Ionic's built-in swipe left to go back gesture.

The code for this demo is available in [this Github repository](https://github.com/leonelngande/ionic4-swipe-navigation-between-pages). You can play with a live demo [here](https://leonelngande.github.io/ionic4-swipe-navigation-between-pages/).

<br>

## Creating a directive to listen for swipe events

I like to keep my directives in individual modules for easy re-use in other projects. So we'll generate a module then create, declare, and export the directive from it with the below commands.

```
ionic generate module swipe
```

```
ionic generate directive swipe/swipe --export=true
```

`SwipeModule` now looks like below.

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { SwipeDirective } from './swipe.directive';

@NgModule({
  declarations: [SwipeDirective],
  imports: [
    CommonModule
  ],
  exports: [SwipeDirective]
})
export class SwipeModule { }
```

Next let's flesh out `swipe.directive.ts` as below.

```typescript
import { AfterViewInit, Directive, ElementRef, EventEmitter, Output, Renderer2 } from '@angular/core';

@Directive({
  selector: '[appSwipe]'
})
export class SwipeDirective implements AfterViewInit {

  /** x position at touchstart */
  xDown = null;
  /** y position at touchstart */
  yDown = null;
  /** Timestamp at touchstart */
  time = 0;

  @Output() swipeLeft: EventEmitter<any>;
  @Output() swipeRight: EventEmitter<any>;
  // @Output() swipeUp: EventEmitter<any>;
  // @Output() swipeDown: EventEmitter<any>;

  constructor(private renderer: Renderer2, private elRef: ElementRef) {
    this.swipeRight = new EventEmitter<any>();
    this.swipeLeft = new EventEmitter<any>();
    // this.swipeUp = new EventEmitter<any>();
    // this.swipeDown = new EventEmitter<any>();
  }

  ngAfterViewInit() {
    /** Listen for touchstart event on element directive is attached to */
    this.renderer.listen(this.elRef.nativeElement, 'touchstart', (event: TouchEvent) => {
      this.handleTouchStart(event);
    });

    /** Listen for touchend event on element directive is attached to */
    this.renderer.listen(this.elRef.nativeElement, 'touchend', (event: TouchEvent) => {
      this.handleTouchMove(event);
    });
  }

  private handleTouchStart(event: TouchEvent) {
    this.xDown = event.touches[0].pageX;
    this.yDown = event.touches[0].pageY;
    this.time = event.timeStamp;
  }

  handleTouchMove(event: TouchEvent) {
    if ( ! this.xDown || ! this.yDown ) {
      return;
    }

    /** @see https://stackblitz.com/edit/angular-swipe-events-with-hostlistner */
    const touch = event.touches[0] || event.changedTouches[0];

    const xUp = touch.pageX;
    const yUp = touch.pageY;

    const xDiff = this.xDown - xUp;
    const yDiff = this.yDown - yUp;
    const timeDiff = event.timeStamp - this.time;

    // simulate a swipe -> less than 500 ms and more than 60 px
    if (timeDiff < 500) {
      // touch movement lasted less than 500 ms
      if (Math.abs(xDiff) > 60) {
        // delta x is at least 60 pixels
        if (xDiff > 0) {
          this.swipeRight.emit(event);
        } else {
          this.swipeLeft.emit(event);
        }
      }

      /*if (Math.abs(yDiff) > 60) {
        // delta y is at least 60 pixels
        if (yDiff > 0) {
          this.swipeDown.emit(event);
        } else {
          this.swipeUp.emit(event);
        }
      }*/
    }

    // Reset values.
    this.xDown = null;
    this.yDown = null;
  }
}
```

For an in-depth explanation of how the directive works, you can have a look at the companion blog post [A Custom Swipe Gesture Directive for Ionic 4 and Angular](<>)

<br>

## Applying the directive, listening for left and right swipes, and navigating accordingly

Let's apply the directive to a page, `ViewItemPage` (defined in Github repository linked above). We'll first update the template file attaching the directive to `ion-content`.

```html
<ion-header>
  <ion-toolbar color="primary">
    <ion-title>{{item.title}}</ion-title>
    <ion-buttons slot="start">
      <ion-back-button defaultHref="/home"></ion-back-button>
    </ion-buttons>
  </ion-toolbar>
</ion-header>

<ion-content appSwipe
             (swipeLeft)="onSwipeLeft($event)"
             (swipeRight)="onSwipeRight($event)">
    
    <!-- ... -->
    
</ion-content>
```

Notice how we've set `defaultHref="/home"` on the `ion-back-button` element. This ensures the back button appears on each ViewItemPage render even when the browser's navigation history is empty. Without that default, the back button will disappear when browser navigation history is empty.

Don't forget to import `SwipeModule` into the module the above component is declared in.

```typescript
// ...

@NgModule({
    imports: [
        // ...
        SwipeModule
    ],
  declarations: [ViewItemPage]
})
export class ViewItemPageModule {}
```

Next, let's add the methods `onSwipeLeft` and `onSwipeRight` to the `.ts` file and add logic to navigate backward and forward.

```typescript
import {Component, OnInit} from '@angular/core';
import {ActivatedRoute, Router} from '@angular/router';
import {ItemService} from '../../services/item.service';
import {NavController} from '@ionic/angular';
import {Item} from '../../models/item.model';

@Component({
  selector: 'app-view-item',
  templateUrl: './view-item.page.html',
  styleUrls: ['./view-item.page.scss'],
})
export class ViewItemPage implements OnInit {

  item: Item;

  constructor(
    private router: Router,
    private route: ActivatedRoute,
    private itemService: ItemService,
    private navCtrl: NavController,
  ) { }

  ngOnInit() {
    this.route.params.subscribe(
      data => {
        this.item = this.itemService.getItemById(Number(data.id));
        // if item is undefined, go back to home
        if (!this.item) {
          this.goBack();
        }
      }
    );
  }

  goBack() {
    this.navCtrl.navigateBack(['/home']);
  }

  onSwipeLeft($event) {
    const previousItem = this.itemService.getPreviousItem(this.item.id);
    if (previousItem) {
      this.navCtrl.navigateBack(['/', 'items', previousItem.id]);
    } else {
      // If no previous item return to the list of items
      this.goBack();
    }
  }

  onSwipeRight($event) {
    const nextItem = this.itemService.getNextItem(this.item.id);
    if (nextItem) {
      this.navCtrl.navigateForward(['/', 'items', nextItem.id]);
    }
  }
}
```

On left swipe, we get the previous item from a service and navigate `back` to it. If no previous item is found, we navigate back to the list of items (homepage).

On right swipe, we get the next item and navigate `forward` to it. If no next item is found, we simply do not navigate.

Of course, the full source code is available on [Github](https://github.com/leonelngande/ionic4-swipe-navigation-between-pages) and you can check it out!