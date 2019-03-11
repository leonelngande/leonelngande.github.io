---
layout: post
hidden: false
title: Prevent/Confirm Form Submission on Enter Key Press in Angular
tags: []
---
```
 <form novalidate [formGroup]="form" (keydown.enter)="confirmSubmit($event)" (submit)="create(form.value)"></form>
```



```
confirmSubmit($event) {
      if (! confirm('Submit form?')) {
          $event.preventDefault();
      }
    }
```
