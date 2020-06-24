+++
title = "Creating thin Docker containers for Rust binaries"
date = 2020-06-24T18:21:13+02:00
slug = "thin-docker-containers-for-rust-binaries"

[taxonomies]
tags = ["rust", "docker"]
+++

Sometimes you need to distribute some tools as Docker containers, for example when you are running some long-running processes in a Kubernetes cluster. Or whatever else, I don't judge. Since nowadays I write all CLI tooling in Rust because it's just pure joy, I decided I need to make myself a small `Dockerfile` template for building thin images that won't take much disk space and are fast to download and run with only the bare minimum.

To ensure we're gonna end up with a really small Docker image, we need to make it a [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/) one. My original plan seemed easy enough. Compile a static binary using the Rust Linux musl toolchain in the first stage. Then move only the binary to a [scratch image](https://hub.docker.com/_/scratch/) in the second stage. The scratch image is the most minimal Docker image you can use, with no extra layers, and nothing else in it really. 

There are official [Rust Docker images](https://hub.docker.com/_/rust), so that's what I chose as a build stage base. They include `rustup`, so I just installed the `musl` toolchain and `musl-tools`, and I thought I was ready to go. However, when I tried to build the image, the build step in the first phase panicked with the following error:

```
thread 'main' panicked at '

Could not find directory of OpenSSL installation, and this `-sys` crate cannot
proceed without this knowledge. If OpenSSL is installed and this crate had
trouble finding it,  you can set the `OPENSSL_DIR` environment variable for the
compilation process.

Make sure you also have the development packages of openssl installed.
For example, `libssl-dev` on Ubuntu or `openssl-devel` on Fedora.
```

Of course, I thought, I'm doing some networking, nowadays encrypted connections are a must. So let's just install `libssl-dev`, as the error suggests. Close, but no cigar. Musl and openssl-sys-extra didn't play well since the SSL package in Ubuntu is linked against `glibc`. Makes sense. What's next? I didn't really feel like compiling OpenSSL myself.

With a lit bit of searching, I found the great [clux/muslrust](https://github.com/clux/muslrust) Docker image. It's based on `Ubuntu Xenial` and conveniently has built a [lot of things with musl-gcc](https://github.com/clux/muslrust#c-libraries), like OpenSSL. A big thanks for this effort to [clux, the creator](https://github.com/clux)!

After replacing the `rust` image with `clux/muslrust` the crate compilation went fine and I thought that's that. But since I kinda rushed it, I totally forgot the small tiny detail of CA certificates. The Docker image was fine, however, the binary was basically not able to do any HTTPS requests since they would error out with certificate errors. Remember, the scratch image is containing basically just the binary.

So after a couple of extra steps to install the CA certificates Ubuntu package and then copy over the certificates from the first to the second stage, I was done. And here's the final result.

```Dockerfile
FROM clux/muslrust AS build
WORKDIR /usr/src

# Update CA Certificates
RUN apt update -y && apt install -y ca-certificates
RUN update-ca-certificates

# Build dependencies and rely on cache if Cargo.toml
# or Cargo.lock haven't changed
RUN USER=root cargo new <YOUR_CRATE>
WORKDIR /usr/src/<YOUR_CRATE>
COPY Cargo.toml Cargo.lock ./
RUN cargo build --target x86_64-unknown-linux-musl --release

# Copy the source and build the application.
COPY src ./src
RUN cargo install --target x86_64-unknown-linux-musl --path .

# Second stage
FROM scratch

# Copy the CA certificates
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
# Copy the statically-linked binary to the second stage
COPY --from=build /root/.cargo/bin/<YOUR_CRATE> .
USER 1000

CMD ["./<YOUR_CRATE>"]
```

This basic `Dockerfile` template is available as a [GitHub Gist](https://gist.github.com/zbrox/716d75c9c8d27016f2f528335617ceb4). I use it in a couple of places, one of which is a public crate called [rmq_monitor](https://crates.io/crates/rmq_monitor). The image size turns out to be a tad more than your binary, in the case of [the rmq_monitor image](https://hub.docker.com/layers/zbrox/rmq_monitor/0.2.5/images/sha256-32ef6ed0329066d3ad83949c0581782eec5aeed0ebdbcd80707ce20a7208fceb?context=explore) it's only __7Mb__ uncompressed.