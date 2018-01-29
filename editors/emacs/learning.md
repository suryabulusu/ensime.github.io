---
layout: section
order: 2
title: Learning Emacs
---

If you are new to Emacs, be prepared for a very rewarding learning curve!

Firstly, install Emacs from [emacs.link](http://emacs.link).

Our goal in this page is to let you know about some excellent resources to learn Emacs and get you set up with some basic configuration.

If you're a Vim-fidel wanting to try out Emacs, you may feel at home in [Evil Mode](https://bitbucket.org/lyro/evil/wiki/Home) (we'll show you how to do that below).

Some useful resources to learn Emacs:

- The built in tutorial! Start Emacs and type `C-h t` (Control and `h`, then `t`)
- [GNU Emacs Manual](https://www.gnu.org/software/emacs/manual/emacs.html)
- [Visual tutorial](http://sachachua.com/blog/2013/05/how-to-learn-emacs-a-hand-drawn-one-pager-for-beginners/)
- [Mastering Emacs](https://www.masteringemacs.org/)
- [GNU Emacs Reference Card (pdf)](https://www.gnu.org/software/emacs/refcards/pdf/refcard.pdf)
- [Sacha's .emacs.d](http://pages.sachachua.com/.emacs.d/Sacha.html)
- [XahLee's ergoemacs](http://ergoemacs.org/)

The use of [EmacsWiki](http://www.emacswiki.org/) is discouraged as much of the content is out of date.

To truly feel comfortable with Emacs, you are recommended to read the [Emacs Lisp Manual](https://www.gnu.org/software/emacs/manual/elisp.html) (at least the language and core libraries). It will set you free.

The following is intended to set some built-in Emacs variables that often confuse / annoy new users and is by no means a fully functional development environment. Put this into your `~/.emacs.d/init.el` and restart Emacs. Make sure you do not have a stale `~/.emacs` file left over from a previous attempt to use Emacs.

```elisp
;; global variables
(setq
 inhibit-startup-screen t
 create-lockfiles nil
 make-backup-files nil
 column-number-mode t
 scroll-error-top-bottom t
 show-paren-delay 0.5
 use-package-always-ensure t
 sentence-end-double-space nil)

;; buffer local variables
(setq-default
 indent-tabs-mode nil
 tab-width 4
 c-basic-offset 4)

;; modes
(electric-indent-mode 0)

;; global keybindings
(global-unset-key (kbd "C-z"))

;; the package manager
(require 'package)
(setq
 package-archives '(("gnu" . "http://elpa.gnu.org/packages/")
                    ("org" . "http://orgmode.org/elpa/")
                    ("melpa" . "http://melpa.org/packages/")
                    ("melpa-stable" . "http://stable.melpa.org/packages/"))
 package-archive-priorities '(("melpa-stable" . 1)))

(package-initialize)
(when (not package-archive-contents)
  (package-refresh-contents)
  (package-install 'use-package))
(require 'use-package)
```

Inside Emacs Lisp files, you can type `C-h v` over any variable (e.g. in the `setq` and `setq-default` lists) to see its help page, and `C-h f` over any function to see its help page.

We recommend [`use-package`](https://github.com/jwiegley/use-package) to manage your configuration and as the recommended way to install packages. e.g. if you want to use `evil-mode`

```elisp
(use-package evil
  :demand)
```

you can execute this without restarting Emacs by moving the cursor ("point" in Emacs nomenclature) to the end of the line and typing `C-x C-e`.


## Updating MELPA

To update all [MELPA](http://melpa.org/#/) packages, type:

```
M-x list-packages [RETURN] U [RETURN] x [RETURN]
```

or invoke the `package-utils-upgrade-all` function from `package-utils`.

## General Emacs Help

Most of the time, you might find what you need from Emacs and not ENSIME. In these cases, the typical help forums are:

- [emacs.stackexchange.com](https://emacs.stackexchange.com/)
- [gnu.emacs.help](https://groups.google.com/forum/#!forum/gnu.emacs.help)
- [#emacs](https://www.emacswiki.org/emacs/EmacsChannel)

If you are uncomfortable engaging with any of these external communities (they do have some level of rules in place, such as [Stack Exchange](http://meta.stackexchange.com/help/be-nice) / [Emacs mailing list](http://www.emacswiki.org/emacs/EmacsMailingLists) / [IRC channels](https://www.emacswiki.org/emacs/EmacsChannel#toc3)), please feel free to talk about general GNU Emacs usage at [gitter.im/ensime/ensime-emacs](http://gitter.im/ensime/ensime-emacs).
