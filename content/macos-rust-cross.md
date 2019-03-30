+++
title = "Getting rust-embedded/cross running on macOS"
date = 2019-03-30T12:25:00+01:00
[taxonomies]
categories = ["programming"]
tags = ["rust"]
+++

One of the great promises and benefits of Rust is that you can easily write code that cross-compiles to various different targets. Not only that but the ecosystem is constantly evolving and trying to make this process
as painless as possible. [Rustup](http://rustup.rs) makes [managing different target toolchains](https://blog.rust-lang.org/2016/05/13/rustup.html) a breeze. However, if you don't want to pollute your environment with all the different dependencies of those target toolchains you can try [cross](https://github.com/rust-embedded/cross). 

<!-- more -->

Simply put `cross` uses Docker containers to build your project. It mimics cargo's CLI arguments as much as possible but instead of using the `rustup` toolchains for a given target it spins up a Docker container. It then builds your project in it and spins down the container. To put it even simpler - it's great!

Initially, I had missed one important disclaimer. I got super confused when it wasn't working on macOS. As of this writing, the current version of `cross` (*0.1.14*) is only working for Linux hosts. On macOS, it will just act as a proxy and let cargo do its usual build process with a different `--target` argument.

Over the last couple of years, Docker support for macOS has been getting more solid. The kind people of the [Rust Tools](https://github.com/rust-embedded/wg#the-tools-team) team have actually made `cross` work with macOS but just haven't gotten around to making a release yet.

The solution until the next release is simple. Instead of doing

```sh
cargo install cross
```

Just do

```sh
cargo install --git https://github.com/rust-embedded/cross
```