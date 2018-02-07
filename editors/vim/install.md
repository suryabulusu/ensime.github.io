---
layout: section
order: 2
title: Installation
---

You will need a version of Vim built with Python plugin support. This is standard on most distribution packages these days. Try `vim --version | grep +python` to confirm.

If you're using Ubuntu, as of 16.04, default Vim doesn't have Python configured. [See here][python vim ubuntu] for options to get Vim with Python. Note you need **Python 2** support. Make sure pip installs below are for Python 2 as well (you might need to use `pip2`).

1. Install external Python modules that `ensime-vim` requires to function:

    ```
    $ pip install websocket-client sexpdata
    ```
1. If you use Neovim, additionally ensure you've installed the `neovim` module:

   ```
   $ pip install neovim
   ```

1. If you're using Neovim with [Nix][nix] you'll need to override the package to ensure that the Python
   interpreter uses the `websocket-client` and `sexpdata` modules.
   This can be achieved easily using [`overlays`][nixOverlays]:

   ``` nix
   self: super:

   {
     neovim = self.neovim.override {
       extraPythonPackages = with self.pythonPackages; [ websocket_client sexpdata ];
     };
   }
   ```

   Make sure to rebuild Neovim after that.

1. Install the ENSIME plugin for your [build tool](/build_tools)
1. Add ensime-vim to your plugin manager of choice in your `.vimrc` (or `init.vim` for Neovim):

    Plugin Manager                                    | Your .{n}vimrc
    --------------------------------------------------|-------------------------------
    [Vim-Plug](https://github.com/junegunn/vim-plug)  | `Plug 'ensime/ensime-vim'`
                                                      | `Plug 'ensime/ensime-vim', { 'do': ':UpdateRemotePlugins' } # if Neovim`
    [Vundle](https://github.com/VundleVim/Vundle.vim) | `Plugin 'ensime/ensime-vim'`
    
   Under the hood this is just another Vim plugin, so other plugin managers should work great as well.
1. Install the [vim-scala] plugin for Scala filetype detection and highlighting. Optionally, install [Syntastic] as well for syntax checking integrated with ENSIME. Instructions for both plugins can be found in their respective documentation.
1. Trigger the plugin installations after updating your configuration. With `vim-plug` this is done by executing `:PlugInstall`. With `Vundle` run `:PluginInstall`.

##### Post-Install Config

Depending on your preferences, you may want to add some of the following to your `.vimrc`

 - Typechecking after writing

    ```
    autocmd BufWritePost *.scala silent :EnTypeCheck
    ```
 - Easy Type Inspection

    ```
    nnoremap <localleader>t :EnType<CR>
    ```
 - Customize the browser used for `:EnDocBrowse` by setting an environment variable, in your shell initialization

    ```
    $ export BROWSER="firefox %s"
    ```

   By default the plugin tries to make a good guess for your platform (see the Python [webbrowser module] for details). You can also [point the variable to a script][browser-script] for something fancier.

[python vim ubuntu]: http://askubuntu.com/questions/764882/ubuntu-16-04-vim-without-python-support
[vim-scala]: https://github.com/derekwyatt/vim-scala
[Syntastic]: https://github.com/scrooloose/syntastic
[webbrowser module]: https://docs.python.org/2/library/webbrowser.html
[browser-script]: https://github.com/ensime/ensime-vim/pull/226#issuecomment-207468659
[nix]: https://nixos.org/nix
[nixOverlays]: https://nixos.org/nixpkgs/manual/#chap-overlays
