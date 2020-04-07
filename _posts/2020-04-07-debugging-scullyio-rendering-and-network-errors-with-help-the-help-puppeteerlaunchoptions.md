---
layout: post
hidden: false
title: >-
  Debugging ScullyIO Rendering and Network Errors with help the help
  puppeteerLaunchOptions
tags: []
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

  puppeteerLaunchOptions: {
    slowMo: 4000,
    devtools: true,
  },
};
```