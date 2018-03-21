---
layout: page
order: 6
title: Supported Text Editors
site_nav_entry: true # this is an entry in the main site nav
---

ENSIME supports several editors. Your choice of editor is a very personal thing and you should probably go with the one that feels right for you... *even if deep down you know that it's not as good as Emacs*.

- [Emacs](emacs) ([homepage][emacs])
- [Atom](atom) ([homepage][atom])
- [Vim](vim) ([homepage][vim])
- [VSC](vscode) ([homepage][vsc])
- [Sublime](sublime) ([homepage][subl])

If you really want to use a particular editor but some feature is missing, take it from us that you will find it incredibly rewarding to get it working! Head over to [`ensime-server` Contributing Guide](/server/contributing) to find out more.

## Feature Matrix

The following chart describes how well ENSIME supports each editor:

[emacs]: http://www.gnu.org/software/emacs/
[emacsI]: /talks/scalasphere16/images/emacs.svg
[atom]: https://atom.io/
[atomI]: /talks/scalasphere16/images/atom-logo.svg
[vim]: http://www.vim.org/
[vimI]: /talks/scalasphere16/images/vim-logo.svg
[vsc]: https://code.visualstudio.com/
[vscI]: https://upload.wikimedia.org/wikipedia/commons/f/f3/Visual_Studio_Code_0.10.1_icon.png
[subl]: https://www.sublimetext.com/
[sublI]: /talks/scalasphere16/images/sublime-logo.svg
[GPL]: http://www.gnu.org/licenses/gpl.en.html
[MIT]: https://opensource.org/licenses/MIT
[Charityware]: http://vimdoc.sourceforge.net/htmldoc/uganda.html#license
[MIT]: https://opensource.org/licenses/MIT
[Find usages]: https://github.com/ensime/ensime-server/issues/425
[Show implementations]: https://github.com/ensime/ensime-server/issues/1131
[Advanced type search]: https://github.com/ensime/ensime-server/issues/472
[Passive suggestions]: https://github.com/ensime/ensime-server/issues/848
[Whitebox macros]: https://github.com/fommil/imaginary-friend
[Apache2]: https://www.apache.org/licenses/LICENSE-2.0

| Learn more →           | [Emacs](emacs)            | [Atom](atom)           | [Vim](vim)          | [VSC](vscode)       | [Sublime](sublime)        |
| ---------------------- | ------------------------- | ---------------------- | ------------------- | ------------------- | ------------------------- |
| Homepage               | [![Emacs][emacsI]][emacs] | [![Atom][atomI]][atom] | [![Vim][vimI]][vim] | [![VSC][vscI]][vsc] | [![Sublime][sublI]][subl] |
| Main language          | Emacs Lisp                | Coffeescript           | Python              | Typescript          | Python                    |
| Upstream Licence       | [GPL]                     | [MIT]                  | [Charityware]       | [GPL]/[Apache2]     | Proprietary               |
| Greetings              | ✔                         |                        |                     |                     |                           |
| Type at point          | ✔                         | ✔                      | ✔                   | ✔                   | ✔                         |
| Contextual completion  | ✔                         | ✔                      | ✔                   | ✔                   | ✔                         |
| Live errors / warnings | ✔                         | ✔                      | ✔                   | ✔                   | ✔                         |
| Semantic highlighting  | ✔                         |                        |                     |                     |                           |
| Implicit conversions   | ✔                         | ✔                      |                     |                     |                           |
| Import class at point  | ✔                         | ✔                      | ✔                   |                     | ✔                         |
| Classpath search       | ✔                         | ✔                      | ✔                   |                     | ✔                         |
| Jump to source         | ✔                         | ✔                      | ✔                   | ✔                   | ✔                         |
| Show documentation     | ✔                         | ✔                      | ✔                   |                     | ✔                         |
| Rename symbol          | ✔                         |                        | ✔                   |                     | ✔                         |
| Organise imports       | ✔                         | ✔                      | ✔                   |                     | ✔                         |
| Debugging              | ✔                         |                        | Partial             |                     |                           |
| Java Support: ENJIME   | ✔                         |                        |                     |                     |                           |
| REPL                   | ✔                         |                        |                     |                     |                           |
| sbt integration        | ✔                         |                        |                     | ✔                   |                           |
| Find usages (opt in)   | ✔                         |                        |                     |                     | ✔              |
| Show implementations   | ✔                         |                        |                     |                     | ✔              |
