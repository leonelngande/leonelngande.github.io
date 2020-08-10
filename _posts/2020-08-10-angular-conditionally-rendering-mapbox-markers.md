---
layout: post
hidden: false
title: "Angular: Conditionally Rendering Mapbox Markers"
author: Leonel Elimpe
tags:
  - Angular
  - Mapbox
last_modified_at: 2020-08-10 02:10:24
---
Say we're using Mapbox to render a map that contains a couple of icons representing places. When the map loads for the first time, no marker should be rendered.

![A mapbox map](/images/uploads/20200808_122418.jpg)

When a user clicks on an icon, we're supposed to move to that location and render a blue marker over it.

![A mapbox map with a blue marker](/images/uploads/20200808_122137.jpg)

When the user clicks on another icon, we're supposed to "move" the blue marker to the new icon.

One way to implement this is to keep track of the marker with a "global" variable. When we need to render the marker, we check if an instance of the marker already exists and destroy it if so, then create a new one and render it. If an instance does not exist, we create a new one and render it. Let's see some code, my example is in the context of an Angular application, but the logic can be applied anywhere.

```typescript
// ...

@Component({
  selector: 'app-mapbox',
  templateUrl: './mapbox.component.html',
  styleUrls: ['./mapbox.component.scss'],
})
export class MapboxComponent {
  // ...
  private map: mapboxgl.Map;
  private flyToMarker: mapboxgl.Marker;

  constructor() { }

  // ...

  async flyToPosition(location: LngLatLike, showMarker = false) {
    await this.map.flyTo({
      center: location,
    });
    if (showMarker) {
        this.removeFlyToMarker();
        this.flyToMarker = new mapboxgl.Marker({
            color: 'blue'
        });
        this.flyToMarker
            .setLngLat(location)
            .addTo(this.map);
    } else {
        this.removeFlyToMarker();
    }
  }

  private removeFlyToMarker() {
      if (this.flyToMarker) {
          this.flyToMarker.remove();
      }
  }
}
```

`flyToPosition` does exactly what we described above. It takes in a location (LatLng coordinates) and a boolean that indicates whether to render a marker at the location or not.

The first time the component is rendered, we call `flyToPosition` passing in only the default map coordinates. When the user clicks on a map icon (a place), we call `flyToPosition` passing in the coordinates of the place, and `true` so a marker is rendered.