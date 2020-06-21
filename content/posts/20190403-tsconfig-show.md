+++
title = "Displaying the computed and in use tsconfig"
date = 2019-04-03T09:29:00+01:00
slug = "tsconfig-show"

[taxonomies]
tags = ["programming", "typescript", "short"]
+++

Usually, the configuration for the TypeScript compiler is pretty light. In some weird situations when things are not going as expected it is nice to be able to view what is the full `tsconfig.json` with all the default and computed properties in it. Since [TypeScript 3.2](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-2.html#the-new---showconfig-flag) you can
do that by passing the `--showConfig` flag.