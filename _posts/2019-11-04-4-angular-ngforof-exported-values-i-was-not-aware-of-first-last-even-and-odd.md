---
layout: post
hidden: false
title: >-
  4 Angular NgForOf Exported Values I Was Not Aware Of - first, last, even, and
  odd
tags:
  - Angular
  - '*ngFor'
author: Leonel Elimpe
---
Angular's [NgForOf](https://angular.io/api/common/NgForOf) provides exported values that can be aliased to local variables. Until today, I've only been aware of the `index` exported value, turns out there's [a couple more](https://angular.io/api/common/NgForOf#local-variables).

Four of them in particular are very applicable in my day-to-day usage of Angular: `first`, `last`, `even`, and `odd`.

* **first**

`first`, a boolean value, is true when the item is the first item in the iterable.

* **last**

`last`, a boolean value, is true when the item is the last item in the iterable.

* **even**

`even`, a boolean value, is true when the item has an even index in the iterable.

* **odd**

`odd`, a boolean value, is true when the item has an odd index in the iterable.

Not leaving out `index`, a number representing the index of the current item in the iterable.

Here's a usage example in which we alias the above five exported values to local variables:

```html
<li *ngFor="let user of users; index as i; first as isFirst; last as isLast; even as isEven; odd as isOdd">
  {{user}} <span *ngIf="isFirst">default</span>
</li>
```

And that's it 🙂, hope this is useful to you in some way.
