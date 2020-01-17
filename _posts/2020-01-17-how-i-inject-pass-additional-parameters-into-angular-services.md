---
layout: post
hidden: false
title: How I Inject/Pass Additional Parameters Into Angular Services
tags: []
---
I haven't had a use case before for passing additional parameters to an Angular service so didn't look much into it. Also I've seen @Inject used in a lot of codebases but only understood today how to properly use it.

If you want to pass additional parameters to your Angular service, what you are looking for is @Inject. It helps you pass your parameters to the service through Angular's dependency injection mechanism.

https://stackoverflow.com/questions/42396804/how-to-write-a-service-that-requires-constructor-parameters

Let's say we're writing a service which requires the id of a specific element in your html template, say a div. 

```typescript
import {Inject, Injectable} from '@angular/core';


@Injectable({
  providedIn: 'root'
})
export class RecaptchaService {
    constructor (private recaptchaContainerId: string) { }
}
```

And our component:

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

How do we go about properly passing in the id to the service? Here's how.

1. We make an injection token out of the service parameter as follows:

```typescript
import {Inject, Injectable} from '@angular/core';


@Injectable({
  providedIn: 'root'
})
export class RecaptchaService {
  constructor (
    @Inject('recaptchaContainerId') private recaptchaContainerId: string
  ) { }
}
```

Explain injection token here.....

2. We then provide this token to the service through our component's providers array as follows:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="recaptcha-container"></div>
  `,
   providers: [
    {provide: 'recaptchaContainerId', useValue: 'recaptcha-container'},
  ]
})
export class AppComponent {

  // We also inject our service
  constructor(private recaptchaService: RecaptchaService) { }

}
```

Explain providers array here...

And that's it! You've successfully passed an additional parameter to an Angular service.
