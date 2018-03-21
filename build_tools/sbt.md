---
layout: section
order: 2
title: sbt
---

- TOC
{:toc}

[sbt-ensime]: https://github.com/ensime/ensime-sbt
[ensime/ensime-sbt#237]: https://github.com/ensime/ensime-sbt/issues/237

The integration of ENSIME with [sbt](http://github.com/sbt/sbt) is provided by [sbt-ensime][], whose primary function is generating the `.ensime` file by inspecting the sbt build setup. In addition [sbt-ensime][] provides various convenience commands for interacting with the [ENSIME server](http://github.com/ensime/ensime-server).

## Install

[sbt-ensime][] is an editor integration plugin and isn't a strict requirement of the build.

Therefore it is recommended to install it as a [global plugin](http://www.scala-sbt.org/0.13/docs/Using-Plugins.html) so that it's always available.

To do so, add it to `~/.sbt/1.0/plugins/plugins.sbt` (create if necessary) as such:

```scala
addSbtPlugin("org.ensime" % "sbt-ensime" % "2.4.0")
```

Then in order to create the `.ensime` file for you project, start `sbt` (in the terminal or your editor's `sbt` mode) and run the `ensimeConfig` command.

### Common Mistakes

1. If you accidentally use `~/.sbt/1.0/plugins.sbt` instead of `~/.sbt/1.0/plugins/plugins.sbt` you'll get an sbt resolution error - make sure you're defining the plugin in the right place.

1. Use `~/.sbt/0.13/plugins/plugins.sbt` if you are using the older sbt 0.13

### Unstable Server

The latest stable release of the server is downloaded by default. If you wish to follow the *unstable* (continuously released) server, add the following to `~/.sbt/0.13/global.sbt`

```scala
import org.ensime.EnsimeCoursierKeys._
import org.ensime.EnsimeKeys._

ensimeRepositoryUrls in ThisBuild += "https://oss.sonatype.org/content/repositories/snapshots/"
ensimeServerVersion in ThisBuild := "3.0.0-SNAPSHOT"
ensimeProjectServerVersion in ThisBuild := "3.0.0-SNAPSHOT"
```

## Learn to Use sbt

If you've come from an IDE you might not be aware of the power of `sbt`. Please take the time to read the [sbt Getting Started Guide](http://www.scala-sbt.org/0.13/docs/Getting-Started.html) before proceeding and appreciate that `sbt` is responsible for building your project, not ENSIME.

## Core Commands

The core `sbt-ensime` plugin allows you to generate `.ensime` files:

* `ensimeConfig` --- Generate a `.ensime` for the project (takes space-separated parameters to restrict to subprojects).
* `ensimeConfigProject` --- Generate a `project/.ensime` for the project definition.

Note that downloading and resolving the sources and javadocs can take some time on first use, so we recommend that you use [coursier](http://get-coursier.io).

## Extra Tasks and Commands

Also bundled are extra workflow tasks, which are used by ENSIME clients:

| `ensimeRunMain`              | Alternative to `runMain` allowing environment variables and jvm arguments to be used, e.g. `a/ensimeRunMain FOO=BAR -Xmx2g foo.Bar baz`. |
| `c/ensimeLaunch MyApp`       | A launch manager that lets you define canned `ensimeRunMain` applications (analogous to IntelliJ's "Run Configurations"---[see below](#launch-configurations)). |
| `b/ensimeCompileOnly`        | Compile a single fully-qualified `.scala` file using `b`'s classpath. Very useful to avoid triggering a full compile or to see the AST during a phase of the compile. e.g. `a/ensimeCompileOnly -Xprint:type /path/to/Foo.scala` |
| `b/ensimeScalariformOnly`    | Format a single fully-qualified `.scala` file using `b`'s Scalariform settings (compatible with, but does not require, `sbt-scalariform`). |
| `ensimeRunDebug`             | Like `ensimeRunMain` but adds debugging flags automatically. |
| `ensimeTestOnlyDebug` | `...debug` extension to `testOnly`. Has the same logic as `ensimeRunDebug` in regard of `ensimeRunMain`. See debugging example [below](#debugging-example).|
{: .sbt-tasks}

The following are tasks can be used to create an optimal workflow (the server must not be running when using these tasks):

| `ensimeServerIndex` | Starts the server to create / update the index and then exits. |
| `ensimeClearCache` | Deletes the current server cache. |
| `ensimeSnapshot` | Creates a snapshot of the `ensime-server`'s index that can be restored at a later date. |
| `ensimeRestore` | Recover a previous snapshot of the server's index. |

For example, the following is a good "start the day" workflow hack while you go and get tea or coffee:

```
alias ensime="ctags -Re . & sbt clean ensimeConfig test:compile ensimeServerIndex"
```

sprinkling in an `ensimeRestore` if you regularly run `ensimeSnapshot`.

### Launch Configurations

To define presets for the `ensimeLaunch` command, use the `LaunchConfig` and `JavaArgs` case classes provided by sbt-ensime, assigning a sequence to the `ensimeLaunchConfigurations` setting for a project:

```scala
ensimeLaunchConfigurations := Seq(
  LaunchConfig("server",
    JavaArgs(
      mainClass = "mypackage.Server",
      classArgs = Seq("start", "--foreground"),
      envArgs   = Map("MYSERVER_PORT" -> "8080"),
      jvmArgs   = Seq("-Xmx2g")
    )
  )
)
```

Invoking `ensimeLaunch server` will then execute the given class with the specified arguments and environment.

An untracked `ensime.sbt` file is [advised](#customise) for this, assuming you have installed sbt-ensime as a global plugin as recommended.

### Debugging Example

Ensime plugin has two helpful comands for debugging: `ensimeRunDebug` for applications and `ensimeTestOnlyDebug` for tests.

```
> ensimeRunDebug foo.Bar
> ensimeTestOnlyDebug foo.BarTest
```

## Customise

For project-specific tailorings, you do not need to commit anything to your project. Simply create an `ensime.sbt` in your project's base directory (beside `build.sbt`). Don't forget to add it to your `~/.gitignore`.

Think long and hard before adding any of these settings to your global `~/.sbt/0.13/global.sbt`. For example, if you were to set a global `ensimeScalaVersion`, it will break if you are working on sbt plugins, old projects or upgrade to the latest scala. When putting settings in your global configuration, you must `import org.ensime.EnsimeKeys._`

The "Find Usages" server feature is disabled by default due to the performance cost. If you would like to enable it, use

```scala
ensimeServerFindUsages in ThisBuild := true
```


Here is an example that sets a specific memory size for the ensime server:

```scala
ensimeJavaFlags in ThisBuild := Seq("-Xss2m", "-Xmx2g", "-XX:MaxMetaspaceSize=512m")
```

another, to use ENSIME on a Java 6 or 7 project (the server needs JDK 8). This is reasonably safe in your `~/.sbt/0.13/global.sbt`:

```scala
ensimeJavaHome in ThisBuild := file("/usr/lib/jvm/java-8-openjdk")
```

another, to add a custom compiler plugin to your entire build (this is just an example, it's not available yet. You can write your own simpler compiler plugins to replace larger, slower, and buggy corporate plugins should the need arise)

```scala
addEnsimeScalaPlugin("org.ensime" %% "ensime-plugin-implicits" % "1.0.0")
```

another, to use the assembly jar which you build locally:

```scala
ensimeServerJars in ThisBuild := List(file("../ensime_2.11-2.0.0-SNAPSHOT-assembly.jar"))
ensimeServerProjectJars in ThisBuild := List(file("../ensime_2.10-2.0.0-SNAPSHOT-assembly.jar"))
```

another, for Android projects:

```scala
ensimeJavacOptions ++= (javacOptions in (core, Compile)).value,
ensimeScalacOptions ++= (scalacOptions in (core, Compile)).value
```

another, to revert to a previous stable or milestone of the server:

```scala
ensimeServerVersion in ThisBuild := "2.0.0-M2" // or "1.0.1"
ensimeProjectServerVersion in ThisBuild := "2.0.0-M2" // or "1.0.1"
```

to not download javadocs (who writes them anyway?)

```scala
transitiveClassifiers := Seq("sources")
```

## Troubleshooting

Always check the [tickets flagged as FAQ](https://github.com/ensime/ensime-sbt/issues?q=label%3AFAQ) before reporting a new issue.

Tickets are hard to reproduce unless you create an example project and share with us the steps to reproduce the problem.

Bug reports in the form of a pull request into the `src/sbt-test/ensime-sbt` directory are well received, which is our suite of integration tests using the [sbt scripted](http://eed3si9n.com/testing-sbt-plugins) framework. Pull requests with tests and fixes are living the dream.

### Starting the Server

1. Update this plugin and re-run the `.ensime` generator
1. If that doesn't work: exit `sbt`, clear out the sbt caches and recreate `.ensime` (change `0.13` by `1.0` if you are using sbt 1.0):

```
sbt> exit

rm -rf ~/.coursier/cache/v1/https/oss.sonatype.org/content/repositories/snapshots \
       ~/.ivy2/cache/org.ensime \
       ~/.ivy2/local \
       project/target \
       ~/.sbt/0.13/target \
       ~/.sbt/0.13/plugins/target

$ sbt
sbt> ensimeConfig
```

If **that** doesn’t work, move your ivy and coursier folders aside, the caches have probably become corrupted:

```
mv ~/.ivy2 ~/.ivy2.bak
mv ~/.coursier ~/.coursier.bak
```

### Cancel Processes

By default, `sbt` will not cancel a running subprocess if you type `C-c`. You are recommended to add the following to your `~/.sbt/0.13/global.sbt` so that it will do so:

```scala
cancelable in Global := true
```

Emacs users should recall that in order to send a control sequence to the `sbt-mode` subprocess, first run `sbt-clear` (bound to `C-c C-v` by default) then `C-c`. i.e. to send `C-c` to `sbt`, type `C-c C-v C-c C-c` (you probably want to bind this to something easier).


### sbt Version

Your `project/build.properties` needs to use, at least, a recent version of 0.13.x

```
sbt.version=0.13.17
```

or

```
sbt.version=1.1.1
```

## Contributing

You can check what runs in the CI by investigating the `.drone.yml`, `appveyor.yml` and `.travis.yml` files. These should give you a good feel for what to run locally.

When submitting a PR, we expect an accompanying test in the `sbt-test` folder. These use the [sbt scripted](http://eed3si9n.com/testing-sbt-plugins) plugin and you can start by copying one of the existing tests (use symbolic links instead of copying files).

To try your build locally, use `sbt publishLocal` (which will also happen as part of running `sbt scripted`) and make your required `sbt-ensime` version match that of the snapshot you are building. Remember to nuke your `~/.ivy2/local` once the fix is merged and published upstream.

Feel free to ask support questions in the [gitter room](https://gitter.im/ensime/home) for the editor that you use (there is no dedicated chat room for `sbt-ensime`). Also feel free to post *reproducible* bugs on the GitHub issue tracker.
