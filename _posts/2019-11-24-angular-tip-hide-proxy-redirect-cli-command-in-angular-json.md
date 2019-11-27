---
layout: post
hidden: false
title: 'Angular Tip: Hide Proxy Redirect CLI Command In angular.json'
tags:
  - Angular
  - Proxy Redirect
  - Angular CLI
author: Leonel Elimpe
---
Wouldn't it be great if instead of doing `ng serve --proxy-config proxy.conf.js` you simply use `ng serve` and the Angular CLI takes care of the proxy redirect command? Well, you can.

In the CLI configuration file, `angular.json`, add the `proxyConfig` option to the serve target:

```
...
"architect": {
  "serve": {
    "builder": "@angular-devkit/build-angular:dev-server",
    "options": {
      "browserTarget": "your-application-name:build",
      "proxyConfig": "src/proxy.conf.js"
    },
...
```

Now we can call `ng serve` to run the dev server with this proxy configuration, neat 🙂.

You can read more on this in the [Angular Docs](https://angular.io/guide/build#proxying-to-a-backend-server), happy coding!
