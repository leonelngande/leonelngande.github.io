---
layout: post
hidden: false
title: Rendering `ion-item`'s Content "Predictably"
author: Leonel Elimpe
tags:
  - Ionic
last_modified_at: 2020-08-27 10:24:42
---
<ion-item>CONTENT</ion-item>

 If you've ever struggled with getting "CONTENT" above to render "predictable", I just found out if you wrap it in ion-row > ion-col, it will do just that.

```html
<ion-item>
    <ion-row style="width: 100%">
        <ion-col>
            <h6>Title</h6>
            <div>Some text</div>
            <app-example-component></app-example-component>
            <ion-list>...</ion-list>

            etc.
        </ion-col>
    </ion-row>
</ion-item>
```

By default we are limited to something like below when using `ion-item` .

```html
<ion-item>
    <ion-avatar slot="start">...</ion-avatar>
	<h6>Title</h6>
    <p>Paragraph</p>
    <ion-label></ion-label>
</ion-item>
```

I've personally struggled with getting more complex components properly rendered inside an ion-item, and have had to resort to using a card sometimes. Wrapping the content of ion-item inside `ion-row > ion-col` as in the first example did the trick for me, I hope it does for you too.