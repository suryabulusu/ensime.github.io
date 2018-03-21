---
layout: section
order: 4
title: User Guide
---

- TOC
{:toc}


## Scala REPL

First, ensure that the classes you want to access are compiled, type `C-c C-v z` to launch the embedded Scala REPL. The REPL should be launched with all your project classes loaded and available.

The SCALA REPL mode highlights stacktraces and lets you jump to source locations.


## Understanding the Server

### How the typechecker works

ENSIME uses the Scala presentation compiler, a light-weight version of the full compiler that only performs the first few stages of a compilation. It returns syntax/type errors, as well as the type of every expression, in fractions of a second rather than several seconds. This is how you receive instant feedback when you edit a `.scala` file.

In order to parse a source file correctly, the presentation compiler needs to know the type of every external class or object the source file uses. That information must be kept up-to-date whenever you make changes.

The presentation compiler loads type information from `.class` files, and only load the source files that are being edited. This only works if your project has been compiled, otherwise ENSIME will show you lots of spurious errors.

The compiler's state will be up to date regarding the file being edited. You should recompile whenever you want to see the effects of inter-file dependencies. When you recompile, ENSIME notices and reloads the new classes.

## sbt

There is support for starting and interacting with sbt inside Emacs, provided by [`sbt-mode`](/editors/emacs/sbt-mode).

This section highlights some of the ENSIME specific extensions and integrations with `sbt-mode`.

Commands beginning `C-c C-b` are used to communicate with sbt. Type `C-c C-b C-h` to see the full list.

The extra sbt integration enables some convenient workflow options:

- For small to medium projects, you can customize `ensime-sbt-perform-on-save` to `"compile"`. This is much more efficient than using sbt's `~compile` polling support.
- For larger projects, compiling on every save may not be practical. You can type `C-c C-b c` every once in a while.
- If your workflow involves running tests often, this will keep your project compiled as a side effect:
  - Type `C-c C-b t` to run all or one of the test suites.  If in a test source file, only its test suite is run.
  - Type `C-c C-b o` to run only one unit test.  This is the `testOnly` command in sbt.  When visitng a test source file and running this command, ENSIME automatically determines its name and only runs that test.
  -  Type `C-c C-b q` to run "testQuick", which only runs the recently failed tests, the tests that weren't run or need to be run next.

