---
layout: post
hidden: false
title: Netlify build command for an Ionic Angular app
author: Leonel Elimpe
tags:
  - Netlify
  - Ionic
  - Angular
last_modified_at: 2020-09-10 09:29:30
---
In the `Build command` input field, enter `ng build --prod`, and in the `Publish directory` input field, enter `www`.

![Ionic build command on Netlify](/images/uploads/ionic-build-command-on-netlify.png)

Here, `ng build --prod` works similar to `ionic build --prod` and will output your Ionic app build to the `www` directory. This directory is outputted to because it is the build `outputPath` specified in `angular.json` of your Ionic app, unless you changed this default setting, in which case you enter the updated output directory name in the `Publish directory` field.

![angular.json outputPath field](/images/uploads/netlify-ionic-build-output-path.png)

If you came here after trying to use `ionic build --prod`, then know I was in the same situation, twice.

Here's [a post on the Netlify Community site](https://community.netlify.com/t/ionic-app-support/14651) indicating built-in support for the `ionic build` command is still lacking, it may be added in the future.