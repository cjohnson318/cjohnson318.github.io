---
layout: post
title:  "Getting Started with Rust and Tauri"
date:   2024-05-20 00:00:00 -0700
tags: rust
---

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

Now put this `index.css` next to `main.js` in `/src/`.

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

{% highlight javascript %}
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

