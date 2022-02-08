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

Table of Contents:

* [Editor](#editor)
  * [- Plugins](#plugins)
    * [-* Syntax checking with vim-lsp](#lsp) 
    * [-* Compiling and viewing with vimtex](#vimtex) 
    * [-* Using code snippets with `ultisnips`](#snippets)
* [Key Mapping](#keymaps)


# Editor {#editor}

Previously I used [RStudio](https://www.rstudio.com/) for R and [IntelliJ](https://www.jetbrains.com/idea/) for everything else code related. These programs are feature rich, but bloated and heavy, consuming significant amounts of RAM. Latex editors that I tried, including [Scribus](https://www.scribus.net/), are light-weight but not feature-rich enough. [NeoVim](https://neovim.io/) is a current iteration of the storied Vim text editor, modernized to include many of the features of IntelliJ, VSCode and other IDEs, while remaining light-weight, responsive and powerful. Vim works very differently from other text editors, and this can be a good thing, but the lack of common ground with prior experience can slow down the learning process, especially at first.  I recommend the article [Learn Vim for the Last Time](https://danielmiessler.com/study/vim/) by Daniel Miessler, which makes the case for why to use Vim better than I can.

Instead of a settings menu, Vim uses a configuration file, allowing maximum flexibility in customization. But if you are not used to mucking around in configuration files, getting your settings just right will take trial and error. On my system with Neovim, the location in is ~/.config/nvim/init.vim. Learning to use the default configuration is helpful when you are working on new or random machines, but selective customization is key to an ergonomic and productive flow.

## Plugins {#plugins}

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
Plug 'honza/vim-snippets' " more snippets
Plug 'tpope/vim-surround' " hotkeys for surrounding tags and braces

call plug#end()
```
{{ nlist() }} Plugin setup using vim-plug.


The first time starting Vim up, run `:PlugInstall` from command mode to download and install the plugins on your machine. Remember to run this command again if you add a new plugin to the list at a later time.

### Syntax checking with vim-lsp {#lsp}

The first time opening a file in .tex format, Vim will prompt you to install the Latex language server using the command `:LspInstallServer`. This is a helper function provided by the [vim-lsp-settings](https://github.com/mattn/vim-lsp-settings) plugin, and requires the vim-lsp plugin to also be present. The plugin is a one-stop-shop for programming languages, and will prompt the user when a new server is available that supports a given file type (confirm [Y] for installation). Most of the features available are for programming, rather than writing *per se*. The tab completion features of [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) are of debatable value here, but it is part of my overall setup.

The real value for Latex editing is from syntax checking, which prints compilation errors to the screen in real time, allowing the user to catch common syntax errors as they occur, rather than at compilation time. In the following example {{figref()}} I have commented out the closing center block, and the language server helpfully informs me I have mismatching environments. This is also useful for catching missing braces, and other common formatting errors.

| ![Syntax Checking \label{fig:1}][fig1] |
|:---:|
| {{ nfig() }} Compiler errors display on screen. |

[fig1]: /lsp_syntax_check.png "Compiler errors display on screen." 

### Compiling and viewing with vimtex {#vimtex}

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

### Using code snippets with `ultisnips` {#snippets}

While working on my thesis, I used the [Lyx](https://www.lyx.org/) editor. I appreciated that the dropdown menus on Lyx listed an alternative hotkey for each command, and I was quickly able to transition to hotkeys for producing most of the formatting blocks for figures, tables, etc. The breaking point came for me when I updated my Lyx edition to find the hotkeys still functional, but no longer listed on the dropdown menu. With no clear pathway to re-enabling this feature in the settings, the relevant hotkeys slowly began to fade from my memory. One day I began to wonder if I could not use the same editor for Latex as I did for other programming languages, and found NeoVim admirably suited to the task. The problem Latex creates is a sea of boilerplate code related to labels, sections, figures, tables, and math equations, with the actual content sandwiched in-between these formatting blocks. Using a snippet engine enables me to produce these repetitive blocks of code by using the tab key to expand short key sequences into larger code blocks.

Both `ultisnips` and `vim-snippets` include snippet libraries that cover the majority of common use cases for Latex. The best way to learn what is available is to peruse the list of defined snippets, which for `ultisnips` is in `/usr/share/vim/vimfiles/Ultisnips/tex.snippets` on my linux machine. While editing a latex document in insert mode, pressing the tab key after any of the short codes specified by the word `snippet` will expand to the code below up until the line `endsnippet`. For example, lines 53--56 of `Ultisnips/tex.snippets` define a snippet called `abs` that expands to the beginning and ending code blocks of an abstract environment {{ figref() }}. The `$0` is a special variable that places the user's cursor inside of the code block upon generation, allowing you to immediately begin filling the block with content, either manually or by pasting from the clipboard. 

| ![Abstract Snippet][snip_abs] |
|:---:|
| {{ nfig() }} Snippet triggered by `abs+<tab>`. |

[snip_abs]: /snip_abs.png "Abstract Snippet" 

While in insert mode, I type `abs` followed by the tab key:

![Abstract Snippet Before](/snip_abs_before.png)

This results in the following expansion, with my cursor centered within the block ready to enter content:

![Abstract Snippet After](/snip_abs_after.png)

The keywords `enum`, `item` and `desc` similarly create the opening and closing blocks for the enumerate, itemize and description environments, respectively. These lists default to having a single `\item`, and the keyword `it` expands to add an additional `\item`. For code blocks that include bracketed optional arguments, pressing the delete key will remove the optional parameters. For instance, the `pac` keyword expands to the line:

![Package Snippet Before](/snip_pac1.png)

The first cursor position snaps to the inside of the brackets for the optional package argument. Here the delete key will remove the brackets as well as the text. Escaping moves the cursor to the next user-defined parameter, in this case the package name (which I have changed to `fontspec`), and escaping again advances the cursor to immediately after the closing brace:

![Package Snippet After](/snip_pac2.png)

The `fig` keyword is one of the more useful snippets in my workflow, because it includes a sensible level of default formatting, including centering, a label, and image width capped to 80% of page width. Here I have edited the default image name to `mypic.png` to illustrate that the figure caption and label both mirror the selected image name, with the start of the caption capitalized. This is not a useful feature to me for captions, but I use it regularly to set label names based on image file names:

![Figure Snippet](/snip_fig.png)

Note that when there is user-defined text in snippet code blocks, the cursor moves through a predefined sequence of fields to receive input from the user. At this time, Vim in is insert mode, and anything you type will appear on the keyboard verbatim. This can be a surprise if you start trying to navigate using the home row and create a figure named `jjjll`. For options, a sensible default appears (e.g. `width=0.8\linewidth`) whereas field names tend to be descriptive (e.g. `\label{fig:name}`. If you type something, this will overwrite the default value. Escaping advances the cursor, either committing your changes or preserving the default as-is. Vim returns to command mode only after escaping through all the user-generated fields in the invoked snippet.

The `cha`, `sec`, `sub`, `ssub` and `par` keywords expand to chapter, section, subjection, sub-subsection and paragraph blocks respectively, starting with the cursor positioned inside the title braces. Variants ending in asterisks (e.g. `sec*`) expand to unnumbered blocks. Although the keyword `tab` will generate an empty tabular environment, I prefer to the use the `gentbl` keyword, followed by the dimensions of the table in format `WIDTHxHEIGHT`. For producing a table with three rows and columns, I would enter the following text in insert mode, followed by the tab key to trigger snippet expansion:

![Table Generator Snippet Trigger](/snip_gentbl.png)

This expands to the following code, with cursor points at each blank field:

![Table Generator Snippet Result](/snip_gentbl2.png)




## Key Maps {#keymaps}

The following lines are a curated selection from config files I found on the web. The comments above indicate the intended effect.
