---
layout: post
title:  "Use Rust and JS to Build a Simple Cross Platform Desktop App"
date:   2024-05-20 00:00:00 -0700
categories: tech
tags: rust
---


![Cross Platform Desktop App Example](/assets/images/Screenshot 2024-05-21 at 2.09.13 PM.png)


[Tauri](https://tauri.app/) is very similar to [Wails](https://wails.io/), except that it is based on Rust instead of Go. Both platforms let you choose from several frontend frameworks, and both platforms allow you to pass data back and forth between the JS and the compiled code. This guide will walk you through setting up Rust, Tauri, and Tailwind.

## Rust Quickstart

First you need to install rustup. Warning, this executes random code from the internet.

{% highlight console %}
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
{% endhighlight %}

Then you should update rustup.

{% highlight console %}
rustup update
{% endhighlight %}

Check to make sure cargo is installed.

{% highlight console %}
cargo --version
{% endhighlight %}

From here, you can perform a number of actions.

  - create a project with `cargo new <project>`
  - build your project with `cargo build`
  - run your project with `cargo run`
  - test your project with `cargo test`
  - build documentation for your project with `cargo doc`
  - publish a library to crates.io with `cargo publish`

Running, `cargo new hello` and `cargo run` will build and run a hello world program 🎉

## Tauri Quickstart

Create a Tauri project. This will run a CLI installer and setup a project for you in a new directory. Warning, this executes random code from the internet.

{% highlight console %}
sh <(curl https://create.tauri.app/sh)
{% endhighlight %}

I made the selections: tauri-app > Javascript > pnpm > Vue > Javascript. Then run,

{% highlight console %}
cd tauri-app
pnpm install
pnpm tauri dev
{% endhighlight %}

The line `pnpm tauri dev` will run the code. Any changes to the source code on the Rust or JS side will automatically update the UI.

## Configure Tailwind

This was a little tricky. From the root of the directory, run the following.

{% highlight console %}
pnpm install tailwindcss postcss-cli autoprefixer
npx tailwindcss init
{% endhighlight %}

The command `npx tailwindcss init` will create a `tailwind.config.js` file. Edit it to look like this:

{% highlight javascript %}
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./src/**/*.{vue,js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
{% endhighlight %}

Now create `postcss.config.js` at the root of the directory.

{% highlight javascript %}
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
{% endhighlight %}

Now put this `index.css` next to `main.js` in `./src/`.

{% highlight css %}
@tailwind base;
@tailwind components;
@tailwind utilities;
{% endhighlight %}

Update `main.js` to look like this.

{% highlight javascript %}
import { createApp } from "vue";
import App from "./App.vue";
import "./index.css";

createApp(App).mount("#app");
{% endhighlight %}

And simplify `App.vue` to look something like this.

{% highlight vuejs %}
<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import Greet from "./components/Greet.vue";
import "./index.css"; // -- new --
</script>

<template>
  <div class="m-4">
    <h1 class="text-3xl">Welcome to Tauri!</h1>
    <Greet />
  </div>
</template>

<style scoped>
</style>
{% endhighlight %}

Now you should be able to start using Tailwind.

## Use Rust to Connect to DuckDB

First, add the duckdb crate by cd-ing into `./src-tauri/` and running,

{% highlight console %}
cargo add duckdb
{% endhighlight %}

I ran into the following issue building the binary: `ld: library 'duckdb' not found`, so I updated the duckdb line in my `cargo.toml` file in `./src-tauri/` to this.

{% highlight console %}
duckdb = { version = "0.10.2", features = [
    "bundled"
] }
{% endhighlight %}

Then, in `./src-tauri/src/main.rs` I created a struct to model my database table, with serialization.

{% highlight rust %}
// Prevents additional console window on Windows in release, DO NOT REMOVE!!
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

use duckdb;
use serde::{Serialize, Deserialize};

#[derive(Debug)]
#[derive(Serialize, Deserialize)]
struct LogAsciiFile {
    api: String,
    wellname: String,
    latitude: f32,
    longitude: f32,
}
{% endhighlight %}

Then I fleshed out the `fetch_data` function that will be called from VueJS.

{% highlight rust %}
// Learn more about Tauri commands at https://tauri.app/v1/guides/features/command
#[tauri::command]
fn fetch_data(path: &str) -> Vec<LogAsciiFile> {
    
    let result = duckdb::Connection::open(path);
    let conn = match result {
        Ok(conn) => conn,
        Err(err) => {
            println!("Error opening connection: {:?}", err);
            return Vec::new();
        }
    };

    // query table by rows
    let result = conn.prepare("SELECT * FROM log_ascii_file_table;");
    let mut stmt = match result {
        Ok(stmt) => stmt,
        Err(err) => {
            println!("Error querying: {:?}", err);
            return Vec::new();
        }
    };

    let log_ascii_file_iter = match stmt.query_map([], |row| {
        Ok(LogAsciiFile {
            api: row.get(1)?,
            wellname: row.get(2)?,
            latitude: row.get(4)?,
            longitude: row.get(5)?,
        })
    }) {
        Ok(iter) => iter,
        Err(err) => {
            println!("Error querying map: {:?}", err);
            return Vec::new();
        }
    };

    let mut wellnames: Vec<LogAsciiFile> = Vec::new();
    for log_ascii_file in log_ascii_file_iter {
        let las = log_ascii_file.unwrap();
        wellnames.push(las);
    }
    return wellnames;
}
{% endhighlight %}

Finally, the `main` function exposes Rust functions to VueJS.

{% highlight rust %}
fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![fetch_data])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
{% endhighlight %}

## Use VueJS and Tailwind to Present Data

This imports an `invoke` function that can be used to call Rust functions from VueJS. Then we wrap the Rust `fetch_data` function with a JS `fetch_data` function, and call it on a form submit in the Vue template.

{% highlight vuejs %}
<script setup>
import { ref } from "vue";
import { invoke } from "@tauri-apps/api/tauri";

import TableRow from "./TableRow.vue";

const result = ref("");
const path = ref("");
let showTable = false;

function fetch_data() {
  invoke("fetch_data", { path: path.value }).then((resp) => {
    result.value = resp;
    if (result.value.length > 0) {
      showTable = true;
    }
  });
}
</script>

<template>
  <form class="row" @submit.prevent="fetch_data">
    <div class="my-2 flex">
      <button class="border rounded p-1 px-2 mr-2 bg-blue-500 text-white" type="submit">Run</button>
      <input class="w-full border rounded p-1" v-model="path" placeholder="Enter a path..." />
    </div>
  </form>
  <div v-show="showTable" class="mt-4">
    <table>
      <thead>
        <tr>
          <th class="text-left">API</th>
          <th class="text-left">Well Name</th>
          <th class="text-left">Latitude</th>
          <th class="text-left">Longitude</th>
        </tr>
      </thead>
      <tbody v-for="row in result" :key="row.id">
        <table-row :api="row.api" :wellname="row.wellname" :latitude="row.latitude" :longitude="row.longitude" />
      </tbody>
    </table>
  </div>
</template>
{% endhighlight %}

{% highlight javascript %}
{% endhighlight %}
