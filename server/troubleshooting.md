---
layout: section
order: 10
title: Troubleshooting
---

## Problem with red squiggly lines and broken completion / types

As documented in more detail in our [Contributing Guide](/server/contributing#scala-compiler-and-refactoring), ENSIME relies on type information provided by Scala's Presentation Compiler and it is known to issue false positives. To help you diagnose problems, we wrote [PC Plod](https://github.com/ensime/pcplod). You can write a PC Plod unit test for the library that you are using to raise awareness with the author of that library that they not compatible with the presentation compiler (and error reporter).

We are also experimenting with workarounds to the specific problem of complex implicit resolutions. These will require you to get involved as a contributor:

1. [ensime/ensime-plugin-implicits](https://github.com/ensime/ensime-plugin-implicits)
1. prefer [semi-auto derivation](https://github.com/fommil/stalagmite/issues/38) to [full generic derivation](http://fommil.com/scalax15/)
1. [SCP-010 Compiler Profiling](https://github.com/scalacenter/advisoryboard/blob/master/proposals/010-compiler-profiling.md)
1. [inductive implicits](https://github.com/scala/scala/pull/5649)

## Freezing Up

1. check if you're running out of RAM, if so try the workarounds documented in [#1756](https://github.com/ensime/ensime-server/issues/1756).
2. ensime 2.0 introduced major performance regressions as the price for new features, the indexer needs a rewrite [#1902](https://github.com/ensime/ensime-server/issues/1902). We need a contributor to dedicate some serious time to fix this.

If performance is more important to you than features such as reverse lookup and rename refactor across files, try disabling the reverse lookup index. e.g. for =sbt= builds add this to your =~/.sbt/1.0/global.sbt=

```scala
ensimeJavaFlags ++= {
  if (name.value == "ensime") Nil
  else Seq("-Densime.index.no.reverse.lookups=true")
}
```

## Anything else

1. fully compile your project, ENSIME uses your binaries for indexing.
1. check [all ENSIME FAQ issues](https://github.com/search?utf8=%E2%9C%93&q=user%3Aensime+is%3Aissue+label%3AFAQ&type=Issues&ref=searchresults) and do a quick search.

If that solved your problem, great! If not, try asking in the chat room for your text editor's plugin.

Remember, everybody is here to help, but nobody is paid to maintain ENSIME (not even the sponsored developers). For ENSIME to be sustainable, we need you to engage in the bug fixing process rather heavily and (ideally) submit a pull request with a fix.
