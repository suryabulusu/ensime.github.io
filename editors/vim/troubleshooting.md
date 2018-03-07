---
layout: section
order: 10
title: Troubleshooting
---

Thank you for using ENSIME on Vim!

~~Vimfidel, have you considered using Emacs?~~ (Sam, get out of here!)

Most tickets can be resolved easily by following a simple process.

0. Fully compile your project.
1. Nuke old versions of the ENSIME server.
   - `rm -rf ~/.coursier/cache/v1/https/oss.sonatype.org/content/repositories/snapshots/org/ensime/`
2. Use the latest release of `ensime` for Vim (i.e. update `ensime-vim` via your favorite package manager).
3. Sometimes we update the command interface for the plugin. This can break Neovim, so run `:UpdateRemotePlugins` and restart nvim.
3. Check the [tickets flagged as FAQ for Vim](https://github.com/ensime/ensime-vim/issues?labels=FAQ).
4. Check the [tickets flagged as FAQ on the server](https://github.com/ensime/ensime-server/issues?labels=FAQ).
5. Search for duplicates, especially the [most recently updated tickets](http://github.com/ensime/ensime-vim/issues?direction=desc&sort=updated).

#### Common Issues

##### I get an error that "`ensime_plugin` is not defined", or there are no `:En*` commands in Vim

If you're using Neovim, try `:UpdateRemotePlugins`.

If not, or that didn't fix the problem, ensure you installed the Python dependencies:

```
$ pip install websocket-client sexpdata
```

Also, confirm that [the version of Python your Vim is built with][vim python version] is the same one that you're running `pip install` with. Check `:py import sys; print(sys.path)` and `pip show sexpdata | grep Location`.

##### Screen Flickers When Printing Compile Errors/ Warnings

If you are using Syntastic, check the value of `syntastic_full_redraws`, as this determines Syntastic's redraw behavior on `SyntasticCheck`. If you are not using Syntastic and seeing flicker please open an issue.

##### Ensime Server Won't Start

The server will fail to start if any of these files still exist in the `.ensime_cache` directory: `http`, `port`, `server.pid`. If you don't see ensime running but those files are present, delete them and you should be all set.

##### My files don't typecheck on write anymore

We recently (Spring 2016) decided that typechecking on write wasn't a sane default. If you'd still like this behavior, just add the following line to your Scala autocommand group:

```
au BufWritePost *.scala :EnTypeCheck
```

##### The Plugin won't download ENSIME server

The bootstrapping process can be slow for ENSIME, but only needs to be done once per Scala version. To ensure we don't repeat any unnecessary work we check for the presence of a classpath-file in `~/.config/ensime/<ScalaVersion>`, where `<ScalaVersion>` is pulled from the `.ensime` config in question. If the classpath file already exist then the `ensime` jars will not be downloaded. Remove the version directory in question if you want to re-pull the jars (after cleaning your Ivy cache of org.ensime of course).

If these solved your problem, great!

If not, please join the conversation on [the ensime-vim Gitter chat](https://gitter.im/ensime/ensime-vim). You will benefit by preemptively following the steps outlined in the [bug report template](https://github.com/ensime/ensime-vim/blob/master/.github/ISSUE_TEMPLATE.MD).

If you think you've found a bug you can [file and issue](https://github.com/ensime/ensime-vim/issues/new) (you'd don't need to ask on the channel, just go right ahead and create a ticket).


[vim python version]: http://stackoverflow.com/questions/10864042/how-to-check-python-version-that-vim-was-compiled-with
