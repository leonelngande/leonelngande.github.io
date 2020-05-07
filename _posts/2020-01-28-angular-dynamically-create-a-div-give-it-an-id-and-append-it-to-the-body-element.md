---
layout: post
hidden: false
title: >-
  Angular: Dynamically Create a Div, Give it an ID, and Append it to the Body
  Element
tags:
  - Angular
  - Renderer2
  - RendererFactory2
author: Leonel Elimpe
---
Here's how to dynamically create a `div`, set it's `id` property, and append it to the `body` element in an Angular service or component. I'll use the example of creating a recaptcha container div on the fly.

For a service, we'll use we use [RendererFactory2](https://angular.io/api/core/RendererFactory2) to get a [Renderer2](https://angular.io/api/core/Renderer2) instance. When injecting Renderer2 directly into the service I got a null injector error, [this](https://github.com/angular/angular/issues/17824#issuecomment-351961146) Github issue sheds more light.

```typescript
import {Injectable, Renderer2, RendererFactory2} from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class RecaptchaService {
  private renderer: Renderer2;
  constructor () {
    // Get an instance of Angular's Renderer2
    this.renderer = this.rendererFactory.createRenderer(null, null);
  }

  createRecaptchaContainer() {
    // Use Angular's Renderer2 to create the div element
    const recaptchaContainer = this.renderer.createElement('div');
    // Set the id of the div
    this.renderer.setProperty(recaptchaContainer, 'id', 'recaptcha-container');
    // Append the created div to the body element
    this.renderer.appendChild(document.body, recaptchaContainer);

    return recaptchaContainer;
  }
}
```

For a component, you can directly inject a Renderer2 instance and proceed to create and append the div element.

```typescript
import {Injectable, Renderer2, RendererFactory2} from '@angular/core';

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html',
  styleUrls: ['./example.component.scss'],
})
export class ExampleComponent {

  constructor (private renderer: Renderer2) { }

  createRecaptchaContainer() {
    // Use Angular's Renderer2 to create the div element
    const recaptchaContainer = this.renderer.createElement('div');
    // Set the id of the div
    this.renderer.setProperty(recaptchaContainer, 'id', 'recaptcha-container');
    // Append the created div to the body element
    this.renderer.appendChild(document.body, recaptchaContainer);

    return recaptchaContainer;
  }
}
```

[Here's](https://alligator.io/angular/using-renderer2/) a good introductory post on using Angular's Renderer2. You can equally have a look at the API docs [here](https://angular.io/api/core/Renderer2).

For the `document.body` reference, here's it's [source](https://stackoverflow.com/a/43552385/6924437).
