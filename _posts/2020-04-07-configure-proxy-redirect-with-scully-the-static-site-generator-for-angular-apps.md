---
layout: post
hidden: false
title: Configure Proxy Redirect with Scully - The Static Site Generator for
  Angular apps
image:
  path: /images/uploads/scullyio-proxy-configuration-cover.png
  caption: ScullyIO proxy configuration cover
tags:
  - ScullyIO
  - Angular
---
* [Introduction](#introduction) 
* [Proxy Configuration with Scully](#proxy-configuration)

  * [Using the scully configuration file](#via-config) 
  * [Using the CLI](#via-cli)
* [Conclusion](#conclusion)

## <a name="introduction">Introduction</a>

[Scully](https://github.com/scullyio/scully) is the awesome new static site generator for Angular, and I've been working on integrating it into [Xamcademy](https://xamcademy.com/courses).

The biggest hurdle so far has been the unavailability of samples to follow for an existing Angular application, so it's taken a lot of trial and error to get things working as expected. I'll be writing more blog posts on my experience as I continue to use it.

## <a name="proxy-configuration">Proxy Configuration with Scully</a>

Scully uses the same config format as [webpackDevServer](https://webpack.js.org/configuration/dev-server/#devserverproxy), also used by the [Angular CLI](https://angular.io/guide/build#proxying-to-a-backend-server).  Here's an example proxy config file located at `src/proxy.conf.json` in an Angular project.

```javascript
{
  "/api": {
    "target": "http://localhost:8200",
    "secure": false,
    "pathRewrite": {"^/api": ""}
  }
}
```

What this means is we can take our Angular app's already existent proxy configuration file and pass it to Scully, ensuring requests are properly redirected as Scully prerenders our routes.

We can point Scully to our proxy configuration file in two ways.

### <a name="via-config">Using the scully configuration file</a>

The [proxyConfig](https://github.com/scullyio/scully/blob/master/docs/scully-configuration.md#proxyconfig) property can be added to `scully.config.js`, it's value being a relative filename for a proxy config file.

To point Scully to our proxy configuration file, we add the following entry to `scully.config.js`:

```javascript
exports.config = {
  projectRoot: "./src",
  projectName: "xamcademy",
  outDir: './dist/static',
  
  proxyConfig: './src/proxy.conf.json', // <---- ADD THIS
  
  // ...
};
```

If say our proxy config file is at the root of the project instead, we can change the relative path to:

```javascript
exports.config = {
  projectRoot: "./src",
  projectName: "xamcademy",
  outDir: './dist/static',
  
  proxyConfig: './proxy.conf.json', // <---- ADD THIS
  
  // ...
};
```

I got a hint after looking at the `outDir` value, `./dist/static`, which points to a relative path from the root `./`. [Here's](https://github.com/scullyio/scully/blob/1c8afa24baf3bbddcc3a27dffcec0790e353422f/scully/utils/serverstuff/proxyAdd.ts#L15) the code that loads the proxy config! Hopefully that link doesn't get broken 😑.

### <a name="via-cli">Using the CLI</a>

Our proxy config file can be passed in using the `proxy` [cli option](https://github.com/scullyio/scully/blob/2ecc2162fceb6e4f5846f28807f479c2c62d5f72/scully/utils/cli-options.ts#L38). It's aliases are `--proxy`, `--proxyConfig`, `--proxy-config`, and `--proxyConfigFile`. Below are usage examples.

```shell
npm run scully --proxy src/proxy.conf.json
```

```shell
npm run scully --proxyConfig src/proxy.conf.json
```

```shell
npm run scully --proxy-config src/proxy.conf.json
```

```shell
npm run scully --proxyConfigFile src/proxy.conf.json
```

If our proxy config file was located at the root of our project instead, we can adjust as follows (using the `scully:serve` command in this example).

```shell
npm run scully --proxy proxy.conf.json
```

```shell
npm run scully --proxyConfig proxy.conf.json
```

```shell
npm run scully --proxy-config proxy.conf.json
```

```shell
npm run scully --proxyConfigFile proxy.conf.json
```

**Note**: You should be aware a proxy config file passed through the command line has priority over one added in the scully configuration file, [more here](https://github.com/scullyio/scully/blob/1c8afa24baf3bbddcc3a27dffcec0790e353422f/scully/utils/serverstuff/proxyAdd.ts#L23).

## <a name="conclusion">Conclusion</a>

With this set up, Scully will now correctly use your proxy configuration to redirect requests while pre-rendering your app's routes.

You can read the docs [here](https://github.com/scullyio/scully/blob/master/docs/scully-configuration.md#proxyconfig), and if you have a specific proxy redirect issue you'd like to discuss, do reach out to me on [Twitter](https://twitter.com/leonelngande), or leave a comment below.

Happy coding! 🙂