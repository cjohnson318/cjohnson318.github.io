---
layout: post
title: "Geospatial Caching"
date:  2025-01-02 00:00:00 -0700
categories: tech
tags: python
---

I was wondering how applications cache GIS data. I learned that there's a
Mapbox Tile Service (MTS) that handles the generation, storage, and delivery of
map tiles to an client application. You create a tile set using your own data,
like a GeoJSON, shapefile, etc., and then your application can either request
those tiles from MTS directly, or cached/proxied through your server. In the 
latter case, the client application would make a request to your server, and
you would check your cache for a tile. If that tile was not found, you'd query
the MTS system for the tile, and then deliver it to the client application.

Here is an example FastAPI backend that serves as a reverse proxy to MapBox.

{% highlight python %}
import os
import json
import redis
import requests
from fastapi import FastAPI, HTTPException, Query
from fastapi.responses import Response

MAPBOX_ACCESS_TOKEN = os.environ.get("MAPBOX_ACCESS_TOKEN")
if not MAPBOX_ACCESS_TOKEN:
    raise ValueError("MAPBOX_ACCESS_TOKEN environment variable not set")

REDIS_HOST = os.environ.get("REDIS_HOST", "localhost")
REDIS_PORT = int(os.environ.get("REDIS_PORT", 6379))
REDIS_DB = int(os.environ.get("REDIS_DB", 0))

redis_client = redis.Redis(host=REDIS_HOST, port=REDIS_PORT, db=REDIS_DB)

app = FastAPI()

@app.get("/tiles/{z}/{x}/{y}.{format}")
async def get_tile(z: int, x: int, y: int, format: str, tileset_id: str = Query("mapbox.mapbox-streets-v8")):
    tile_key = f"{tileset_id}/{z}/{x}/{y}.{format}"
    cached_tile = redis_client.get(tile_key)

    if cached_tile:
        return Response(content=cached_tile, media_type=f"image/{format}" if format in ["png", "jpg", "jpeg"] else "application/x-protobuf")
    else:
        try:
            url = f"https://api.mapbox.com/v4/{tileset_id}/{z}/{x}/{y}.{format}?access_token={MAPBOX_ACCESS_TOKEN}"
            response = requests.get(url, stream=True)
            response.raise_for_status()  # Raise HTTPError for bad responses (4xx or 5xx)

            tile_content = response.content
            redis_client.setex(tile_key, 3600, tile_content) # Cache for 1 hour

            return Response(content=tile_content, media_type=response.headers['Content-Type'])
        except requests.exceptions.RequestException as e:
            raise HTTPException(status_code=500, detail=str(e))
{% endhighlight %}

And this is an example of a JS frontend component that serves a MapBoxGL
component.

{% highlight react %}
import React, { useRef, useEffect, useState } from 'react';
import mapboxgl from 'mapbox-gl';

mapboxgl.accessToken = 'YOUR_MAPBOX_ACCESS_TOKEN'; // For base map only

const Map = () => {
    const mapContainer = useRef(null);
    const [map, setMap] = useState(null);

    useEffect(() => {
        const initializeMap = () => {
            const map = new mapboxgl.Map({
                container: mapContainer.current,
                style: 'mapbox://styles/mapbox/streets-v12', // Or another base map style
                center: [-77.032, 38.913],
                zoom: 9
            });

            map.on('load', () => {
                map.addSource('my-tiles', {
                    'type': 'raster', // or vector for .pbf
                    'tiles': [
                        'http://localhost:8000/tiles/{z}/{x}/{y}.png?tileset_id=mapbox.satellite', // Use your backend URL
                    ],
                    'tileSize': 256
                });
                map.addLayer({
                    'id': 'my-tiles-layer',
                    'type': 'raster',
                    'source': 'my-tiles',
                    'minzoom': 0,
                    'maxzoom': 22
                });

                setMap(map);
            });
        };
        if (!map) initializeMap();
    }, [map]);

    return <div ref={mapContainer} />;
};

export default Map;
{% endhighlight %}
