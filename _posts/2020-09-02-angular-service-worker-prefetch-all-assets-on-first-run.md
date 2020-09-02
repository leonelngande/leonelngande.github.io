---
layout: post
hidden: false
title: "Angular Service Worker: Prefetch all assets on first run"
author: Leonel Elimpe
tags:
  - Angular
  - PWA
last_modified_at: 2020-09-02 10:17:05
---
I've been working on the PWA version of a mobile app and it was brought to my attention that when using the PWA offline, some icons that were placed in the `/src/assets` directory fail to load.

I've always assumed the service worker created for Angular PWAs automatically downloads and caches all assets on first run, that is all the image and icon files found in the `/src/assets` directory. It turns out that's not the case.

As described in the [Angular docs](https://angular.io/guide/service-worker-config#installmode), there are two `installMode`s for assets.

> The `installMode` determines how these resources are initially cached.

* `prefetch` tells the Angular service worker to fetch every single listed resource while it's caching the current version of the app. This is bandwidth-intensive but ensures resources are available whenever they're requested, even if the browser is currently offline.
* `lazy` does not cache any of the resources upfront. Instead, the Angular service worker only caches resources for which it receives requests. This is an on-demand caching mode. Resources that are never requested will not be cached. This is useful for things like images at different resolutions, so the service worker only caches the correct assets for the particular screen and orientation.

When Angular creates the service worker configuration for a project as `ngsw-config.json`, by default it set's the `installMode` for assets to `lazy`. What I want is `prefetch` such that resources in the assets directory are available whenever they're requested, even if the browser is currently offline.

So that's it, I changed `installMode` to `prefetch` and now all icons appear in offline mode after the first run of the PWA application.

```json
{
  // ...
    
  "assetGroups": [
    // ...
      
    {
      "name": "assets",
      "installMode": "prefetch",
      "updateMode": "prefetch",
      "resources": {
        "files": [
          "/assets/**",
          "/*.(eot|svg|cur|jpg|png|webp|gif|otf|ttf|woff|woff2|ani)"
        ]
      }
    }
  ]
}
```

You can read all about Service worker configuration in the Angular [documentation](https://angular.io/guide/service-worker-config#service-worker-configuration).