<!-- pages/map.vue -->
<template>
  <div id="map"></div>
</template>

<script>
import boroBound from '~/static/borough-boundaries.json';
import population from '~/static/population.json';

import { distance, booleanPointInPolygon, bbox, point, polygon } from '@turf/turf';


export default {
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      mapboxgl.accessToken = 'pk.eyJ1IjoiaW5rY2hlcnJ5IiwiYSI6ImNsbXBueGx3YjBtZ2EybG8zZXR1ZXlsbGgifQ.-kmLDhW0-rqfE-2FrTnlfQ';


      const map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/dark-v11',
        center: [-73.9712488, 40.7830603], // Longitude first, then Latitude
        zoom: 12,
        pitch: 45,
      });

      function randomPointInPolygon(numPairs) {
        const points = [];
        const generatedIds = new Set();

        function getRandomId() {
          return Math.floor(Math.random() * 1000000);  // Adjust the range as necessary
        }

        function generateUniqueId() {
          let id = getRandomId();
          while (generatedIds.has(id)) {
            id = getRandomId();
          }
          generatedIds.add(id);
          return id;
        }

        while (points.length < numPairs * 2) { // because we're generating pairs
          const lng1 = Math.random() * (maxLng - minLng) + minLng;
          const lat1 = Math.random() * (maxLat - minLat) + minLat;
          const point1 = point([lng1, lat1]);

          const lng2 = Math.random() * (maxLng - minLng) + minLng;
          const lat2 = Math.random() * (maxLat - minLat) + minLat;
          const point2 = point([lng2, lat2]);

          var polygonSourceId = 'bounds';
  
          // Query the rendered features under the point to see if it is within the polygon
          var features1 = map.queryRenderedFeatures(point1, { layers: [polygonSourceId] });
          var features2 = map.queryRenderedFeatures(point2, { layers: [polygonSourceId] });

          if (features1.length && features2.length) {
            const distanceBetween = distance(point1, point2, { units: 'miles' });


            const id1 = generateUniqueId();
            const id2 = generateUniqueId();

            point1.properties = {
              id: id1,
              twinId: id2,
              distance: distanceBetween,
              info: "Information for Point 1"
            };
            point2.properties = {
              id: id2,
              twinId: id1,
              distance: distanceBetween,
              info: "Information for Point 2"
            };

            points.push(point1);
            points.push(point2);
          }
        }

        return points;
      }

      map.on('load', function() {

        const bounds = boroBound.features.flatMap(boro => boro.geometry.coordinates.flatMap(poly => poly));
        console.log(bounds);
        const boroMask = {
          "type": "FeatureCollection",
          "features": [
            {
              "type": "Feature",
              "properties": {},
              "geometry": {
                "type": "Polygon",
                "coordinates": [
                  [
                    [-180, -90],
                    [-180, 90],
                    [180, 90],
                    [180, -90],
                    [-180, -90]
                  ],
                  ...bounds
                ]
              }
            }
          ]
        };

        // const randomPoints = randomPointInPolygon(100);

        // add sources
        map.addSource('bounds', {
          type: 'geojson',
          data: {
            "type": "FeatureCollection",
            "features": [
              {
                "type": "Feature",
                "properties": {},
                "geometry": {
                  "type": "Polygon",
                  "coordinates": bounds
                }
              }
            ]
          },
        });
        map.addSource('boroMask', {
          type: 'geojson',
          data: boroMask,
        });

        // const randomPointsGeoJSON = {
        //   type: 'FeatureCollection',
        //   features: randomPoints,
        // };

        // map.addSource('randomPoints', {
        //   type: 'geojson',
        //   data: randomPointsGeoJSON,
        // });

        // map.addLayer({
        //   id: 'portalLayer',
        //   type: 'circle',
        //   source: 'randomPoints',
        //   paint: {
        //     'circle-radius': 5,
        //     'circle-color': '#007cbf',
        //   }
        // });

        map.addLayer({
          id: 'boro-outline',
          type: 'line',
          source: 'bounds',
          layout: {},
          paint: {
            'line-color': '#f00',
            'line-opacity': 0.5,
            'line-width': 3
          }
        });

        map.addLayer({
          'id': 'boro-fill',
          'type': 'fill',
          'source': 'boroMask',
          'layout': {},
          'paint': {
            'fill-color': '#000',
            'fill-opacity': 0.5
          }
        });

        map.addLayer({
          'id': '3d-buildings',
          'source': 'composite',
          'source-layer': 'building',
          'filter': ['==', 'extrude', 'true'],
          'type': 'fill-extrusion',
          'minzoom': 15,
          'paint': {
            'fill-extrusion-color': '#aaa',
            'fill-extrusion-height': ['get', 'height'],
            'fill-extrusion-base': ['get', 'min_height'],
            'fill-extrusion-opacity': 0.6
          }
        });

        const textLabelLayers = [
          'country-label', 'state-label', 'place-city', 'place-town', 'road-label', 'poi-label', 'waterway-label', 'continent-label', 'settlement-major-label', 'settelment-minor-label',
        ];

        textLabelLayers.forEach(layerId => {
          if (map.getLayer(layerId)) {
            map.setLayoutProperty(layerId, 'visibility', 'none');
          }
        });
      });

      map.on('click', 'portalLayer', function(e) {
        const clickedId = e.features[0].properties.id;
        const twinId = e.features[0].properties.twinId;

        // Set the color of the clicked point and its twin to white, others to '#007cbf'
        map.setPaintProperty('portalLayer', 'circle-color', [
          'match',
          ['get', 'id'],
          clickedId, '#F00',
          twinId, '#F00',
          '#007cbf' // default color for the rest
        ]);

        console.log(e.features[0].properties);
      });


    }
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
#map {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
}
</style>