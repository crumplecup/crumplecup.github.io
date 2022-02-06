+++
title = "Latex Flow"
date = 2022-02-04
[taxonomies]
categories = ["Latex"]
tags = ["info"]
[extra]
author = "Erik Rose"

+++

# Latex - Better Than Reinventing the Wheel

Latex first entered onto my radar in graduate school as research papers began to strain my relationship with MS Word. First I used a number of desktop publishing programs, starting with Adobe InDesign and trying a few open source alternatives, and these options offer better presentation, but at the cost of program size and complexity.  Support for features like auto-numbering figures and tables was non-existent, making it an ultimate non-starter for research.  When I poked around on the internet looking for what other people used to draft and edit their thesis, Latex came up multiple times. "This is what the professionals use" I thought, and resolved to figure it out.

My first negative impression was not struggling to get it to compile (that would come later). Rather, the verbosity and boilerplate quality of the syntax immediately put me off, in the same way that seeing the html tags add a level of visual complexity that can make it hard to follow the content on a website. It wasn't the complexity, though, so much as the extra typing that really stuck in my craw. I flirt enough with RSI as it is. These notes are the changes that I made to my development environment to reduce computer-related stress by minimizing the need for repetitive typing and mouse-use.

Features of this setup include:
 * autocompletion for frequently-used code blocks
 * syntax checking for paired braces etc.
 * spell-checking and syntax highlighting


# Editor

