---
layout: post
hidden: false
title: >-
  Configure Proxy Redirect with Scully - The Static Site Generator for Angular
  apps
tags:
  - ScullyIO
  - Angular
---
[Scully](https://github.com/scullyio/scully) is the awesome new static site generator for Angular, and I've been working on integrating it into [Xamcademy](https://xamcademy.com/courses).

The biggest hurdle so far has been the documentation, given it's taken a lot of trial and error to get things working as expected. Also most content (blog posts) I've come across lean more towards the blogging aspect of Scully, and not using it with an existing Angular application. I'll be writing more blog posts on my experience as I continue to use it.

To point Scully to your proxy configuration file, add the following entry to your `scully.config.js` file:

```javascript
exports.config = {
  projectRoot: "./src",
  projectName: "xamcademy",
  outDir: './dist/static',
  
  proxyConfig: './src/proxy.conf.json', // <---- ADD THIS
  
  // ...
};

```

If your proxy configuration file is at the root of your project instead, you can change the relative path to:

```javascript
exports.config = {
  projectRoot: "./src",
  projectName: "xamcademy",
  outDir: './dist/static',
  
  proxyConfig: './proxy.conf.json', // <---- ADD THIS
  
  // ...
};
```

I got a hint after looking at the `outDir` value, `./dist/static`, which points to a relative path from the root `./`.

With this set up, Scully will now correctly pass your proxy configuration file to your Angular app while pre-rendering it.

You can read the docs [here](https://github.com/scullyio/scully/blob/master/docs/scully-configuration.md#proxyconfig), and if you have a specific proxy redirect issue you'd like to discuss, do reach out to me on [Twitter](https://twitter.com/leonelngande) or leave a comment below.

Happy coding! 🙂