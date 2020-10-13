---
layout: post
hidden: false
title: Native Platform Check Without Importing Capacitor
author: Leonel Elimpe
tags: []
last_modified_at: 2020-10-13 10:47:22
---
I wanted to be able to check the platform my app is running on without importing [Capacitor](https://capacitorjs.com), thereby improving my web build's bundle size. Capacitor already exposes the following methods for querying the current platform:

* `Capacitor.getPlatform();` // 'web', 'ios', or 'android'
* `Capacitor.platform;` // 'web', 'ios', or 'android'
* `Capacitor.isNative;` // true or false

These work as expected, my issue is to use any of these, one has to import `Capacitor`, which significantly increases my web build's bundle size.

```typescript
import { Capacitor } from '@capacitor/core';

const isNative = Capacitor.isNative;
```

I'd like to ideally be able to check the current platform without incurring this bundle size increase, especially when I just want to check the platform and I don't actually use Capacitor plugins etc. To fix this, I ended up using some code copied from [Ionic Framework's platform check source code on Github](https://github.com/ionic-team/ionic-framework/blob/master/core/src/utils/platform.ts).

```typescript
export const isNative = () => {
  return isCapacitorNative(window);
};

/**
 * @see https://github.com/ionic-team/ionic-framework/blob/master/core/src/utils/platform.ts
 */
const isCapacitorNative = (win: Window): boolean => {
  const capacitor = win['Capacitor'];
  return !!(capacitor && capacitor.isNative);
};
```

This still makes use of the `isNative` property on the `Capacitor` object to determine the platform, though here we don't explicitly import `Capacitor`. With Capacitor always included in native builds, this should always work as expected.

Here's a usage example.

```typescript
import {RouterModule, Routes} from '@angular/router';
import {NgModule} from '@angular/core';
import {isNative} from './helpers/is-native';

export const webRoutes: Routes = [
  // Use these routes if a web app
];

const nativeRoutes: Routes = [
  // Use these routes if a native app
];

const routes = isNative() ? nativeRoutes : webRoutes;

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {
}
```

<br>

You can check out the Ionic Framework platform check [source code](https://github.com/ionic-team/ionic-framework/blob/master/core/src/utils/platform.ts) linked above for more granular solutions to platform checking 🔥.