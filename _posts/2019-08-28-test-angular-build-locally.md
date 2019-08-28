---
layout: post
hidden: false
title: Test Angular Build Locally
tags:
  - Angular
---
To test your Angular build locally:

1. Build your app: `ng build --prod`
2.  Install http-server for serving the app: `npm i -g http-server`
3. [cd](https://en.wikipedia.org/wiki/Cd_(command)) (change directory) into the the build location and run the app with: `http-server`
4. Open http-server url appending /index.html to it, should look something like this `http://127.0.0.1:8080/index.html` 

🎉
