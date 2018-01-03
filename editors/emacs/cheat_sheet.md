---
layout: section
order: 1
title: Cheat Sheet
---

## Customisation

To customise your ENSIME Emacs experience, don't forget to read through the user-customisable variables with `M-x customize-group RET ensime RET`, as defined in [ensime-vars.el](http://github.com/ensime/ensime-emacs/blob/master/ensime-vars.el).

## Source

| Shortcut    | Description |
|-------------|-------------|
| `C-c C-v f` | Format the current source file |
| `C-c C-v t` or hover mouse | Show the type of the symbol under the cursor |
| `C-c C-v T` | Show the fully qualified type of the symbol under the cursor |
| `C-u C-c C-v t` | (universal argument adds to the kill ring) |
| `C-c C-v e` or hover mouse | Show compile warnings under the cursor ; show implicit conversions when applicable |

## Presentation Compiler

| Shortcut    | Description |
|-------------|-------------|
| `C-c C-c c` | Re-typecheck the current file |
| `C-c C-c r` | Restart the presentation compiler for all open files |

## Refactoring

| Shortcut    | Description |
|-------------|-------------|
| `C-c C-r a` | Add type to the symbol under the cursor |
| `C-c C-r o` | Organize imports in the current file |
| `C-c C-r t` | Import the type under the cursor |
| `C-c C-r r` | Rename the symbol under the cursor |
| `C-c C-r l` | Extract the region into a local value |
| `C-c C-r m` | Extract the region into a method |
| `C-c C-r i` | Inline the local value under the cursor |

## Navigation

| Shortcut    | Description |
|-------------|-------------|
| `M-.` or `Control+Left-Click` | Jump to the definition of the symbol under the cursor |
| `C-u M-.` | Jump to the definition of the type of the symbol under the cursor |
| `C-c C-v r` | Find usage of symbol at point |
| `C-c C-v h` | Find implementations of type at point |
| `M-,` | Pop back to the previously visited position |
| `C-c C-v .` | Select the surrounding syntactic context. Subsequent taps of '.' and ',' will grow and shrink the selection, respectively |
| `C-c C-v v` | Search globally for methods or types |

## Documentation

| Shortcut    | Description |
|-------------|-------------|
| `C-c C-v d` | Browse the documentation of the symbol under the cursor |

## sbt

Much of this is a wrapper over [`sbt-mode`](https://github.com/hvesalai/sbt-mode), so don't forget that you also have commands provided as-is by that package.

| Shortcut    | Description |
|-------------|-------------|
| `C-c C-b s` or `C-c C-v s` | Start a sbt process, or switch to an existing one |
| `C-c C-b S` | Switch to buffer containing the stack trace parser |
| `C-c C-b c` | Compile |
| `C-c C-b C` | Compile just this file |
| `C-u C-c C-b C` | Compile just this file and print its AST |
| `C-c C-b n` | Clean |
| `C-c C-b p` | Package |
| `C-c C-b r` | Run |
| `C-c C-b T` | Run all tests |
| `C-c C-b t` | Run the current module/suite tests |
| `C-c C-b q` | Run "quick" tests (tests impacted by recent changes) |
| `C-c C-b o` | Run only the current test |


## Debugger

| Shortcut    | Description |
|-------------|-------------|
| `C-c C-d d` | Start the debugger |
| `C-c C-d b` | Set a breakpoint at the current line |
| `C-c C-d u` | Remove the breakpoint at the current line |
| `C-c C-d a` | Remove all breakpoints |
| `C-c C-d r` | Start debugging the current program |
| `C-c C-d s` | Step into method invocations |
| `C-c C-d o` | Step out of method invocations |
| `C-c C-d n` | Step to the next line |
| `C-c C-d c` | Continue the program execution |
| `C-c C-d q` | Stop debugging the current program |
| `C-c C-d t` | Show the current backtrace |
| `C-c C-d i` | Inspect the value of the symbol under the cursor |

## REPL

| Shortcut    | Description |
|-------------|-------------|
| `C-c C-v z` | Start the Scala REPL or switch to it |
| `C-c C-v C-r` | Send the region to the REPL |
| `C-c C-v b` | Evaluate the current buffer in the REPL |
| `C-c C-v l` | Prompt for a file and load it in the REPL |

## Miscellaneous

| Shortcut    | Description |
|-------------|-------------|
| `C-c C-b S` | Create a "stacktrace" buffer or switch to it |
| `M-x ensime-shutdown` | Shut down ENSIME |
| `M-x ensime-reload` | Reload the .ensime file and recompile the project |


