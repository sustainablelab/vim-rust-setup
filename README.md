# About

Notes on Vim setup for programming in Rust.

* [Rust Installation](#rust-installation)
* [Vim Plugin Management](#vim-plugin-management)
* [Install a Rust package and two Vim plugins](#install-a-rust-package-and-two-vim-plugins)

# Rust installation

## Install Rust

Installing Rust gets you `rustup`, `rustc`, `cargo`, `clippy`, `rust-docs`, `rust-std`, and `rustfmt`:

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ source "$HOME/.cargo/env"
```

*This creates hidden folders `~/.rustup` and `~/.cargo`. It also modifies
`~/.profile, ~/.bash_profile, and ~/.bashrc`. You can undo all of these changes
(e.g., when Rust breaks and you need to do a clean install) by running `rustup
self uninstall`.*

## Install Extra Rust Stuff

Install Rust package `cargo-watch` (so you can automatically run your program each time you save):

```
$ cargo install cargo-watch
```

*Now from the command line you can do `cargo watch -cx test` to run unit tests each time you save.*

If you decide to use an
[LSP](https://microsoft.github.io/language-server-protocol/) server (i.e.,  to
use Vim ALE with Rust), you will also need `rust-src` and `rust-analyzer`:

```
$ rustup component add rust-src
$ rustup component add rust-analyzer
```

*This is analogous to installing the VSCode `rust-analyzer` extension.*

## Quick Overview of Rust Tooling

### Open the Rust docs

* `$ rustup doc --book` -- Go straight to the Rust Book
* `$ rustup doc --std` -- Go straight to the `std` reference docs, and use the search bar
* `$ rustup doc --reference` -- Look up precise language definitions

### New Rust project

Start a new project:

```
$ cargo new my_project_name
```

### Add dependencies

Add dependencies in `Cargo.toml`:

```
$ cd my_project_name
$ vim Cargo.toml
```

Example default `Cargo.toml`:

```toml
[package]
name = "my_project_name"
version = "0.1.0"
edition = "2021"

[dependencies]
```

Add a dependency on the latest version of SDL2:

```toml
[dependencies]
sdl2 = "*"
```

### Tell Git to ignore Rust build output

Create a `.gitignore`:

```
/target
```

*Build output goes in `./target/`, e..g, `target/release`, `target/debug`, `target/doc`.*

### Rust project documentation

Generate local project documentation (includes documentation for dependencies) and open in browser:

```
$ cargo doc --open
```

*Note that project documentation is build output, so it is ignored by Git.*

# Vim Plugin Management

This is how I set up Vim on Linux and Windows.

*I say Windows but I'm still working in a POSIX environment, either Cygwin,
MSYS2, or WSL2, depending on whether I'm building for Windows or Linux. It's
been a while since I've touched Cygwin. The steps below definitely work on
MSYS2 and WSL2.*

## Make the `.vim` folder a Git repo

Vim is usually customized in a `~/.vimrc` file. Instead of this:

* I create a `~/.vim` folder
* I save my VIMRC file as `~/.vim/vimrc`
    * Note the `vimrc` filename does not have a dot prefix!
* I turn the `.vim` folder into a Git repository.

I keep a private copy of my `.vim` folder on GitHub in a repo named
`vim-dotvim.git` for
[quickly reproducing my Vim setup on a new machine](#reproduce-my-vim-setup-on-a-new-machine).

## Handle Vim plugins as Git submodules

I manage Vim plugins by adding them as Git submodules.

*There are popular plugins for managing Vim plugins, but this method works for
me. It is simple, straight-forward, and does not introduce an additional plugin
dependency. If you are comfortable with Git submodules, give this method a
shot.*

This uses a package mechanism introduced in Vim 8. Plugins go in the
`~/.vim/pack/bundle/start/` folder. Make this folder:

```
$ mkdir -p ~/.vim/pack/bundle/start/
```

### Add a plugin

Add a plugin with `git submodule add`:

```
$ cd ~/.vim/
$ git submodule add https://github.com/tpope/vim-commentary.git pack/bundle/start/vim-commentary/
```

*Note Vim plugins go in the `~/.vim/pack/bundle/start/` folder.*

### Update a plugin

To update a Vim plugin, get the latest version as you would with any Git repo:

```
$ cd ~/.vim/pack/bundle/start/vim-commentary/   # Enter the plugin folder
$ git remote update                             # Fetch latest info about the remote
$ git status                                    # Double-check you haven't made local changes yet
$ git pull                                      # Update local copy
$ cd ~/.vim                                     # Return to my .vim repo
$ git commit -m 'Update Git submodule vim-commentary' # Commit: using new plugin version
$ git push                                      # Push updated .vim repo to remote vim-dotvim.git
```

*Note: updating a plugin (`git pull`) also changes the `.vim` repo because it
is now using a different version of one of its plugins. Git automatically
stages this modification to `.vim` for you, but you still have to commit and push.*

## Reproduce my Vim setup on a new machine

Setting up Vim on a new machine is just four easy steps:

Step 1: Install Vim (see caveat below):

```
sudo apt install vim
```

Step 2: I clone my private `vim-dotvim.git` repo as `~/.vim`:

```
cd ~
git clone github:sustainablelab/vim-dotvim .vim
```

Step 3: I pull in all of my Vim plugins like this:

```
$ git submodule update --init --recursive
```

Step 4: Start Vim. Generate help tags for all of my plugins (and for my own Vim help files):

```vim
:helptags ALL
```

*Caveat: the above four steps get me 90% of the way there and Vim is usable.
But the Vim available via the usual package installation method (e.g., `sudo
apt install vim`) is often missing features I want, so I end up building Vim
from source. This is not a big deal!*

*Clone https://github.com/vim/vim. Get the Vim build dependencies with `sudo
apt build-dep vim`. Build from source -- `make` -- run the tests -- `make test`
-- then install -- `sudo make install` (puts Vim in `/usr/local`). The hardest
part (and still not that bad) is figuring out what to edit in the
`vim/src/Makefile` to enable the desired features. After making changes to the
Makefile: rebuild with -- `make reconfig` -- and reinstall with `sudo make
install`.*

# Install a Rust package and two Vim plugins

The Vim environment setup is just one Rust package, two Vim plugins, and then
some VIMRC customization.

1) `rusty-tags` : Rust package for making **tags** files
    1) [`rusty-tags`](https://github.com/dan-t/rusty-tags)
    generates tags files. It requires `ctags` and will work with either
    `exuberant-ctags` (more common) or `universal-ctags` (preferred).
1) `rust.vim` : Vim plugin for **file detection** and **syntax highlighting**
    1) This is the official Rust plugin! The *rust-lang.org* website points you
    to the [rust.vim syntax plugin](https://github.com/rust-lang/rust.vim) on
    their [tools page](https://www.rust-lang.org/tools).
1) `ALE` : Vim plugin for **linting** (uses the `rust-analyzer` LSP)
    1) There are several Vim linting plugins. I'm still new to this LSP stuff.
    I've only ever tried [`ALE`](https://github.com/dense-analysis/ale). Seems
    alright.
    1) Skip `ALE` unless you *really* want to replicate the VSCode experience:
        1) language-specific omni-complete
        1) opening documentation in the Vim preview window
        1) linting
            1) Linting means **way more** than auto-formatting code on a save
            to follow Rust conventions about white-space and curly braces. The
            `rust.vim` plugin already does that. This `ALE` linting means the
            compiler is constantly crunching on your Rust buffer as you make edits.
            1) I don't really understand what's going on under the hood. I
            think the `rust-analyzer` (and `clippy` if you add
            `#![deny(clippy::thing_to_complain_about)]` attributes all over
            your code) does partial builds every time you edit a Rust buffer.
            Read the [`rust-analyzer` docs](https://rust-analyzer.github.io/book/).
            1) However it works, it enables the following neat features:
                1) Put untouchable comments in my buffer (Vim's equivalent of
                the VSCode lightbulbs) explaining errors.
                1) Color my code to flag warnings/errors (Vim's equivalent of
                the VSCode yellow and red squiggles).
                1) Display data types (more untouchable text in my buffer).
            1) While this is really neat, I don't generally want this.
            1) I think Rust is the one case where I might want this. The
            compiler is **so good** it will teach me idiomatic Rust if I give
            it a way to talk to me. Setting up linting is one way to let the
            Rust compiler talk to me.
            1) I'm using LSP with Rust (for now), but I'm still on the fence
            about using LSP in general. Maybe it's just what I'm used to, but
            syntax highlighting and code-hopping with tags is kind of all I
            need (the tags also gives limited autocompletion, just using tags).
            Even for Rust, I can see turning off the LSP because the other
            out-of-the-box Rust tooling is so good. Rust's package manager
            `cargo` makes it easy to open documentation for the codebase, which
            includes all of the dependencies listed in `Cargo.toml`. And the
            compiler messages are excellent.

