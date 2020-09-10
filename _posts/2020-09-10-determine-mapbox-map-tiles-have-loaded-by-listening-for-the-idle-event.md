---
layout: post
hidden: false
title: Determine Mapbox map tiles have loaded by listening for the "idle" event
author: Leonel Elimpe
tags:
  - Mapbox
last_modified_at: 2020-09-10 10:29:43
---
```typescript
import {Component, OnInit} from '@angular/core';
import mapboxgl from 'mapbox-gl/dist/mapbox-gl.js';

// ...
export class MapboxComponent implements OnInit {

  private map: mapboxgl.Map;

  constructor() { }

  async ngOnInit() {
    this.map = await this.initializeMap();

    this.map.once('idle', async (e) => {
      /**
       * Do something now, given all currently requested tiles have loaded,
       * no camera transitions are in progress, and
       * all fade/transition animations have completed.
       */
    });
  }

  private async initializeMap(): Promise<mapboxgl.Map> {
    // Map instance is initialized here, etc
  }
}
```

The `idle` event is fired after the last frame rendered before the map enters an "idle" state. When this event is fired, we are sure of the following things:

- All currently requested tiles have loaded.
- No camera transitions are in progress.
- All fade/transition animations have completed.

I had a situation where I had to wait for all map tiles to load before proceeding, and after a while searching for the ideal map event came across `idle` in this [Stackoverflow answer](https://stackoverflow.com/a/54140160/6924437).

My example above is within an Angular component, and the most important bit is in the `ngOnInit` function where we listen once for the idle event. As explained in the above linked Stackoverflow answer, we can equally continuously listen for the idle event.

```javascript
    this.map.on('idle', (e) => {
        // do things every time the map idles
    });
```