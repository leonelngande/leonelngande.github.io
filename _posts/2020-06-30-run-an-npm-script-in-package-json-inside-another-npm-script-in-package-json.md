---
layout: post
hidden: false
title: Run an npm script in package.json inside another npm script in package.json
author: Leonel Elimpe
tags:
  - Javascript
  - NPM
last_modified_at: 2020-06-30 01:45:33
---
This post might seem trivial but I did spend a good amount of time searching this online just to be sure. So here we are lol.

Say you have an npm script called  `"build"` inside package.json to build your application, and another script called `"deploy:staging"`  to deploy your application to your staging environment. You'd like the deploy script to run the build script before deploying, how do you do that? Well, just call the build script inside the deploy script as you'd do from the terminal: `npm run build`. See below.

```
{
  // ...
  
  "scripts": {
    // ...
    
    "build": "ng build",
    "deploy:staging": "npm run build && netlify deploy",
    
    // ...
  },
```

Notice `npm run build` being called inside the `deploy:staging` script before a deploy is triggered with the Netlify CLI.

<br>

## Further reading

[Helpers and Tips for NPM Run Scripts - Michael Kuehnel](https://michael-kuehnel.de/tooling/2018/03/22/helpers-and-tips-for-npm-run-scripts.html)