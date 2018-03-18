---
layout: page
title: Opinions for the Scala LSP-WG and BTP-WG
---

*by Sam Halliday*

Although called the Language Server Protocol Working Group, we can hope that the
group's remit covers Scala Language Servers, rather than being specific to one
specific protocol.

Scala already has a language server, named ENSIME, which is community maintained
and is the development environment of choice for approximately 1,000 Scala and
Java developers using a variety of text editors: Emacs, Vim, Atom, VSCode and
Sublime.

ENSIME intentionally appeals to contributors, rather than consumers, as a
survival strategy. Our lack of commercial backing makes it impossible for us to
offer any kind of support. Some IDE users can be very demanding when things
don't work as they expect, so we do this to avoid contributor burn out. Any
thinking along the lines of "more users leads to more contributors" has been
proven false: we tried and it didn't work.

I am the biggest contributor to ENSIME, having taken over from the original
author, Aemon Cannon, [in
2014](https://github.com/ensime/ensime-server/graphs/contributors?type=a). I
thought the LSP-WG would appreciate my opinions on how to build a Language
Server, address some misconceptions, and point out areas where I feel efforts
would be best focused. The BTP-WG should also be aware of some of the components
that ENSIME currently supports but could be modularised into microservices.

## Scope of a Language Server

There is a misconception that a language server is a simple wrapper around an
interactive compiler, also called a presentation compiler. Such a language
server would only be able to provide a minimal set of features.

After many iterations, we have accepted that the ENSIME language server should
be responsible for providing the following user features

-   *Type at point*
-   *Contextual completion*
-   *Live errors & warnings*
-   *Semantic highlighting*
-   *Implicit conversions*
-   Import class at point
-   Classpath search
-   Jump to source
-   Open (hosted) documentation in browser
-   Find usages
-   Show implementations

The presentation compiler is used only for the first five features, in italics.

ENSIME also provides the following, but there is a strong argument that these
features should be provided by standalone microservices

-   Java support: type at point, completion, etc
-   Rename symbol and other refactorings
-   Organise imports and other code cleanups

ENSIME has held plans to implement

-   Natural type search, similar to Hoogle in Haskell
-   Passive suggestions / hints
-   `.sbt` files
-   `.dot` file generation of hierarchies
-   dead code analysis
-   visualising how two classes are related
-   ENSIME features from the REPL

Specifically, ENSIME has dropped support for the following features, which are
best managed by the build tool or some other component

-   interactive debugging
-   compiling
-   running the tests
-   running the application
-   launching the REPL
-   formatting the code

## Components of a Language Server

The ENSIME architecture circa 2016 can be visualised as

![img](http://ensime.org/talks/scalasphere16/images/architecture.png)

as described in [a talk by myself and Rory Graves](http://ensime.github.io/talks/scalasphere16/).

The most important components are:

-   build definition file
-   protocol
-   file watchers
-   search database
-   source resolving
-   documentation hosting
-   analyser

Note that the presentation compiler is a subset of the analyser.

Let's go through each component to describe why it exists, how we implemented
it, and any lessons learnt.

### Build Definition File

The text editor requires access to information such as the source directories
that are managed, scala / server jars, and the jvm arguments to launch the
language server for that project, which the user may customise. This information
is also useful for other navigation tasks such as finding test directories
associated to the production code, implemented client side and not requiring a
language server to be present.

ENSIME is cross built for major versions of scala for maximum compatibility,
launching the same Java process as used by the build tool. An alternative
solution would be to have special cased "language version" logic in the server.
We felt such approaches were overly complicated and cross building is
recommended.

Once started, the language server needs to know:

-   a directory that can be used for caching
-   location of source directories
-   location of target directories
-   location of third party jars, sources and docs
-   scalac and javac flags, broken down by subproject and configuration
-   the dependency relationships between subprojects

"magic directories" such as the base directory of an sbt project are problematic
because they must be special-cased in all components. Therefore we choose not to
support them. I recommend that sbt removes this overly complex feature, offering
a migration task to detect base `.scala` files and offer to create the
`src/main/scala` and move files.

The build definition file is produced by a plugin to the build tool. ENSIME has
plugins for

-   sbt
-   maven
-   gradle

with the primary goal of the plugin to produce the build definition file. This
allows us to lean on the build to download source and doc jars.

The sbt plugin has custom tasks to allow the user to script tasks such as
launching the language server to produce an index of the project, or to snapshot
/ recover a previous snapshot, resulting in much faster interactive startup
times (more about this later).

Additional features provided by the sbt plugin include extra commands that are
generally useful for developers using text editors and I recommend that they be
adopted by sbt core:

- `ensimeRunMain` an alternative to `runMain` allowing environment variables and
  jvm arguments to be used, e.g. `a/ensimeRunMain FOO=BAR -Xmx2g foo.Bar baz`.
- `c/ensimeLaunch MyApp` a launch manager that lets you define canned
  `ensimeRunMain` applications (analogous to IntelliJ's "Run Configurations")
- `b/ensimeCompileOnly` Compile a single fully-qualified `.scala` file using
  `b`'s classpath. Very useful to avoid triggering a full compile or to see
  the AST during a phase of the compile. e.g. `a/ensimeCompileOnly
  -Xprint:type /path/to/Foo.scala`
- `ensimeRunDebug` Like `ensimeRunMain` but adds debugging flags
- `ensimeTestOnlyDebug` extension to `testOnly` with debugging flags enabled

with tasks such as `ensimeScalariformOnly` being pushed upstream to the relevant
`sbt-scalariform` or `sbt-scalafmt` projects, necessary to enable "format on
save" features in text editors.

We would welcome a standardisation of the build definition file format, so long
as it contains all the information above and allows the user to override various
parts about their build when used in the language server. The [ENSIME
sbt](http://ensime.github.io/build_tools/sbt/#customise) documentation shows
many examples of where the user may wish to override a setting.

There has been some efforts between IntelliJ, ScalaIDE and ENSIME developers to
share sbt keys at https://github.com/JetBrains/sbt-ide-settings and to collect
information about the build structure at
https://github.com/JetBrains/sbt-structure, however time pressures have meant
that `sbt-ensime` has been unable to integrate these projects.

There is huge value in the `sbt-ensime` plugin being a single source file as it
allows installation in corporate environments with strict firewall rules and no
access to common distribution mechanisms such as maven central or bintray.
Indeed, in such scenarios, having a single jar distribution of the language
server itself is necessary to simplify the security audit process.

### Protocols

#### SWANKY and JERKY

ENSIME used to use a very adhoc protocol, derived from the SWANK protocol used
for Emacs communication with a common lisp language server.

However, nowadays a very standard JSON WebSockets protocol called JERKY, with an
S-Expression variant called SWANKY for a more efficient Emacs client
implementation.

WebSockets allows entirely asynchronous communication, including push
notification from the server, and is supported by middleware in all modern
languages. We use netty to provide NIO WebSockets support.

The protocol messages are derived from a simple ADT in the
https://github.com/ensime/ensime-server/tree/3.0/api/src/main/scala/org/ensime/api
package. The amount of code dedicated to the protocol is therefore insignificant
as we get it for free through typeclass derivation. We created an internal fork
of spray-json, adding typeclass derivation, to reduce our dependency list, which
is important to allow us to move fast to support new versions of scala.

Many of the parameters on the protocol messages have been added to provide user
experience enhancements on the client side. If you do not understand why some
information has been provided, it is recommend to search for that field in the
clients.

Note that all file paths must be canonicalised before being sent and when
received, including in the config file. We achieve this with a simple typeclass.
Failure to do so will result in bugs that can only be reproduced on specific
user's machines.

#### LSP

The LSP is truly terrible. The value of the LSP lies in the fact that many
editors will put common infrastructure in place to support common tasks such as
completion. The amount of developer time that this saves is of the order of
hours or days. Unfortunately there are many flavours of LSP and a significant
amount of client side work is still required for any given language or server.
Excuse me for not being the least bit excited about it.

ENSIME 3.0.0-SNAPSHOT implements the LSP on the server side, funded by the
sponsorship programme in response to huge public interest. I recommend that the
LSP-WG take ownership of the ENSIME LSP implementation in a separate repo with a
dependency on ensime stable releases. This is the same setup that we have in
https://github.com/dragos/dragos-vscode-scala

The ENSIME sponsorship pot also funded the development of scalajs LSP clients
for Atom and VSCode, in https://github.com/ensime/ensime-lsp-clients We felt
that ScalaJS would draw more contributors, as a common complaint is that users
do not want to learn coffeescript or typescript. I recommend that the LSP-WG
consolidates all LSP client code into a single repository to avoid wheel
reinvention and development in silos.

### Search Database

The search feature, find usages, show hierarchy and find source features are all
implemented by the Indexer.

I recently gave a talk about the ENSIME 2.0 implementation of the Indexer, which
was funded by GSoC and the sponsorship fund.
http://ensime.github.io/talks/scalasphere17/ This talk is recommended viewing
for anybody wanting to write a language server as it addresses one of the most
complex problems on the JVM: the naming mismatches between the various
components. Problems that we have solved in ensime.

The ENSIME 2.0 indexer is based on OrientDB and it has introduced performance
regressions. The single biggest problem facing ENSIME is the indexer, which
needs a rewrite https://github.com/ensime/ensime-server/issues/1902

The indexer parses raw `.class` files with ASM and extracts the scalap
information and attempts to reconcile it with the java names. I would prefer it
if the scalap format was better documented so that we did not have to rely on
the scalap tool and API, which is tied to the scala-library release cycle.
Notably, scalap has been dropped from dotty which raises concern.

If I were to rewrite the indexer, I would introduce an immutable persistence
format that lives in a shared cache that mirrors the root file system, rather
than centralised in a per-project database, to dramatically speed up startup
time and avoid redoing work between projects. Deletion of entries has been our
bottleneck when using both H2 and OrientDB, which is seen by users when they
change their middleware deps or perform a full clean/compile cycle. The UX for
this at the moment is quite terrible and just needs a little work.

If the LSP-WG has any resource to improve ENSIME, I would strongly recommend
rewriting the indexer, perhaps as a shared dependency to be used by other
projects.

There has been some movement towards this in
https://github.com/scalameta/scalameta/blob/master/semanticdb/metacp/src/main/scala/scala/meta/cli/Metacp.scala
but it is far from the persisted data required to implement the features that
ensime provides. We could also imagine having scalameta trees to provide
additional features for scala files, perhaps better position information for
position information. And in the ideal situation, we could incorporate
semanticdb files if the library author has published them. I am unaware of a
Java equivalent of scalameta, but it would be very useful for enhanced Java
support.

### Source Resolving

One of the smallest features of ENSIME is the component that associates
classfiles to sources. For java, this is trivial, because the conventions of
where a source file goes are directly related to its FQN. However, scala allows
sources to be defined anywhere.

As recommended in
[SCP-011](https://github.com/scalacenter/advisoryboard/pull/21) it would be very
helpful if the scala compiler added information to the class about the relative
location of a source file to its source root. Currently only the filename is
used. Which also causes problems for the debugger.

We use a simple heuristic that allows sources to be present in any root
directory of the symbols they define.

We perform source resolving before persisting to the index, so this calculation
is only performed once per class file.

### Documentation Hosting

This is quite simple, it simply means that we expose all the doc jars on a HTTP
endpoint. However, more complicated is resolving the URL of the docs for a given
symbol. It would be really useful if scalac did more for us here as we have lots
of hacks to do it ourselves, and to workaround changes during minor releases.

### Analyser

The ENSIME analyser is the component that wraps the interactive compiler. It
adds many features, such as the ability to lookup symbols by java FQN, and back
again, produce scaladoc names for symbols, and interact with the
scala-refactoring library.

Now that scalafix has arrived, I recommend a rewrite of the scala-refactoring
library as scalafix rules.

A shared library that can be used by all language servers would be very useful.
We discussed the possibility of doing this with the ScalaIDE team but it was
never considered a priority by those who fund the ScalaIDE so it was never
implemented.

The ENSIME analyser uses the technique of loading binaries /of the current
subproject/ into the presentation compiler, allowing it to support huge projects
and only load the files the user has open. Any attempt to load more than the
user's files into memory result simply do not scale. The PC can be restarted
quickly when binaries change.

For the first Scala Center advisory board meeting, Bill Venners presented
ENSIME's suggestion to fix various bugs in the presentation compiler
https://github.com/ensime/ensime.github.io/blob/e922e7cb3c06ee5f245083591315e8f564bfc343/contributing/center.md

It was decided not to fix the presentation compiler in scala 2.x, but to instead
focus on Dotty. I therefore gave my advice in
https://github.com/lampepfl/dotty/issues/1523#issuecomment-255862771 which
should be required reading for anybody working on improvements to the
presentation compiler in dotty.

Nevertheless, I think it was wrong to put off improving the PC, so I recommend
that the LSP-WG invest in improving the presentation compiler in scalac. One of
the things that would really help is the ability to kill a hung PC instance. We
have workarounds to try and stop the PC from performing complex implicit type
inference tasks, see https://github.com/ensime/ensime-plugin-implicits, but a
simple timeout would be very excellent. Currently if the PC starts to eat up
100% CPU then there is no way to fix it except to bounce the ensime process.

Annotation macros and whitebox macros break everything, with various typer
problems and off-by-one semantic highlighting bugs being quite frustrating. The
first step to doing this would be to create a good corpus of bugs... user
provide bugs are very low quality. Users complain about unexpected "red
squiggles" but are hopeless at providing an example of the failure. I created
https://github.com/ensime/pcplod with Rory to let people create standalone
reproductions, and for macro / compiler plugin authors to test their code inside
the PC. Further effort into making it easier for users of the PC to write a
failing test would be beneficial, as well as improvements to pcplod such as
extracting semantic information at a location from the PC.

As an alternative to annotation macros, I have started work on an abstract class
to allow annotation macros to be rewritten as compiler plugins
https://gitlab.com/fommil/scalaz-deriving/blob/master/deriving-plugin/src/main/scala/scalaz/plugins/deriving/AnnotationPlugin.scala
this is much better behaved than annotation macros, but it does not support
annotations by their FQN. I recommend that a phase be added to the compiler that
names annotations, or to improve my =AnnotationPlugin.scala= to add support for
naming of expected annotations (ignoring shadowing by annotations of the same
name).

I believe that the semanticdb is the future for any interactive tools in scala,
dramatically simplifying the job of both the Analyser and Indexer. But it is not
going to work if the underlying PC (that produces interactive semanticdbs) is
broken. The scalameta API is also *significantly* easier to work with than the
presentation compiler. It is extremely difficult to reason about anything inside
the PC because everything is mutable, lives in a big cake mixin, with dependent
types, with adhoc thread execution rules. As a tool author, I want to be able to
interact with the scala AST as an ADT, in a functional way. I look forward to
the RSC as an alternative to scalac for this one feature alone.

ENSIME also provides an Analyser for Java, but we only support Java 8 and other
versions of Java are supported only if the Oracle SDK has not broken binary
compatibility between releases, which it often does. I recommend that any common
build structure format be supported by java language servers, if this group
intends to create a standalone scala server, and expect the Java community to
provide support for Java, note that any features that make use of the indexer
will need to be duplicated across the Java and Scala microservices.

### File Watching

Java 7 introduced an efficient file watching API. We watch the target (class
file) directories for changes so that we know to restart the Analyser when there
have been many changes and reindex binaries that appear. In practice, the
analyser only needs to be bounced if there have been changes introduced in files
that are not opened by the user (e.g. file was changed and closed, or there was
a git checkout). A threshold on number of binary changes is our heuristic for
this.

We already know about changes to the source files through protocol messages, so
have no need to watch those directories.

It would be useful if ENSIME could run in a mode where the build tool told us
(e.g. through an HTTP POST message) when new files are created so we do not need
to watch for changes.

Given that so many interactive tools require a file watcher, it would be useful
if a common library was written to avoid wheel reinvention. We tried to get SBT
to reuse our library (abandoned in https://github.com/ensime/java7-file-watcher
and now part of the ensime-server monorepo) but they have created their own
infrastructure that appears to require blocking polling of the filesystem, which
is very inefficient on large projects, encrypted drives, Windows, and battery
powered laptops. We already see reimplementation, with bloop using the
https://github.com/gmethvin/directory-watcher. Personally I think relying on JNA
or native code is techdebt and the Java 7 API has been sufficient for our users.

## CI and testing

Testing was the focus for me and Rory when we took over from Aemon. The biggest
code quality weakness of ENSIME is its lack of unit tests, which we can
attribute to the use of akka and =Future=, which do not lend themselves to unit
testing.

If I had time to rewrite ensime, I would use higher kinded types to abstract all
the interfaces over an =F[_]= such that unit tests could be entirely pure and
predictable, and fast.

However, we're in the situation where we need large integration tests on user
projects, which results in a much slower developer cycle and puts many
contributors off.

Unfortunately, nothing can be done to improve the testing around the PC... that
kind of code simply cannot be unit tested, only integration tested. If scalac
provided an ADT of its AST, instead of the large mutable dependently typed cake
mixin, we'd be in a much better testing story.

One very real consequence of complex testing is that free CI providers such as
Travis are unable to provide enough power for the ensime tests. I have invested
significant time optimising our tests so that they can be run on commodity
hardware, but we had to run the main build on a privately hosted drone server,
that is no longer available. With recent refactorings in the 3.0 branch removing
the debugger, we should now be able to just about run our tests on travis for
the main branch, but build times will easily be in excess of 30 minutes.

## Highest Priority Recommendations

In terms of how the LSP-WG could directly benefit ENSIME, I recommend:

- investing in improving the presentation compiler
- writing and maintaining shared library components such as:
  - indexer
  - file watcher
  - presentation compiler helpers
- provide high powered CI machines for use by the tooling ecosystem
- take over the ensime-lsp project and clients
