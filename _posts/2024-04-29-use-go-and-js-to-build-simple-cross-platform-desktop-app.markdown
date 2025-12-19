---
layout: post
title:  "Use Go and JS to Build a Simple Cross Platform Desktop App"
date:   2024-04-29 00:00:00 -0700
categories: tech
tags: go javascript
---

![Cross Platform Desktop App Example](/assets/images/Screenshot 2024-05-07 at 9.54.36 PM.png)

Wails is a cross-platform desktop solution that leverages Go for the business logic and cross-platform part, and several JS frameworks/libraries for the presentation layer. This is achieved by allowing functions written in Go to be called automagically from the JS presentation layer.

So far, everything I've put into this project "just works". I added Tailwind for styling my Vue components, I added routing to present users with different views, I added OS operations to check that a file exists in the file system, I added a library to talk to DuckDB, I made (local) network requests. Everything went smoothly.

## Project Setup

First, I set up wails according to its documentation, something along the lines of...

{% highlight console %}
go install github.com/wailsapp/wails/v2/cmd/wails@latest
wails doctor
wails init -n projectname -t vue|react|svelte|preact|lit|vanilla
{% endhighlight %}

This created a directory called `projectname` with a two subdirectories: `build` and `frontend`. The presentation layer, written in vue, react, etc., will sit in the `frontend` subdirectory.

## Accessing a Database

I wanted this desktop application to pull data from a local DuckDB database. I was able to write a quick internal Go file to interact with that database using this code. This does the bare minimum, define a struct to hold a record from the database, define an interface for the data store: Open, Close, and a read function, GenerateLASMetadata. The read function takes a channel, reads rows from the database, creates a LogAsciiFile struct per row, and pushes those structs into the channel it was provided.

{% highlight go %}
package logascii

import (
	"database/sql"

	_ "github.com/marcboeker/go-duckdb"
)

// LogAsciiFile is a struct that represents a row in the log_ascii_file_table
type LogAsciiFile struct {
	ID        int     `db:"id" json:"id"`
	API       string  `db:"api" json:"api"`
	WellName  string  `db:"wellname" json:"wellname"`
	FileName  string  `db:"filename" json:"filename"`
	Latitude  float32 `db:"latitude" json:"latitude"`
	Longitude float32 `db:"longitude" json:"longitude"`
}

// LogAsciiDataStoreInterface is an interface that defines the methods that a LogAsciiDataStore must implement
type LogAsciiDataStoreInterface interface {
	Open(path string) error
	Close() error
	GenerateLASMetadata(lasChan chan LogAsciiFile) error
}

// LogAsciiDataStore is a struct that contains the database name and a pointer to the database
type LogAsciiDataStore struct {
	DBname string
	DB *sql.DB
}

// Open opens a connection to the database
func (lads *LogAsciiDataStore) Open(path string) error {
	db, err := sql.Open("duckdb", path)
	if err != nil {
		return err
	}
	lads.DB = db
	return nil
}

// Close closes the connection to the database
func (lads *LogAsciiDataStore) Close() error {
	return lads.DB.Close()
}

// GenerateLASMetadata returns a channel of LogAsciiFile structs
func (store LogAsciiDataStore) GenerateLASMetadata(lasChan chan LogAsciiFile) error {
	defer close(lasChan)
	rows, err := store.DB.Query("SELECT * FROM log_ascii_file_table")
	if err != nil {
		return err
	}
	defer rows.Close()

	for rows.Next() {
		var las LogAsciiFile
		err := rows.Scan(&las.ID, &las.API, &las.WellName, &las.FileName, &las.Latitude, &las.Longitude)
		if err != nil {
			continue
		}
		lasChan <- las
	}
	return nil
}
{% endhighlight %}

## Providing Data to JS from Go

The top-level `app.go` serves as the bridge between Go and your preferred JS library. I've defined a `GetData` function in `app.go` that I will be able to access from JS. This function creates a connection to the database store, creates a channel of type `LogAsciiFile`, starts fetching data from the database into that channel, and then collects the data into a slice of type `LogAsciiFile`. Once that is complete, the slice is passed into JS as a JSON list of objects.

{% highlight go %}
package main

import (
	"context"
	"encoding/json"
	"os"

	logascii "projectname/internal" // This is a custom package that is imported from the local directory
)

// App struct
type App struct {
	ctx context.Context
}

// NewApp creates a new App application struct
func NewApp() *App {
	return &App{}
}

// startup is called when the app starts. The context is saved
// so we can call the runtime methods
func (a *App) startup(ctx context.Context) {
	a.ctx = ctx
}

func pathDoesNotExist(path string) bool {
	if _, err := os.Stat(path); os.IsNotExist(err) {
		return true
	} else {
		return false
	}
}

func (a *App) GetData(path string) []logascii.LogAsciiFile {
        // if the path does not exist, return an empty slice
	if pathDoesNotExist(path) { 
		return []logascii.LogAsciiFile{}
	}
	store := logascii.LogAsciiDataStore{}
	store.Open(path)
	defer store.Close()

	lasChan := make(chan logascii.LogAsciiFile)
	go store.GenerateLASMetadata(lasChan)

	var lasFiles []logascii.LogAsciiFile
	for las := range lasChan {
		lasFiles = append(lasFiles, las)
	}
	return lasFiles
}
{% endhighlight %}

## Presenting Data in MapBox

On the JS side, I pull the database records into a VueJS component, and then present them as markers in MapBox.

{% highlight vuejs %}
<template>
  <div class="bg-gray-100 rounded mt-4 p-4">
    <div ref="mapContainer" class="map-container"></div>
  </div>
</template>

<script>
import '../../node_modules/mapbox-gl/dist/mapbox-gl.css'
import { GetData } from '../../wailsjs/go/main/App'
import mapboxgl from 'mapbox-gl'
mapboxgl.accessToken = 'MAPBOX-TOKEN'

export default {
  name: 'MapCard',
  data() {
    return {
      map: {},
      center: { lng: -102.224518, lat: 31.213995 },
      style: 'mapbox://styles/mapbox/streets-v12',
      dbConnectionStore: null,
    }
  },
  methods: {
    initMap(options) {
      const map = new mapboxgl.Map(options)
      return map
    },
    addMarker(map, coordinates) {
      new mapboxgl.Marker().setLngLat(coordinates).addTo(map)
    },
  },
  async mounted() {
    this.map = this.initMap({
      container: this.$refs.mapContainer,
      style: this.style,
      center: this.center,
      zoom: 6,
    })
    const path = '/path/to/duck/db/minimal.duckdb'
    const result = await GetData(path)
    for (let item of result) {
      this.addMarker(this.map, [item.longitude, item.latitude])
    }
  },
}
</script>

<style scoped>
.map-container {
  flex: 1;
  display: flex;
  height: 100vh;
}
</style>
{% endhighlight %}

## Compression with UPX

At this point, our executeable is about 40M, but we can use UPX to compress this down to about 14M.

{% highlight console %}
brew install upx
wails build -clean -upx
{% endhighlight %}
