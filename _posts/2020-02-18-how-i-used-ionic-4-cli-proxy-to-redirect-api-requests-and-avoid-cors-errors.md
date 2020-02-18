---
layout: post
hidden: false
title: How I Used Ionic 4 CLI Proxy To Redirect API Requests And Avoid CORS Errors
tags:
  - Ionic
  - Angular
  - API Redirect
---
I recently refactored an Ionic Angular app that previously made use of jQuery to fetch data and update the view. Having moved the API calls into dedicated services, all API requests we're blocked by the browser given the different origins (localhost vs external api url).

I went ahead and created the file `src/proxy.conf.json` and updated `angular.json` to incorporate the new proxy settings (more [here](https://angular.io/guide/build#proxying-to-a-backend-server)).

```json
{
  "/api/*": {
    "target": "https://my-api-domain.com/api",
    "changeOrigin": true,
    "secure": false,
    "pathRewrite": {"^/api" : ""}
  }
}
// src/proxy.conf.json
```

```json
// ...
"architect": {
  "serve": {
    "builder": "@angular-devkit/build-angular:dev-server",
    "options": {
      "browserTarget": "app:build",
      "proxyConfig": "src/proxy.conf.json"
    },
// ...
// angular.json
```

I then adapted my service to the new proxy setting as below, replacing the previously used full api path with just /api).

```typescript
import { Injectable } from '@angular/core';
import {HttpClient} from '@angular/common/http';
import {JourneyName} from '../models/journey-name';
import {Observable} from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class JourneysService {

  constructor(private http: HttpClient) {}

  getJourneyName(journeyId: number): Observable<JourneyName> {
    const body = new FormData();
    body.append('journey', `${journeyId}`);

    return this.http.post(
        `/api/get-journey-name`,
        body
      ) as Observable<JourneyName>;
  }
}

```

To my surprise I still got the CORS error after testing. A couple more minutes searching Google and I found out Ionic has it's own CLI proxy - [Service Proxies](https://ionicframework.com/docs/v3/cli/configuring.html#service-proxies), meaning it doesn't use that of Angular under the hood. Ok.

Went ahead and configured it as follows inside `ionic.config.json`:

```json
{
  "name": "sampleAppName",
  "integrations": {
    "cordova": {}
  },
  "type": "angular",
  "id": "01aae245",
  "proxies": [
    {
      "path": "/api",
      "proxyUrl": "https://my-api-domain.com/api"
    }
  ]
}

```

Now it should work! :-) Ran `ionic serve`, didn't work, was still getting the CORS error as proxy redirect was for some reason not being triggered. Then tried `ionic serve --proxy-config src/proxy.conf.json`, still didn't work.

Did another Google search, found [this](https://stackoverflow.com/a/55185835/6924437) Stackoverflow answer which suggested using `ionic serve -- --proxy-config src/proxy.conf.json`, notice the extra dashes `--`. This worked!!

Thus I ended up passing the Angular proxy.conf.json to the `ionic serve` command. Will update this post should I find a way to make the proxy redirect work with the configuration in ionic.config.json, and if you do know how to, please leave a reply in the comment :-).

Also what exactly do the extra dashes `--` do? Will equally update the post when I find out, I believe I've come across something similar before.
