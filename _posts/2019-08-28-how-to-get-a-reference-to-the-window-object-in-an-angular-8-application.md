---
layout: post
hidden: false
title: How to get a Reference to the Window Object in an Angular 8 Application
tags:
  - Angular
  - Window Object
---
There are many articles on the web showing various methods of getting a reference to the window object in Angular (primarily through the dependency injection mechanism). However those that are popular on Google search are from 2016, 2017, etc, and the methods are mostly overly complicated (understandably).

If you've had issues with these like me, there is a much easier way to do this.

Just provide the `Window` object directly as a service within your core/app module. Like this:

```typescript
providers: [
    { provide: Window, useValue: window }
]
```

Then you can just inject it directly anywhere you want - e.g. in a service:

```typescript
constructor(private window: Window) {
    // ...
}
```

_Credits_: I found this little gem [here](https://juristr.com/blog/2016/09/ng2-get-window-ref/#comment-4545454475).
