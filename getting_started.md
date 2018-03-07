---
layout: page
order: 1
title: Getting Started
site_nav_entry: true # this is an entry in the main site nav
---

Fantastic, and welcome!

`ensime-server` is a process that indexes your dependencies and understands your source code using the same [scalac](http://www.scala-lang.org/files/archive/nightly/docs/compiler/index.html#scala.tools.nsc.interactive.package) and [javac](https://docs.oracle.com/javase/8/docs/jdk/api/javac/tree/) that you use to build your project.

ENSIME is not an IDE, it is just one tool in a toolbox. You must also have:

1. a build tool to compile your project
1. a `.ensime` file describing your project layout. To generate this file, see [our list of build tool plugins](/build_tools/).
1. a text editor. See [our list of editor plugins](/editors/).

The build tool plugin downloads the `ensime-server` and your text editor launches it. There's no separate "Download" page.
Read the [server documentation](/server) to further customise your experience.

## Getting Help

If things don't work as you expected, please read the [Getting Help](/getting_help) guide before asking for help.

## Contributing

You are invited to read the [`ensime-server` Contributing Guide](/server/contributing) and improve ENSIME. You'll get lots of help from existing contributors. Please take a moment to read [The Open Source Entitlement Complex](https://medium.com/@fommil/the-open-source-entitlement-complex-bcb718e2326d#.tvgf7fn0v) to understand why *we empower, but our contributors do not provide customer support*.

## Sponsoring

Whether or not you can help with direct contributions, [please consider sponsoring a developer](/sponsor).
