---
layout: post
hidden: false
title: Automatically Add a Protocol to a Url String in Angular
tags:
  - Angular
  - Url Protocol
---
This little utility function can help you easily determine if a url string contains a protocol and add one if missing.

```typescript
export const GetLocation = (href: string) => {
    const parser = document.createElement("a");
    parser.href = href;
    return parser;
};

export const AutoAddProtocol = (href: string) => {
    let hrefCopy = href;
    const parser = GetLocation(href);
    if (parser.protocol !== 'http:' || parser.protocol !== 'https:') {
        hrefCopy = 'http://' + href;
    }

    return hrefCopy;
};
```

(Functions courtesy of answers to [this](https://stackoverflow.com/questions/736513/how-do-i-parse-a-url-into-hostname-and-path-in-javascript) StackOverflow question).

`GetLocation` leverages native browser functionality to parse the url string and return it. `AutoAddProtocol` is then used to check if the url contains a protocol, adding `http://` as the protocol if one is not found. Of course you can equally use `https://` here.

FYI, this can only be used in a browser environment given we're leveraging native browser functionality.

A good use case for this functionality is stopping your application's base url from being prepended to external urls you use on anchor tags.

```typescript
<a href="www.mysite.com" target="_blank">My Site</a>
```

In the case above, your site url (say `https://siteurl.com`) will likely be prepended to the href address. So we end up with `https://siteurl.com/www.mysite.com` . 

Adding a protocol to the external url ascertains you get just `www.mysite.com` as the address.

Here's a usage sample.

```typescript
import {Component} from '@angular/core';
import {AutoAddProtocol} from 'helpers/auto-add-protocol';

@Component({
   selector: 'app-root',
   template: `<a [href]="autoAddProtocol('www.mysite.com')" target="_blank">My Site</a>`
})
export class AppComponent {
   autoAddProtocol = AutoAddProtocol;
}
```

Happy coding!