I manage Vim plugins by [adding them as Git submodules](#handle-vim-plugins-as-git-submodules),
so all of the "install plugin" instructions below are `git submodule add PLUGIN-REPO PLUGIN-LOCATION`.

## Official Rust syntax plugin `rust.vim`

`rust.vim` : Vim plugin for **file detection** and **syntax highlighting**

### Install `rust.vim`

Install the "official" syntax plugin, `rust.vim`:

```
$ cd ~/.vim/
$ git submodule add https://github.com/rust-lang/rust.vim pack/bundle/start/rust.vim
```

### Read the `rust.vim` docs

Start Vim. Read Vim's Rust syntax documentation to see which global settings you would like to enable:

```vim
:h rust
```

### Edit your VIMRC for `rust.vim` global settings

Set your Rust global settings by editing your VIMRC.

Fix code formatting on save:

```vim
let g:rustfmt_autosave = 1 " run :RustFmt automatically when saving a buffer
```

*This setting is off by default. It is the only change I make to the default settings.*

## Rust tags package

*Generate tags files so you can use all the usual Vim tag-hopping shortcuts in Rust codebases.*

### Install package `rusty-tags`

Install the package:

```
$ cargo install rusty-tags
```

#### You also need `ctags`

Package `rusty-tags` uses `ctags`. If you don't have `ctags`, install `universal-ctags`:

```
sudo apt install universal-ctags
```

I have two versions of `ctags` installed:

```
/usr/bin/ctags-exuberant
/usr/bin/ctags-universal
```

My `ctags` points to `ctags-exuberant`:

```
$ stat /usr/bin/ctags
  File: /usr/bin/ctags -> /etc/alternatives/ctags
$ stat /etc/alternatives/ctags
  File: /etc/alternatives/ctags -> /usr/bin/ctags-exuberant
```

### Generate Rust tags

Enter any project folder with a `Cargo.toml` file and generate tags like this:

```
$ rusty-tags vi
```

I add this to my VIMRC to generate tags with `;tr<Space>` as I browse the codebase in Vim:

```vim
nnoremap <leader>tr<Space> :call RustyTags()<CR>
function RustyTags()
    exec "botright terminal rusty-tags vi"
    wincmd p " Go back to Rust window
    setlocal tags=./rusty-tags.vi;/ " Set 'tags' for this buffer only
endfunction
```


## Add Rust autocmds to VIMRC

I add this to my VIMRC to find the local rusty-tags file and to use it just for
this Rust buffer.

```vim
augroup vimrc
    autocmd! " Remove all vimrc autocommands
    " ... Some autocommands ...
    autocmd BufRead *.rs :setlocal tags=./rusty-tags.vi;/
    autocmd BufEnter *.rs :set foldmethod=syntax
    " ... Some more autocommands ...
```

*The first autocommand tells Vim to do this whenever I am editing a Rust
buffer.*

*The second autocommand tells Vim to use syntax folding as the code-folding
method whenever my cursor enters a Rust buffer.*

## `ALE` linting engine and `rust-analyzer` linter

*From the [`rust-analyzer` docs](https://rust-analyzer.github.io/book/):
"`rust-analyzer` is a library for semantic analysis of Rust code as it changes
over time." It can be run "as part of a server that implements the [Language
Server Protocol
(LSP)](https://microsoft.github.io/language-server-protocol/)."*


Use `rust-analyzer` as the language server and `ale` as the engine that tells
Vim to use `rust-analyzer` as the LSP server.


I already have these steps above, but I'm including them again to emphasize they are essential for ALE to work.

```
$ rustup component add rust-src
$ rustup component add rust-analyzer
```

Now install ALE:

```
$ cd ~/.vim
$ git submodule add github:dense-analysis/ale.git pack/bundle/start/ale
```

And add this to my VIMRC:

```vim
let g:ale_linters = {'rust': ['analyzer']} " ALE use 'rust-analyzer' as the LSP server
```

Generate Vim help for `ale`:

```vim
:helptags ALL
```

*Note: `:helptags ALL` is not specific to `ale`. I write my own Vim help files
as a way of documenting everything. `:helptags ALL` updates tags across all
help files.*

### Inspecting and controlling ALE

To see what ALE is doing for a filetype, put cursor in buffer of the filetype
and hit `:ALEInfo` -- this opens a preview window and you can hit `Space` on
any line to get help on that setting.

To completely toggle ALE on/off, do `:ALEToggle`. The syntax "help" can be
annoying sometimes, so I bind this to `;ra<Space>`:

```vim
nnoremap <leader>ra<Space> :ALEToggle<CR>:echomsg ">^.^< ALEToggle: g:ale_enabled="..g:ale_enabled<CR>
```

#### About `ALE`

* `ALE` is not just for Rust. In another filetype (`.c` for example), try
  `:ALEFixSuggest` to get a list of tools appropriate to this filetype.
* `ALE` is the Asynchronous Lint Engine -- see `:h ale`
  * `ALE` sends the contents of buffers to linter programs using the Vim help for
    job-control (Vim must be compiled with +job, +channel, and
    +timers).
  * Linting runs all the time

### Omnicomplete tips

* Omnicomplete: `i_CTRL-X_CTRL-O`
  * *Note: with `g:ale_completion_enabled = 1`, once the menu pops up it's as
    if you already hit `i_CTRL-X_CTRL-O` and now you need only hit `CTRL-N` or
    `CTRL-P` to navigate the list of options!*
