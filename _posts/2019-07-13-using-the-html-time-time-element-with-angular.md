---
layout: post
hidden: false
published: false
title: Using the html <time></time> element with Angular
tags:
  - Angular
  - html time
---
> <time \[dateTime]="someTime">{{ someTime }}</time>

So had to use the html time element today in an Angular component template and ran into this error:

`ERROR Error: Uncaught (in promise): Error: Template parse errors:`

`Can't bind to 'datetime' since it isn't a known property of 'time'.` 

Here's what my template looked like:

```typescript
<time datetime="{{comment.updated_at}}">
  {{ comment.updated_at }}
</time>
```

I was baffled, I confirmed on [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/time) that the time element has a datetime attribute. Further investigation brought me to [this issue in the Angular repository](https://github.com/angular/angular/issues/26255).

There [Alexey Zuev](https://twitter.com/yurzui?lang=en) explains:

I would say that Angular properly uses time tag.

<time> tag with datetime attribute should look like:

```typescript
<time [attr.datetime]="time">{{time}}</time>
```

while <time> tag with dateTime property is:

```typescript
<time [dateTime]="time">{{time}}</time>
```

So the take-way point here is the difference between the attribute name and the property name as illustrated by [Pawel Kozlowski](https://twitter.com/pkozlowski_os?lang=en) (a comment on the issue mentioned above):

* attribute name is `datetime`: [https://developer.mozilla.org/en-US/docs/Web/HTML/Element/time
  ](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/time)
* property name (default binding in Angular) is `dateTime`: <https://developer.mozilla.org/en-US/docs/Web/API/HTMLTimeElement>

So here's how my template ended up looking:

```typescript
<time [dateTime]="comment.updated_at">
  {{ comment.updated_at }}
</time>
```
