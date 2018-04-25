---
layout: section
order: 10
title: Troubleshooting
---

## Problem with red squiggly lines and broken completion / types

As documented in more detail in our [Contributing Guide](/server/contributing#scala-compiler-and-refactoring), ENSIME relies on type information provided by Scala's Presentation Compiler and it is known to issue false positives. To help you diagnose problems, we wrote [PC Plod](https://github.com/ensime/pcplod). You can write a PC Plod unit test for the library that you are using to raise awareness with the author of that library that they not compatible with the presentation compiler (and error reporter).

We are also experimenting with workarounds to the specific problem of complex implicit resolutions. These will require you to get involved as a contributor:

1. [ensime/ensime-plugin-implicits](https://github.com/ensime/ensime-plugin-implicits)
1. prefer [semi-auto derivation](https://gitlab.com/fommil/scalaz-deriving) to [full generic derivation](http://fommil.com/scalax15/)
1. [SCP-010 Compiler Profiling](https://github.com/scalacenter/advisoryboard/blob/master/proposals/010-compiler-profiling.md)
1. [inductive implicits](https://github.com/scala/scala/pull/5649)

The presentation compiler supports regular blackbox macros but does not support advanced whitebox macros (e.g. shapeless `Generic` fundef materialisers) or annotation macros (like simulacrum's `@typeclass`). If you are a library author, consider using a compiler plugin based on [AnnotationPlugin.scala](https://gitlab.com/fommil/scalaz-deriving/blob/master/deriving-plugin/src/main/scala/scalaz/plugins/deriving/AnnotationPlugin.scala) instead of macro-paradise.

## Freezing Up

1. Check if you're running out of RAM, if so try the workarounds documented in [#1756](https://github.com/ensime/ensime-server/issues/1756).
2. If you enabled reverse lookups, this could be the cause. Note that earlier versions of ensime-sbt would generate project configurations with this enabled, by default.
3. See "Problem with red squiggly lines" above, as it has the same root cause.

## Anything else

1. Ensure that your build tool and editor plugins are up to date.
1. Fully compile your project, ENSIME uses your binaries for indexing.
1. Check [all ENSIME FAQ issues](https://github.com/search?utf8=%E2%9C%93&q=user%3Aensime+is%3Aissue+label%3AFAQ&type=Issues&ref=searchresults) and do a quick search.

If that solved your problem, great! If not, try asking in the chat room for your text editor's plugin.

Remember, everybody is here to help, but nobody is paid to maintain ENSIME (not even the sponsored developers). For ENSIME to be sustainable, we need you to engage in the bug fixing process rather heavily and (ideally) submit a pull request with a fix.
