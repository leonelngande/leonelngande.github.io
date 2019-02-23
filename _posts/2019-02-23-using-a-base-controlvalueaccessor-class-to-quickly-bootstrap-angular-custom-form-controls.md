---
layout: post
hidden: false
title: >-
  Using a base ControlValueAccessor class to quickly bootstrap Angular custom
  form controls
tags:
  - Angular
  - ControlValueAccessor
  - Custom Form Control
---
Having created a good number of custom form controls in Angular, it always felt repetitive implementing the ControlValueAccessor interface in each custom component.

A few days back while pulling my hair out trying to make a custom form control component (control is a FormArray) work with a form, I came across this jewel by [Nick Arthur plays Life](https://medium.com/@narthur157/custom-angular-reactive-forms-what-i-wish-i-knew-v5-6-5b1c2f8e1974):

```
import { ControlValueAccessor } from '@angular/forms';

export class BaseControlValueAccessor<T> implements ControlValueAccessor {
  public disabled = false;

  /**
   * Call when value has changed programatically
   */
  public onChange(newVal: T) {}
  public onTouched(_?: any) {}
  public value: T;

  /**
   * Model -> View changes
   */
  public writeValue(obj: T): void { this.value = obj; }
  public registerOnChange(fn: any): void { this.onChange = fn; }
  public registerOnTouched(fn: any): void { this.onTouched = fn; }
  public setDisabledState?(isDisabled: boolean): void { this.disabled = isDisabled; }
}
```



The class above simply implements Angular's ControlValueAccessor interface, making it possible to extend it in your custom form control component and not have to worry about writing the accessor methods.

You equally have to pass in (when extending) the type of value (generic `T`) your custom form control works with, `string`, `number`, `array`, etc. This makes sure the proper value is passed in and written to the input.

```
public writeValue(obj: T): void { this.value = obj; }
```

Here's a sample usage:

```
import { Component, forwardRef } from '@angular/core';
import {NG_VALUE_ACCESSOR} from '@angular/forms';
import {BaseControlValueAccessor} from '../../BaseControlValueAccessor';

@Component({
  selector: 'check',
  template: `<div (click)="toggle()" [ngClass]="{'check-selected': value}"></div>`,
  styles: [`
    div {
      width: 24px;
      height: 24px;
      border: 1px solid black; 
      margin-right: 24px;
      display: inline-block;
    }

    .check-selected {
      background-color: black;
    }
  `],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => CheckComponent),
      multi: true
    }
  ]
})
export class CheckComponent extends BaseControlValueAccessor<boolean> {
  toggle() {
    this.value = !this.value;
    this.onChange(this.value);
  }
}
```

Don't forget to call `super()` in your custom form control component's constructor.

```
constructor(private formBuilder: FormBuilder) {
    super();
}
```

Make sure to check out the [link](https://medium.com/@narthur157/custom-angular-reactive-forms-what-i-wish-i-knew-v5-6-5b1c2f8e1974) and the [stackblitz](https://stackblitz.com/edit/angular-custom-reactive-forms?embed=1&file=src/app/app.component.ts) linked in it, and if you have any doubts, please mention in the comments section.

Happy coding!
