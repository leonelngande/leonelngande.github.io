---
layout: post
hidden: false
title: Debugging ScullyIO Rendering and Network Errors with the help of
  puppeteerLaunchOptions
tags:
  - Angular
  - ScullyIO
author: Leonel Elimpe
---
With my Angular app's Scully build broken due to proxy redirect issues, I was very happy today discovering we can control the underlying mechanism of rendering the pages to some degree.

As per the Scully [documentation](https://github.com/scullyio/scully/blob/master/docs/scully-configuration.md#puppeteerlaunchoptions), we can add `puppeteerLaunchOptions` (a set of configurable options to set on the browser) to our Scully configuration file. You can find details on all the options [here](https://pptr.dev/#?product=Puppeteer&version=v2.0.0&show=api-puppeteerlaunchoptions).

To debug the proxy configuration issue affecting the rendering of my pages, two options in particular were super helpful:

* `slowMo`: Slows down Puppeteer operations by the specified amount of milliseconds. Useful so that you can see what is going on. And
* `devtools`: Whether to auto-open a DevTools panel for each tab.

Here's how to add these options to your `scully.config.js` file:

```javascript
exports.config = {
  projectRoot: "./src",
  projectName: "xamcademy",
  outDir: './dist/static',

  /* ADD THESE */
  puppeteerLaunchOptions: {
    slowMo: 4000,
    devtools: true,
  },
};
```

Of course we have to use the [\--showBrowser](https://github.com/scullyio/scully/blob/master/docs/scully-cmd-line.md#showbrowser) option with the scully build command so we see the effect of the puppeteer launch options.

So run your scully build command adding the showBrowser option: `npm run scully --showBrowser`. You'll now notice the addition of the Chrome devtools to each page scully is prerendering, as well as a noticeable slowdown of each page's rendering.

With this slowdown, you have enough time to switch to the network tab and inspect the requests, hopefully finding the cause of the errors.

Of course you can adjust the duration of the slowdown by increasing or reducing the value of `slowMo` above. That's it for this post, happy bug fixing!