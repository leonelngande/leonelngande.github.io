---
layout: post
hidden: false
title: Using a directive to render Google Maps directions in Angular
image: {}
tags:
  - Angular
  - Directive
  - Google Maps
  - AGM
  - Directions Service
  - Directions Renderer
  - Waypoints
---
A few months back while working on a location based taxi calling app, I had to build in functionality to render the route from the user's location to the selected or searched listing on the map. 

The Angular project used Sebastian Holstein's open source [angular-google-maps](https://github.com/SebastianM/angular-google-maps) package for maps functionality. For installation instructions, visit the [getting started page](https://angular-maps.com/guides/getting-started/).

To make it possible to add the directions render to any other google map I worked with, I wrote this directive.

```
import {Directive, Input, OnChanges, OnInit, SimpleChanges} from '@angular/core';
import {GoogleMapsAPIWrapper} from '@agm/core';

// You can use any other interface for origin and destination, but it must contain latlng data
export interface ILatLng {
  latitude: number;
  longitude: number;
}

// the will keep typescript from throwing errors w.r.t the google object
declare var google: any;

@Directive({
  selector: '[appDirectionsMap]'
})
export class DirectionsMapDirective implements OnInit, OnChanges {
  @Input() origin: ILatLng;
  @Input() destination: ILatLng;
  @Input() showDirection: boolean;

  // We'll keep a single google maps directions renderer instance so we get to reuse it.
  // using a new renderer instance every time will leave the previous one still active and visible on the page
  private directionsRenderer: any;

  // We inject AGM's google maps api wrapper that handles the communication with the Google Maps Javascript
  constructor(private gmapsApi: GoogleMapsAPIWrapper) {}

  ngOnInit() {
    this.drawDirectionsRoute();
  }

  drawDirectionsRoute() {
      this.gmapsApi.getNativeMap().then(map => {
        if (!this.directionsRenderer) {
          // if you already have a marker at the coordinate location on the map, use suppressMarkers option
          // suppressMarkers prevents google maps from automatically adding a marker for you
          this.directionsRenderer = new google.maps.DirectionsRenderer({suppressMarkers: true});
        }
        const directionsRenderer = this.directionsRenderer;

        if ( this.showDirection && this.destination ) {
            const directionsService = new google.maps.DirectionsService;
            directionsRenderer.setMap(map);
            directionsService.route({
                origin: {lat: this.origin.latitude, lng: this.origin.longitude},
                destination: {lat: this.destination.latitude, lng: this.destination.longitude},
                waypoints: [],
                optimizeWaypoints: true,
                travelMode: 'DRIVING'
            }, (response, status) => {
                if (status === 'OK') {
                    directionsRenderer.setDirections(response);
                    // If you'll like to display an info window along the route
                    // middleStep is used to estimate the midpoint on the route where the info window will appear
                    // const middleStep = (response.routes[0].legs[0].steps.length / 2).toFixed();
                    // const infowindow2 = new google.maps.InfoWindow();
                    // infowindow2.setContent(`${response.routes[0].legs[0].distance.text} <br> ${response.routes[0].legs[0].duration.text}  `);
                    // infowindow2.setPosition(response.routes[0].legs[0].steps[middleStep].end_location);
                    // infowindow2.open(map);
                } else {
                    console.log('Directions request failed due to ' + status);
                }
            });
        }

      });
  }

  ngOnChanges(changes: SimpleChanges) {
      if (changes.destination || changes.showDirection) {
          // this checks if the show directions input changed, if so the directions are removed
          // else we redraw the directions
          if (changes.showDirection && !changes.showDirection.currentValue) {
              if (this.directionsRenderer !== undefined) { // check this value is not undefined
                  this.directionsRenderer.setDirections({routes: []});
                  return;
              }
          } else {
              this.drawDirectionsRoute();
          }
      }
  }

}
```

_directions-map.directive.ts_

That looks like a lot, but i've put in comments where necessary to guide you. The directive takes in as input a starting point address (`origin`), the `destination` address, and a boolean that determines whether a route is drawn for the origin and destination or not (`showDirection`).

Here's an example usage with an agm map:

```
<agm-map appDirectionsMap [showDirection]="this.displayDirections"
         [origin]="origin"
         [destination]="destination"

         [zoom]="zoom"
         [disableDefaultUI]="false"
         [zoomControl]="false">

    <agm-marker
            [latitude]="origin.latitude"
            [longitude]="origin.longitude">
    </agm-marker>
</agm-map>
```

_map.component.html_

```
import {Component} from '@angular/core';
import {ILatLng} from '../../directives/directions-map.directive';

@Component({
    selector: 'app-map',
    templateUrl: './map.component.html',
    styleUrls: ['./map.component.css']
})
export class MapComponent {

  // Washington, DC, USA
  origin: ILatLng = {
    latitude: 38.889931,
    longitude: -77.009003
  };
  // New York City, NY, USA
  destination: ILatLng = {
    latitude: 40.730610,
    longitude: -73.935242
  };
  displayDirections = true;
  zoom = 14;
}
```

_map.component.ts_

So there you go. Unfortunately I can't set up a live example given it'll require a google maps api key which now requires billing enabled by default. But if you have any questions or found a bug, or even a better implementation, please do post in the comments section.

Happy coding!
