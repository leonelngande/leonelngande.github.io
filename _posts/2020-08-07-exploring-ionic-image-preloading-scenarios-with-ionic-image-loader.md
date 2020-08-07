---
layout: post
hidden: false
title: Exploring Ionic Image Preloading Scenarios with ionic-image-loader
author: Leonel Elimpe
excerpt: "[ionic-image-loader](https://github.com/zyra/ionic-image-loader) is an Ionic 2+ component that loads images in a background thread and caches them for later use. It also makes available a service to manually preload images, which will be the focus of this post."
tags:
  - Ionic
last_modified_at: 2020-08-07 04:58:31
---
## Introduction

[ionic-image-loader](https://github.com/zyra/ionic-image-loader) is an Ionic 2+ component that loads images in a background thread and caches them for later use. It also makes available a service to manually preload images, which will be the focus of this post.

<br>

## Setup

For demonstrations, we'll be preloading images on `City` models with the below interface.

```typescript
export interface City {
  id: number;
  name: string;
  description: string;
  elevationMap: string;
  image: string;
  latitude: number;
  longitude: number;
}
```

`image` and `elevationMap` are properties whose values are links to images we'll be preloading. Here's a sample City object:

```json
{
    "id": 1,
    "name": "Buea",
    "description": "The city of legendary hospitality",
    "elevationMap": "https://www.floodmap.net/Elevation/ElevationMap/Maps/?gz=2233410_12",
    "image": "https://upload.wikimedia.org/wikipedia/commons/6/68/Buea_from_Fako.jpg",
    "latitude": 4.1560,
    "longitude": 9.2632
}
```

We'll equally be using a service which injects `ionic-image-loader`'s `ImageLoaderService`.

```
ng generate service services/image-preload
```

```typescript
import {Injectable} from '@angular/core';
import {ImageLoaderService} from 'ionic-image-loader';

@Injectable({
  providedIn: 'root',
})
export class ImagePreloadService {
    
  constructor(
    private imageLoaderService: ImageLoaderService,
  ) { }
    
}
```

<br>

## Scenarios

### 1. Preload and wait, render only the preloaded image URLs

In this scenario, we want to preload all images and render the preloaded image URLs. This means we'll have to wait for all the images to preload before we begin rendering the images. For one of my use cases, the app is supposed to work fully offline and so it fetches the records (cities), preloads all the image URLs, then saves the cities for offline use (after swapping remote image URLs with local ones) with the Ionic Storage plugin.

The code below doesn't do that specifically, but demonstrates how to fetch records, preload all image URLs in the records, replace each remote image URL with the preloaded local URL, and wait for all images to preload before returning the result. It goes without saying that you may want to display a spinner if the images are many.

```typescript
@Component({
    selector: 'app-cities',
    template: `
        <ng-container *ngFor="let city of cities$ | async">
            <img-loader [src]="city.image" useImg *ngIf="city.image"></img-loader>
            <img-loader [src]="city.elevationMap" useImg *ngIf="city.elevationMap"></img-loader>
        </ng-container>
    `,
})
export class CitiesComponent implements OnInit {

    cities$: Observable<City[]>;

    constructor(
        private imagePreloadService: ImagePreloadService,
        private cityService: CityService
    ) { }

    ngOnInit() {
        this.cities$ = this.cityService.fetchAll()
            .pipe(
                switchMap((cities: City[]) => {
                    return this.imagePreloadService.preload<City>(
                        cities,
                        ['image', 'elevationMap']
                    );
                })
            );
    }

}
```

```typescript
// ...

export class ImagePreloadService {
    
  // ...

  async preload<T>(preloadables: T[], preloadableProperties: Array<keyof T>): Promise<T[]> {
    const promises = [];
    preloadables.forEach(preloadable => {
      preloadableProperties.forEach((property) => {
        promises.push(
            this.imageLoaderService.preload(preloadable[property as string])
                .then(localUrl => {
                  preloadable[property as string] = localUrl;
                })
                .catch(e => {
                  console.log(`Failed to preload image ${preloadable[property]}`);
                })
        );
      });
    });
    await Promise.all(promises);

    return preloadables;
  }
}
```

<br>

### 2. Preload but render the page while waiting for images to be preloaded in the background

With this scenario, we don't want to wait for all images to preload before rendering the page, so we preload each image asynchronously and go ahead to render the page. You can [configure](https://github.com/zyra/ionic-image-loader#advanced-usage) the image loader plugin to show a loader while the image is being preloaded.

Let's see some code for this scenario, we'll re-use what's written in the first example, but adapted for this scenario, mostly at the level of ImagePreloadService.

```typescript
@Component({
    selector: 'app-cities',
    template: `
        <ng-container *ngFor="let city of cities$ | async">
            <img-loader [src]="city.image" useImg *ngIf="city.image"></img-loader>
            <img-loader [src]="city.elevationMap" useImg *ngIf="city.elevationMap"></img-loader>
        </ng-container>
    `,
})
export class CitiesComponent implements OnInit {

    cities$: Observable<City[]>;

    constructor(
        private imagePreloadService: ImagePreloadService,
        private cityService: CityService
    ) { }

    ngOnInit() {
        this.cities$ = this.cityService.fetchAll()
            .pipe(
                switchMap((cities: City[]) => {
                    return this.imagePreloadService.preload<City>(
                        cities,
                        ['image', 'elevationMap']
                    );
                })
            );
    }

}
```

```typescript
// ...

export class ImagePreloadService {
    
  // ...

  preload<T>(preloadables: T[], preloadableProperties: Array<keyof T>) {
    preloadables.forEach(preloadable => {
      preloadableProperties.forEach((property) => {
          this.imageLoaderService.preload(preloadable[property as string])
              .catch(e => {
                  console.log(`Failed to preload image ${preloadable[property]}`);
              });
      });
    });
  }
}
```

<br>

## Conclusion

I hope this was clear and concise enough. Can you think of another scenario? Do leave a comment and I'll add a section for it.
