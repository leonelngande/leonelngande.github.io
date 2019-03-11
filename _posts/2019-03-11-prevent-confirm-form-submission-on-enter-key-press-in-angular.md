---
layout: post
hidden: false
title: Prevent/Confirm Form Submission on Enter Key Press in Angular
tags: []
---
_**Note:**_ _Don't do this unless absolutely necessary as this is a default form behaviour. I used this because I had a large contact creation form and allowing this led to the premature creation of records._

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