For more information on the sbt testing commands, see [sbt Testing](http://www.scala-sbt.org/0.13/docs/Testing.html)

- Type `C-c C-b r` and ENSIME will execute the main "run" tasks in sbt for the project.

## Editing

### Error highlighting

ENSIME will highlight errors and warnings in source files through the use of the Scala presentation compiler, a lightweight version of the Scala compiler. This is triggered in several ways:

- when you save a file
- when you type `C-c C-c c`
- after a short pause in typing. The frequency of these checks is controlled through the variables `ensime-typecheck-idle-interval` and `ensime-typecheck-interval`. This 
feature can be disabled by setting `ensime-typecheck-when-idle` to `nil`.

Hovering the mouse over a highlighted error or typing `C-c C-v e` displays the error details in the echo area.

### Semantic highlighting

ENSIME's semantic highlighting adds true colour-coding based on the compiler's analysis of the source code.

Semantic Highlighting is enabled by default. To disable it, set the customization variable `ensime-sem-high-enabled-p` to `nil`. When Semantic Highlighting is enabled, colours are refreshed every time you save the file.

To customise the colours used during semantic highlighting, change the value of the variable `ensime-sem-high-faces`, which is a list of `(symbolType . face)` associations.

### Implicit conversion highlighting

ENSIME highlights implicit conversions during semantic highlighting. Hover over a highlighted area or type `C-c C-v e` to see the details of the implicit conversion.

### Code completion

ENSIME completion is initiated automatically (via the `company-mode` backend) or by pressing the `TAB` key. Currently this works for local variables, method parameters, unqualified method names, and type names. An alternative completion backend is provided for users of the `ac` (auto complete) package, but it is unmaintained.

### Summarising the file

ENSIME provides an `imenu`-compatible summary of your source file, accessible through `M-x imenu` and compatible viewers. We recommend [`popup-imenu`](https://github.com/ancane/popup-imenu) as it gives a visual overview of your file.

## Refactoring

Refactoring is provided by the [github.com/scala-ide/scala-refactoring](https://github.com/scala-ide/scala-refactoring) library.

### Add type to symbol

Place your cursor over the symbol you'd like to add the type and type `C-c C-r a`.

### Rename

Place your cursor over the symbol you'd like to rename. Type `C-c C-r r` and follow the minibuffer instructions.

Note that you must enable the "find usages" feature in your build tool plugin for this to work across files.

### Organize imports

Type `C-c C-r o` in a Scala source buffer. Follow the minibuffer instructions.

### Extract method

Select a region by setting the mark using `C-SPC` and then placing the point at the end of the region. All selected code will be extracted into a helper method. Type `C-c C-r m` and follow the minibuffer instructions.

### Inline local

Place your cursor over the local val whose value you'd like to inline. Type `C-c C-r i` and follow the minibuffer instructions.

### Documentation browsing

Type `C-c C-v d` to browse a symbol's Javadocs / Scaladocs in your browser.

You can also view a list of all the documentation available for your project by typing `M-x ensime-project-docs`.

### Type and method search

Type `C-c C-v v` to start a global search. This should provide a live search view with support for camel case. We'd really like to integrate this search with more conventional search feedback systems (e.g. ido / helm), see [ensime-emacs#186](https://github.com/ensime/ensime-emacs/issues/186).

### Jump to test

ENSIME allows you to quickly jump to a class' unit test. With the cursor inside a class definition, type `C-c C-t t` to go to the corresponding test class. If the class doesn't exist, a new file will be created. With the cursor inside a test class, type `C-c C-t i` to go to the corresponding implementation class.

If your project uses an unusual convention, you must customize `ensime-goto-test-configs` or `ensime-goto-test-config-defaults`. These variables also specify the template to use when creating new test classes.

### Jump to definition

Press `M-.` with the cursor on a variable, method, type, or package to visit that object's definition. `M-,` returns to where you were.

If this doesn't work for you, please try to create a fully reproducible example and submit it as a ticket on the server at [ensime-server#492](https://github.com/ensime/ensime-server/issues/492). We know it sometimes breaks, but we need solid reproduction cases to investigate further.

## ElDoc

ElDoc shows hints regarding the current line of code in the echo area. It can be enabled by setting `ensime-eldoc-hints` to `'all`, `'error`, `'type` or `implicit`. This also limits the hints shown to the resepective type of messages.

## Helm/Ivy

To enable helm/ivy support use `(setq ensime-search-interface 'helm)` or `(setq ensime-search-interface 'ivy)` in your configuration.
Alas not a lot is changed by this option. This will hopefully change in the future.

## Debugging

### Breakpoints

With your cursor on a line of Scala source, type `C-c C-d b` to set a breakpoint. Type `C-c C-d u` to remove the breakpoint. Note that breakpoints can be added and removed outside of any debug session. Breakpoints are not, however, persisted between runs of ENSIME.

### Launching the debugger

Launch your application using your build tool with debugging enabled. e.g. via the [sbt-ensime debugging](http://ensime.github.io/build_tools/sbt/#debugging-example) support.

Then attach ENSIME to the running process with `M-x ensime-db-attach`. The first breakpoint your program hits will be highlighted and centered in Emacs.

### Run control

Type `C-c C-d c` to continue after hitting a breakpoint, or `C-c C-d s` to step into the current line, or `C-c C-d n` to step to the next line, or `C-c C-d o` to step out of the current function.

### Value inspection

When execution is paused, with your cursor over a local variable, type `C-c C-d i` to inspect the runtime value of a variable.

### Show backtrace

When execution is paused, type `C-c C-d t` to display the current backtrace.