Previously I used [RStudio](https://www.rstudio.com/) for R and [IntelliJ](https://www.jetbrains.com/idea/) for everything else code related. These programs are feature rich, but bloated and heavy, consuming significant amounts of RAM. Latex editors that I tried, including [Scribus](https://www.scribus.net/), are light-weight but not feature-rich enough. [NeoVim](https://neovim.io/) is a current iteration of the storied Vim text editor, modernized to include many of the features of IntelliJ, VSCode and other IDEs, while remaining light-weight, responsive and powerful. Vim works very differently from other text editors, and this can be a good thing, but the lack of common ground with prior experience can slow down the learning process, especially at first.  I recommend the article [Learn Vim for the Last Time](https://danielmiessler.com/study/vim/) by Daniel Miessler, which makes the case for why to use Vim better than I can.

Instead of a settings menu, Vim uses a configuration file, allowing maximum flexibility in customization. But if you are not used to mucking around in configuration files, getting your settings just right will take trial and error. On my system with Neovim, the location in is ~/.config/nvim/init.vim. Learning to use the default configuration is helpful when you are working on new or random machines, but selective customization is key to an ergonomic and productive flow.

## Plugins

The preamble of my init.vim file is the plugin invocation. Plugins extend the functionality of Vim for specific use cases. The [vim-plug](https://github.com/junegunn/vim-plug) plugin manager is popular and works without issue on my machine (it should also work on MacOS according to [this article](https://dev.to/dafloresdiaz/neovim-for-macos-3nk0)). The most relevant plugins for my latex workflow are [vim-tex](https://github.com/lervag/vimtex) for compiling tex files to pdfs and displaying them in a viewer, [ultisnips](https://github.com/SirVer/ultisnips) for snippet completion, and [vim-lsp](https://github.com/prabirshrestha/vim-lsp) for syntax checking.

Using [vim-plug](https://github.com/junegunn/vim-plug) requires separate installation, although you should be able to copy and paste the necessary commands for your system from the instructions on the github site into your terminal. Once installed, use the following code at the start of init.vim to invoke the plugin manager and load the specified plugins {{ reflist() }}.

```vim
" plugin manager junegunn/vim-plug
call plug#begin('~/.vim/plugged')

" language server protocal
Plug 'prabirshrestha/vim-lsp'
Plug 'mattn/vim-lsp-settings' " Installation helper
Plug 'neovim/nvim-lspconfig'

" text editing
Plug 'lervag/vimtex' " compile and display latex
Plug 'SirVer/ultisnips' " snippet engine
Plug 'tpope/vim-surround' " hotkeys for surrounding tags and braces

call plug#end()
```
{{ nlist() }} Plugin setup using vim-plug.


The first time starting Vim up, run `:PlugInstall` from command mode to download and install the plugins on your machine. Remember to run this command again if you add a new plugin to the list at a later time.

### Syntax checking with vim-lsp

The first time opening a file in .tex format, Vim will prompt you to install the Latex language server using the command `:LspInstallServer`. This is a helper function provided by the [vim-lsp-settings](https://github.com/mattn/vim-lsp-settings) plugin, and requires the vim-lsp plugin to also be present. The plugin is a one-stop-shop for programming languages, and will prompt the user when a new server is available that supports a given file type (confirm [Y] for installation). Most of the features available are for programming, rather than writing *per se*. The tab completion features of [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) are of debatable value here, but it is part of my overall setup.

The real value for Latex editing is from syntax checking, which prints compilation errors to the screen in real time, allowing the user to catch common syntax errors as they occur, rather than at compilation time. In the following example {{figref()}} I have commented out the closing center block, and the language server helpfully informs me I have mismatching environments. This is also useful for catching missing braces, and other common formatting errors.

| ![Syntax Checking \label{fig:1}][fig1] |
|:---:|
| {{ nfig() }} Compiler errors display on screen. |

[fig1]: /fig1.png "Compiler errors display on screen." 

### Compiling and viewing with vimtex

The `vimtex` plugin provides functions for compiling and viewing .tex files. I like to select my own font for cover letters beyond the default serif and standard fonts, but the default compiler did not support the package I used for importing fonts, `fontspec`. The `lualatex` engine combined with the `nvr` compiler works without issue. The [mupdf](https://mupdf.com/) reader supports navigation from the home key row using the same key mappings as Vim, providing a seamless transition between the editing and viewing experience. Specify the viewer and compiler options using the following code {{ reflist() }}.

```vim
" changed from defaults for use with the fontspec package
let g:vimtex_compiler_progname = 'nvr'
let g:vimtex_compiler_engine = 'lualatex'
" pdf viewer of choice
let g:vimtex_view_general_viewer = 'mupdf'
```
{{ nlist() }} Configuration options for the `vimtex` plugin.

The language server is a little overly aggressive with specific warnings, tracked here as an [issue](https://github.com/lervag/vimtex/issues/2024). I have included the fix below {{ reflist() }}, pasted straight from the post to my `init.vim` file.

```vim
" prevents spurious warnings
let g:vimtex_quickfix_ignore_filters = [
      \ 'Underfull',
      \ 'Overfull',
      \]
```
{{ nlist() }} Ignore these warnings related to underfull or overfull text lines.

The `vimtex` plugin makes the functions `:VimtexCompile` and `:VimtexView` available as Vim commands. Typing these commands manually is not exactly ergonomic, so we map these commands to a select set of hotkeys as an added layer of convenience. In my case, I use the space bar followed by the `w` key start the compiler, which will build the pdf and open the viewer. The compiler stays open and updates upon changes, but I can stop and restart it by pressing `<space>+w` repeatedly. I like to quit the viewer when not in active use (using the `q` key in mupdf), and open a fresh instance when I am ready to review changes, using the space bar followed by the `e` key {{ reflist() }}. This prevents multiple instances from accumulating in the workspace, and ensures I do not accidentally review versions that are out-of-date.

```vim
" map vimtex view for rapid pdf display
nnoremap <space>e :VimtexView <CR> 	" spacebar + e opens pdf viewer
nnoremap <space>w :VimtexCompile <CR>	" spacebar + w starts/stops compiler
```
{{ nlist() }} Key bindings for viewing and compiling latex files.

## Keymapping

The following lines are a curated selection from config files I found on the web. The comments above indicate the intended effect.
