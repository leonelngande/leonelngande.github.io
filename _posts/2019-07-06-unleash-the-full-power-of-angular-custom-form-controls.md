---
layout: post
hidden: false
title: Unleash the Full Power of Angular Custom Form Controls
tags: []
---
Have you watched the ngconf presentation by Kara Ericson on custom form controls? If not so, I implore you to have a look at it before continuing.

Custom form controls are a really powerful feature of the Angular Framework, with which you can make just about any component a form input. 

Personally, I've had quite a hard time fully comprehending them, most especially injecting the parent NgControl into the custom form control to get access to validation state, and if necessary add custom validators. That is until now.

If you're interested in creating a base class implementing ControlValueAccessor logic and extending your custom form controls from it, checkout this article I wrote on the topic.

We'll be creating a custom form control for question selection, a functionality I use in the [Xamcademy](https://www.xamcademy.com) admin area. Here's the question interface:

```typescript
export interface IQuestion {
  id: number;
  question: string;
}
```

For question selection, we'll make use of the native angular 6+ select component [ng-select](https://github.com/ng-select/ng-select).

The final output looks like this:

<---------------------------- ADD GIF OF WORKING QUESTION SELECTOR HERE ---------------------->

Let's begin by creating the question form.

```typescript

```

















What's next? Go even a step further by strongly typing your reactive forms, which can make things even easier for you given you'll be able to do stuff like:

```typescript
get question() {
  return this.form.question;
}

```
