---
layout: section
order: 5
title: Contributing
---

- TOC
{:toc}

The general architecture of ENSIME looks like this:

![architecture](http://ensime.org/talks/scalasphere16/images/architecture.png)

which is explained in a recent [talk about ENSIME](/talks/scalasphere16/) by @rorygraves and @fommil and an [even more recent one](/talks/scalasphere17/) by @fommil (both are still relevant). All our code is hosted at [github.com/ensime](https://github.com/ensime/).

## Where to Start

The first thing to do is to decide what you want to contribute!

If you are looking for a small task to pick up, have a look at our [latest milestone on github](https://github.com/ensime/ensime-server/milestone/9).

Anything with the **Low Hanging Fruit** or **Needs Reproduction** labels are suitable for a new contributor. Please contact us on the github issue directly if you want to contribute something: this avoids duplication of efforts, and also lets us discuss the best way to do it.

If you still don't know, join the conversation at [gitter.im/ensime/ensime-server](https://gitter.im/ensime/ensime-server).

If you can't come to a hack day but you'd really like to have some live help from somebody, please ask. We can use the libre software [meet.jit.si](https://meet.jit.si/) to screen share and discuss tickets anytime that is convenient.

If you'd like an ENSIME logo sticker for your laptop, let us know in your PR and privately email your postal address (or feel literally [free](http://www.gnu.org/philosophy/free-sw.en.html) to print your own, the logo is [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) and is not trademarked).


## Editors

If you want to work on editor support for ENSIME there may be additional documentation at these locations:

- [Emacs](/editors/emacs/contributing)
- [Atom](/editors/atom/contributing)
- [Visual Studio Code](/editors/vscode/contributing)
- [Vim](/editors/vim/contributing)
- [Sublime](/editors/sublime/contributing)

If you want to add support for an editor not listed here, please get in touch at [gitter.im/ensime/ensime-server](https://gitter.im/ensime/ensime-server) and read the server documentation below to get an idea of what the API looks like.


## Build Tools

If you want to work on build tool support, there may be documentation at these locations:

- [sbt](/build_tools/sbt#contributing)
- [Maven](/build_tools/maven#contributing)
- [Gradle](/build_tools/gradle#contributing)

If you want to add support for a new build tool, the [manual `.ensime`](/build_tools/manual) instructions are a good place to start.


## Scala Compiler and Refactoring

Much of the Scala support of ENSIME is provided by [the scala compiler itself](https://github.com/scala/scala) and the [refactoring library](https://github.com/scala-ide/scala-refactoring). Scala IDE also uses these components and their documentation on accessing the [presentation compiler](http://scala-ide.org/docs/dev/architecture/presentation-compiler.html#scalapresentationcompiler) is recommended reading. We also recommend watching [a recent talk about scala-refactoring](https://twitter.com/mlangc/status/697322490482315264) by @mlangc.

If you'd like to improve these components, you may wish to ask questions at [gitter.im/scala/contributors](https://gitter.im/scala/contributors) and [gitter.im/scala-ide/scala-ide](https://gitter.im/scala-ide/scala-ide) respectively.

Many problems (e.g. the infamous ["red squiggles on valid code"](https://github.com/ensime/ensime-server/issues/673)) may be incredibly difficult to solve in the compiler and we created [ensime/pcplod](https://github.com/ensime/pcplod) to help you create minimal examples of what is broken and to allow you to work with macro/plugin authors to fix the issues. If the problem appears to be a presentation compiler bug, you will need to fix the bug upstream.

## Server

The remainder of this document focuses on contributing to the server component, which is hosted at [github.com/ensime/ensime-server](https://github.com/ensime/ensime-server). If you pick up a ticket, please comment on it to let us know so that we can help (and also to avoid overlapping with somebody else). Feel free to ask questions on the github issue tracker or [gitter.im/ensime/ensime-server](https://gitter.im/ensime/ensime-server).

The server API is documented in [org/ensime/api](https://github.com/ensime/ensime-server/tree/2.0/api/src/main/scala/org/ensime/api)
with example JSON payloads in [org/ensime/jerky](https://github.com/ensime/ensime-server/blob/2.0/protocol-jerky/src/test/scala/org/ensime/jerky/JerkyFormatsSpec.scala). The preferred protocol for most editors is JSON over WebSockets, which we call our JERKY protocol. A legacy S-Expression over TCP protocol is used by Emacs (deriving from the SWANK protocol of [SLIME](https://github.com/slime/slime)), but ENSIME 2.0 will hopefully see us move to [S-Expressions over WebSockets](https://github.com/ensime/ensime-server/issues/1189), which we will call our SWANKY protocol.

### Compiling and Tests

Setting up `sbt` and `jenv` is documented in [Learning Scala](/learning_scala).

- without the `sbt` 0.13.13+ official start script (**not** the short bash script), project compilation will fail.
- without `jenv` the tests will fail to pass: we require more knowledge about JDK than typical projects.
- you must install the JDK sources (`src.zip`) with your OS package manager.

`ensime-server` has both unit tests and integration tests, both can be run with `sbt tests`.

Before running the integration tests you must run `sbt prep` to build an assembly jar and generate an index cache. This must be repeated if you change anything that will impact the indexing.

The `.drone.yml` file documents the exact commands that we use during our CI and should serve as a good reference. Our CI resources are very limited so please do an `sbt ci` before pushing.

### Implementing a Feature or Bugfix

It all starts with a test. Find a test that is already testing something similar to what you want to achieve, and adapt it.

We have several styles of tests: unit tests, in-memory source tests (e.g. [`RichPresentationCompilerSpec`](https://github.com/ensime/ensime-server/blob/2.0/core/src/it/scala/org/ensime/core/RichPresentationCompilerSpec.scala)), on-disk source tests (e.g. [`BasicWorkflow`](https://github.com/ensime/ensime-server/blob/2.0/core/src/it/scala/org/ensime/intg/BasicWorkflow.scala)) and the Emacs client tests. It may be instructive for you to read and understand some of these tests to get a feel for the right level to write your test.

### Guidelines

We do not have any official style guide. Code formatting is enforced by scalariform. When in doubt, prefer functional idioms encouraged by [Typelevel](http://typelevel.org) such as referential transparency, typeclass derivation and immutable data structures. We made a huge mistake depending on akka and we would like to [fix that by replacing it](https://github.com/ensime/ensime-server/issues/1351).

Note that many functional styles of programming [introduce huge memory overheads](https://skillsmatter.com/skillscasts/6939-optimising-scala-for-fun-and-profit) and we have several critical paths where we cannot afford to trade style over performance. We encourage *locally scoped* mutability if it improves performance and / or readability. We integrate with several heavily mutable external systems, such as the ASM classpath visitor, file systems, databases and the compiler APIs, so we try to be pragmatic.

Because we are at the forefront of the development cycle for new versions of Scala, we prefer to restrict our usage of external scala libraries. Therefore, please prefer Java dependencies where possible.

### Manual QA Testing

To build your own server jar:

```
sbt ++2.10.6 ensime-server/assembly # replace with your version of scala
```

**check that again**: Make sure the Scala major version matches the Scala major version of the project in which you wish to use the ensime-server snapshot build.

Next, use your build tool to provide this information in the `.ensime` file of your project. e.g. if you use `sbt-ensime`, put this in your `~/.sbt/0.13/global.sbt` (all projects) or `ensime.sbt` (just one project):

```scala
import org.ensime.EnsimeKeys._

ensimeServerJars in ThisBuild := Seq(file("/path/to/your/ensime-assembly.jar"))
```

make sure you're running the developer version of your editor plugin.
