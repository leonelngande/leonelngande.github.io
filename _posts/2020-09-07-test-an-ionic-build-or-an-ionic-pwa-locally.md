---
layout: post
hidden: false
title: Test an Ionic Build, or an Ionic PWA locally
author: Leonel Elimpe
tags:
  - Ionic
  - PWA
last_modified_at: 2020-09-07 10:45:26
---
If you don't have [`http-server`](https://www.npmjs.com/package/http-server) installed, go ahead and install it globally.

```
npm install --global http-server
```

> `http-server` is a simple, zero-configuration command-line http server. It is powerful enough for production usage, but it's simple and hackable enough to be used for testing, local development, and learning.

Now run your production build (or development build if you prefer) and serve it with http-server as below.

```
ionic build --prod && http-server www/
```

Your app will be served at the URLs `http://192.168.1.9:8080`  and`http://127.0.0.1:8080` which you can open in your browser.

If you have a Service Worker installed in your app, you can equally test it's functioning using the above URL(s), however, note it'll only work if you did a production build.

If you redirect requests to a local server during development, you can use the `--proxy` option to pass in a proxy redirect URL. For example:

```
ionic build --prod && http-server www/ --proxy=http://localhost:9000/api
```

You can have a look at `http-server`'s [README](https://www.npmjs.com/package/http-server#available-options) for all available options.