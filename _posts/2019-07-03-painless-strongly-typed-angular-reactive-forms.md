---
hidden: false
title: Painless Strongly Typed Angular Reactive Forms
tags:
  - Angular
  - Strongly Typed Reactive Forms
  - Reactive Forms
author: Leonel Elimpe
---
A few days back while looking into strongly typing reactive forms in Angular, I came across [this post](https://alex-klaus.com/stongly-typed-angular-forms/) by [Alex Klaus](https://twitter.com/_AlexKlaus). Given reactive forms don't currently support strong typing (see issues [\#13721](https://github.com/angular/angular/issues/13721) and [\#17000](https://github.com/angular/angular/issues/17000)), he suggests making use of [Daniele Morosinotto](https://twitter.com/dmorosinotto) solution which involves leveraging Typescript declaration files (`*.d.ts`) .

> The most elegant solution I’ve seen was provided by Daniele Morosinotto, who suggested leveraging TypeScript declaration files (*.d.ts) to introduce generic interfaces extending the standard ones (e.g. AbstractControl, FormControl, etc.). It doesn’t introduce any new functionality and has no footprint in the compiled JavaScript, but at the same time enforcing strong type checking. Brilliant!

Adopting the solution is straightforward:

1. Download `TypedForms.d.ts` from [his gist](https://gist.github.com/dmorosinotto/76a9272b5c45af1f78a61e7894df5777) and save it as `src/typings.d.ts` in your project (Angular 6+ already knows how to use this file, see more in the [Angular docs](https://angular.io/guide/using-libraries#library-typings)).
2. Start using the new types (`FormGroupTyped<T>`, `FormControlTyped<T>`, etc.) whenever you need a strong type validation (see examples in [that gist](https://gist.github.com/dmorosinotto/76a9272b5c45af1f78a61e7894df5777)).

After importing the typings, my typed form declarations now looked like this:

```typescript
form: FormGroupTyped<MyTypedInterface>;
```

I however had an issue getting it to work with the `FormBuilder` service and [reached out](https://twitter.com/_AlexKlaus/status/1143318167189504000) to him. His suggestion was to just cast the resulted FormGroup to a typed one.

```typescript
this.form = this.formBuilder.group({
     field1: null,
     field2: null
}) as FormGroupTyped<MyTypedInterface>;
```

Which did the trick, however... now my IDE bring up an error in the html template file:

```html
<form novalidate [formGroup]="form" (submit)="submit(form.value)"></form>

<!-- Error message: Type FormGroupTyped<MyTypedInterface> is not assignable to type FormGroup -->
```

I can live with this for now until the Angular Team adds strong typing to Reactive Forms, given am aware of the cause. Please share your thoughts in the comments below, or on [Twitter](https://twitter.com/leonelngande).

Update September 4th, 2019:

I found  an easy workaround for the template error. Simple create a getter that returns the typed form cast as a FormGroup, which would cause Angular to complain no more in the template.

```typescript
  get untypedForm() {
    return this.form as FormGroup;
  }
```

```html
<form novalidate [formGroup]="untypedForm" (submit)="submit(form.value)"></form>

<!-- No more error message in form template -->
```
