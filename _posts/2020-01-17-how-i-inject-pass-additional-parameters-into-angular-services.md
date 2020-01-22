---
layout: post
hidden: false
title: Passing Additional Parameters To An Angular Service
tags:
  - Angular
  - Dependency Injection
  - Injection Token
---
If you want to pass additional parameters to your Angular service, what you are looking for is @Inject decorator. It helps you pass your parameters to the service through Angular's dependency injection mechanism.

> `@Inject()` is a *manual* mechanism for letting Angular know that a *parameter* must be injected.
>
> \-- <cite>[Rangle.io](https://angular-2-training-book.rangle.io/di/angular2/inject_and_injectable#inject)</cite>

Let's say we're writing a recaptcha service which requires the id of the recaptcha container in your html template, as below. 

```typescript
import {Inject, Injectable} from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class RecaptchaService {
    constructor (private recaptchaContainerId: string) { }
}
```

Our example component:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="recaptcha-container"></div>
  `,
})
export class AppComponent {

  constructor() { }

}
```

How do we go about properly passing in the `recaptchaContainerId` to the service? Here's how.

<p>&nbsp;</p>

1. We make an [injection token](https://angular.io/api/core/InjectionToken) out of the `recaptchaContainerId` parameter as follows:

```typescript
import {Inject, Injectable} from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class RecaptchaService {
  constructor (
    // Make recaptchaContainerId an injection token
    @Inject('recaptchaContainerId') private recaptchaContainerId: string
  ) { }
}
```

> Injection tokens are a feature of Angular that allows the injection of values that don't have a runtime representation. What we mean by this, is that you can't inject something like an interface as it only exists as a TypeScript construct, not JavaScript. Injection tokens provide a simple mechanism to link a token, to a value, and have that value injected into a component.
>
> \-- <cite>[Inversion of Control](https://www.inversionofcontrol.co.uk/injection-tokens-in-angular/)</cite>

<p>&nbsp;</p>

2. We then provide this token to the service through our component's providers array as follows:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-example',
  template: `
    <div id="recaptcha-container"></div>
  `,
   providers: [
     // Provide a value to be used by Angular's dependency injection
     // machanism to pass 
    {provide: 'recaptchaContainerId', useValue: 'recaptcha-container'},
  ]
})
export class ExampleComponent {

  // We also inject our service
  constructor(private recaptchaService: RecaptchaService) { }

}
```

Note that I've deliberately limited the scope of the provided `recaptchaContainerId` to this component due to the nature of such a token (an element id). In another component where RecaptchaService is injected, I might be using a different id.

It's still very ok to provide the recaptchaContainerId token in the above component's module or the root AppModule. More [here](https://angular.io/guide/providers#limiting-provider-scope-with-components).

And that's it! We've successfully passed an additional parameter to an Angular service. Found something that can be improved? Please feel free to use the comment :-).

This [Stackoverflow question](https://stackoverflow.com/questions/42396804/how-to-write-a-service-that-requires-constructor-parameters) served as a valuable guide in writing this blog post.
