---
layout: post
title:  "Getting Started with Tauri"
date:   2024-0-0 00:00:00 -0700
tags: rust
---

Tauri is very similar to Wails, except that it is based on Rust instead of Go. Both platforms let you choose from several frontend frameworks, and both platforms allow you to pass data back and forth between the JS and the compiled code.

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

The line `pnpm tauri dev` will run the code. Any changes to the source code on the Rust or JS side will automatically update the UI. I'll follow up with more a less trivial example in a few days.
