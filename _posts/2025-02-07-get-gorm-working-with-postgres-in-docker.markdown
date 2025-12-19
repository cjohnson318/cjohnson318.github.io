---
layout: post
title: "Get GORM working with Postgres in Docker"
date: 2025-02-07 00:00:00 -0700
categories: tech
tags: go
---

This was challenging because even though my Go web service depended on the
PostgreSQL db service in docker compose, I still needed to add some extra
clauses to get these two technologies to synchronize. The trick was adding a
block to the db service to make sure postgres was up, and adding more metadata
to the `depends_on` block in the web service to coordinate with the db service
about when it was healthy and ready.

## Docker Compose Setup

A lot of this is normal, we're picking the latest postgres image, we're picking
simple values for the environment variables. The interesting thing is the
`healthcheck` block on the db service, and the `depends_on` block on the web
service. The web service waits until the db is in the `service_healthy` state.

The arguments in the `healthcheck` block are `-U` which should match 
`POSTGRES_USER`, and `-d` which should match `POSTGRES_DB`. I tried using
string interpolation here, but it didn't work right away, and rather than go
down a string interpolation rabbit hole, I just hardcoded those values for now.

{% highlight go %}
services:
  db:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d postgres"]
      interval: 10s
      retries: 5
      start_period: 30s
      timeout: 10s
    volumes:
      - pgdata:/var/lib/postgresql/data 
  web:
    build: .
    environment:
      GO_PORT: 8080
    ports:
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy
        restart: true
    links:
      - db
volumes:
  pgdata:
{% endhighlight %}

## Accessing Postgres from GORM

Setting up [GORM](https://gorm.io/) was the easier part of this. Here is my
database service code.

{% highlight go %}
package service

import (
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

type DbService struct {
	DB *gorm.DB
}

func NewDbService() DbService {
	dsn := "host=db user=postgres password=password dbname=postgres port=5432 sslmode=disable connect_timeout=5 TimeZone=America/Denver"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		panic("failed to connect database")
	}
	return DbService{DB: db}
}
{% endhighlight %}

And then the main entrypoint looks like this. In the `init()` function we set
up the database object, and then share that with the various services that
interact directly with the database, and then share the services with the
handlers that accept requests, and return [Templ](https://templ.guide/) 
components to the client.

{% highlight go %}
package main

import (
	"os"

	"notesApp/handler"
	"notesApp/model"
	"notesApp/service"

	"github.com/labstack/echo/v4"
)

var dbService service.DbService

func init() {
	dbService = service.NewDbService()
	err := dbService.DB.AutoMigrate(&model.Note{})
	if err != nil {
		panic("failed to migrate database")
	}
}

func main() {
	app := echo.New()
	app.Static("/static", "static")

	noteService := service.NoteService{DbService: dbService}
	noteHandler := handler.NoteHandler{NoteService: noteService}
	basicHandler := handler.BasicHandler{NoteService: noteService}

	app.GET("/", basicHandler.ShowHome)
	app.GET("/write", basicHandler.ShowWrite)
	app.GET("/notes/:id", noteHandler.ShowNote)
	app.POST("/notes", noteHandler.CreateNote)

	port := os.Getenv("GO_PORT")
	app.Logger.Fatal(app.Start(":" + port))
}
{% endhighlight %}
