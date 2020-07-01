---
layout: post
hidden: false
title: Add your Proxy Configuration File to angular.json - and forget it ever existed
author: Leonel Elimpe
image:
  path: /images/uploads/proxy-config-in-angular-dot-json.png
  thumbnail: /images/uploads/proxy-config-in-angular-dot-json.png
tags:
  - Angular
last_modified_at: 2020-07-01 09:31:00
---
What if you never have to write the --proxy-config CLI option again when serving your Angular application? 

```
ng serve --proxy-config proxy.conf.json
```

Hmmm, what a sweet thought.

Well, it is possible. Angular makes it possible to specify a proxy configuration file in `angular.json` to serve with your application. So when next you run `ng serve`, it's picked up automatically, no need typing `ng serve --proxy-config proxy.conf.json`.

To start using this, in the CLI configuration file, `angular.json`, add the `proxyConfig` option to the `serve` target:

```
...
"architect": {
  "serve": {
    "builder": "@angular-devkit/build-angular:dev-server",
    "options": {
      "browserTarget": "your-application-name:build",
      "proxyConfig": "src/proxy.conf.json"
    },
...
```

To run the dev server with this proxy configuration, call `ng serve`  ✌️.

<br>

## Further reading

- [Proxying to a backend server - angular.io](https://angular.io/guide/build#proxying-to-a-backend-server)

