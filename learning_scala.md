---
layout: page
order: 15
title: Learning Scala
site_nav_entry: true # this is an entry in the main site nav
---

This guide assumes no prior knowledge of Scala and will get you set up to the point where you have Scala installed on your computer and can hack on a project.

## Learn

We recommend reading [Programming in Scala](http://www.artima.com/shop/programming_in_scala_3ed) to learn the language.

If you come from an object oriented background, you will hear people talk about *Functional Programming* (FP), monads and typeclasses. Learn about them by getting [Functional Programming in Scala for Mortals](https://leanpub.com/fp-scala-mortals) by `@fommil`.

Before installing any software, you can complete exercises online at [scala-exercises.org](https://www.scala-exercises.org/) and execute code in your browser at [scalafiddle.io](https://scalafiddle.io/).

## Install

1. **Install Java**: Scala runs on the [Java Virtual Machine (JVM)](https://en.wikipedia.org/wiki/Java_virtual_machine).  
Prefer free/libre [OpenJDK](https://en.wikipedia.org/wiki/OpenJDK) to proprietary versions of Java.  
Install OpenJDK using your system package manager:
  * `sudo pacman -S openjdk8-src` ([ArchLinux](https://wiki.archlinux.org/index.php/java))
  * `sudo apt-get install openjdk-8-source` ([Debian](https://wiki.debian.org/Java/))
  * [Zulu Java](http://www.azul.com/downloads/zulu/) for all other systems
2. **Install sbt**: sbt will install all the build tooling you need, *including Scala*.
  * `sudo pacman -S sbt` (ArchLinux)
  * [Lightbend instructions](http://www.scala-sbt.org/0.13/docs/Setup.html) for all other systems
3. (GNU / BSD, *Optional*): If you anticipate using multiple versions of Java, install [jenv](http://jenv.be).

## First Project

If a volunteer would like to pad out the following, that'd be great. Until then, try [Daniel Spiewak's "Getting Started in Scala" guide](https://gist.github.com/djspiewak/cb72c41ac335a3a9b28b3307be04aa43)

1. checkout a basic tutorial project, e.g. [99 problems](https://github.com/rabbitonweb/99-scala-problems) or [scalania](https://github.com/jaceklaskowski/scalania)
2. start sbt and cheatsheet of basic commands (for now, try [sbt common commands](http://www.scala-sbt.org/0.13/docs/Running.html#Common+commands))
3. hello world in the editor without ensime (e.g. [/editors/emacs/scala-mode](/editors/emacs/scala-mode)) starting from scratch with `sbt new eed3si9n/hello.g8`

Only when you have been able to write simple puzzles should you start ensime server, for now read [Getting Started](/getting_started).

Bear in mind that there are several components (for Emacs with sbt) that go beyond just Scala the language:

- Emacs itself for common text editing and project management
- sbt the build tool for downloading your deps and building / running your app and tests
- sbt-mode for interacting with sbt from within emacs
- sbt-ensime, our plugin to the build tool to generate a `.ensime` file and provide nice functionality in the build.
- ensime-emacs, our emacs plugin to talk to the server
- ensime-server, our server component that indexes your binaries and understands source code context
