---
layout: post
title: "Using Valhalla for Routing in Mapbox"
date: 2025-01-25 00:00:00 -0700
tags: javascript react
---

![Valhalla Routing Example](/assets/images/valhalla-routing-example.png)

[Valhalla](https://github.com/valhalla/valhalla) is the MIT licensed open
source routing engine used by Mapbox and Tesla. In this blog post, I'll run
Valhalla from docker compose to provide routing information to a Typescript
React frontend.

## Docker Configuration

First we'll create a `custom_files/` directory in our project directory for
the Open Street Map PBF data. Then we can point to that directory in the
docker compose file.

There is also an option, using the `tile_urls` environment variable to
download PBF data directly from the Geofabrik server. I used a Hawaii tile
for testing because it was a small(er) PBF file.

The important part here is that we're using local PBF data, and that we're
hosting Valhalla on the default port 8002.

{% highlight yaml %}
services:
  web:
    build: ./backend
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - MONGO_URI=mongodb://db:27017
  db:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
  valhalla:
    image: ghcr.io/gis-ops/docker-valhalla/valhalla:latest
    ports:
      - 8002:8002
    volumes:
      - ./custom_files/:/custom_files # if using local files
    environment:
      # auto-download PBFs from Geofabrik
      # - tile_urls=https://download.geofabrik.de/europe/andorra-latest.osm.pbf https://download.geofabrik.de/europe/albania-latest.osm.pbf
      - server_threads=2  # determines how many threads will be used to run the valhalla server
      - serve_tiles=True  # If True, starts the service. If false, stops after building the graph.
      - use_tiles_ignore_pbf=False  # load existing valhalla_tiles.tar directly
      - tileset_name=valhalla_tiles  # name of the resulting graph on disk
      - build_elevation=False  # build elevation with "True" or "Force": will download only the elevation for areas covered by the graph tiles
      - build_admins=False  # build admins db with "True" or "Force"
      - build_time_zones=False  # build timezone db with "True" or "Force"
      - build_transit=False  # build transit, needs existing GTFS directories mapped to /gtfs_feeds
      - build_tar=False  # build an indexed tar file from the tile_dir for faster graph loading times
      - force_rebuild=True  # forces a rebuild of the routing tiles with "True"
      - update_existing_config=True  # if there are new config entries in the default config, add them to the existing config  
      # - path_extension=graphs  # this path will be internally appended to /custom_files; no leading or trailing path separator!

volumes:
  mongodb_data:
{% endhighlight %}

## React Client

Since this is a simple proof of concept, the client it pulling route data
directly from the Valhalla server, and not using backend API. In a production
environment, route requests would go through a loadbalancer, and then to a
dedicated routing server cluster, with caching and analytics along the way.

The only gotcha was that Mapbox expects *(longitude, latitude)* pairs, while
Valhalla returns *(latitude, longitude)* pairs, and that the Valhalla had
scaled the coordinates to *10x* what I expected, i.e., a 20 degree latitude was
200 degrees. Besides that, adding a source GeoJSON and a layer was pretty easy.


{% highlight typescript %}
import React, { useRef, useEffect } from 'react';

import TextField from '@mui/material/TextField';
import Button from '@mui/material/Button';

import mapboxgl from 'mapbox-gl';
import polyline from '@mapbox/polyline';
import 'mapbox-gl/dist/mapbox-gl.css'

const Map: React.FC = () => {

  const mapContainer = useRef<HTMLDivElement>(null);
  let map = useRef<mapboxgl.Map | null>(null);

  const lng = -156.4;
  const lat = 20.7;
  const zoom = 10;
  const [mapLoaded, setMapLoaded] = React.useState(false);
  const [originLat, setOriginLat] = React.useState(20.71631);
  const [originLng, setOriginLng] = React.useState(-156.44612);
  const [destLat, setDestLat] = React.useState(20.78871);
  const [destLng, setDestLng] = React.useState(-156.46741);

  useEffect(() => {
    mapboxgl.accessToken = import.meta.env.VITE_MAPBOX_ACCESS_TOKEN; // Set your access token
    if (!mapContainer.current) {
      return;
    }
    map.current = new mapboxgl.Map({
      container: mapContainer.current,
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [lng, lat],
      zoom: zoom
    });
    map.current.on('load', () => { // Listen for the 'load' event
      setMapLoaded(true); // Set state to true when the map is loaded
    });
    // Add zoom controls
    map.current.addControl(new mapboxgl.NavigationControl(), "top-left");
  }, []);

  async function getDirections() {
    const serverUrl = "http://localhost:8002/route";
    const url = `${serverUrl}?json={"locations":[{"lat":${originLat},"lon":${originLng}},{"lat":${destLat},"lon":${destLng}}],"costing":"auto"}`;
    let resp;
    try {
      resp = await fetch(url);
    } catch (error) {
      console.error('Error fetching Valhalla route:', error);
      return;
    }
    let data;
    try {
      data = await resp.json();
    } catch (error) {
      console.error('Error parsing Valhalla route JSON:', error);
      return;
    }
    if (data.trip && data.trip.legs && data.trip.legs.length > 0 && data.trip.legs[0].shape) {
      const encodedPolyline = data.trip.legs[0].shape;

      // Decode the polyline
      const decodedPolyline = polyline.decode(encodedPolyline);

      // Convert to GeoJSON format (Mapbox requires [longitude, latitude])
      const geojson: GeoJSON.Feature<GeoJSON.LineString> = {
        type: 'Feature',
        geometry: {
          type: 'LineString',
          coordinates: decodedPolyline.map(coords => [0.1*coords[1], 0.1*coords[0]]) // Important: [lng, lat]
        },
        properties: {}
      };
      displayRouteOnMapbox(geojson);
    } else {
      console.error('Valhalla route not found or missing shape:', data);
    }
  }

  function displayRouteOnMapbox(geojson: GeoJSON.Feature<GeoJSON.LineString>) {
    console.log(`maploaded: ${mapLoaded}, geojson: ${JSON.stringify(geojson)}`);
    if (!mapLoaded) {
      console.error('Map not loaded yet');
      return;
    }
    map.current.addSource('route', {
      'type': 'geojson',
      'data': geojson
    });
    map.current.addLayer({
      'id': 'route',
      'type': 'line',
      'source': 'route',
      'layout': {
        'line-join': 'round',
        'line-cap': 'round'
      },
      'paint': {
        'line-color': '#ff4f84',
        'line-width': 3,
        'line-opacity': 0.5
      }
    });
  }

  return (
    <div>
      <div className="sidebarStyle">
          <div>Longitude: {lng} | Latitude: {lat} | Zoom: {zoom}</div>
      </div>
      <div>
        <TextField
          id="origin-lat"
          label="Origin Latitude"
          variant="outlined"
          value={originLat}
          onChange={(e) => setOriginLat(parseFloat(e.target.value))}
          margin="normal" // Adds some vertical spacing
        />
        <TextField
          id="origin-lng"
          label="Origin Longitude"
          variant="outlined"
          value={originLng}
          onChange={(e) => setOriginLng(parseFloat(e.target.value))}
          margin="normal"
        />
        <TextField
          id="dest-lat"
          label="Destination Latitude"
          variant="outlined"
          value={destLat}
          onChange={(e) => setDestLat(parseFloat(e.target.value))}
          margin="normal"
        />
        <TextField
          id="dest-lng"
          label="Destination Longitude"
          variant="outlined"
          value={destLng}
          onChange={(e) => setDestLng(parseFloat(e.target.value))}
          margin="normal"
        />
        {mapLoaded && 
          <div>
            <Button variant="contained" onClick={getDirections}>
              Get Directions
            </Button>
          </div>
        }
      </div>
      <div ref={mapContainer} className="map-container" style={{ height: '400px' }} />
    </div>
  );
};

export default Map;
{% endhighlight %}