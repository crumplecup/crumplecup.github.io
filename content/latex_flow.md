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

Table of Contents:

* [Editor of Choice](#editor)
  * [- Plugin Management](#plugins)
    * [-* Syntax Checking](#lsp) 
    * [-* Compiling and Viewing](#vimtex) 
    * [-* Using Snippets](#snippets)
* [Key Mapping](#keymaps)


# Editor of Choice {#editor}

Previously I used [RStudio](https://www.rstudio.com/) for R and [IntelliJ](https://www.jetbrains.com/idea/) for everything else code related. These programs are feature rich, but bloated and heavy, consuming significant amounts of RAM. Latex editors that I tried, including [Scribus](https://www.scribus.net/), are light-weight but not feature-rich enough. [NeoVim](https://neovim.io/) is a current iteration of the storied Vim text editor, modernized to include many of the features of IntelliJ, VSCode and other IDEs, while remaining light-weight, responsive and powerful. Vim works very differently from other text editors, and this can be a good thing, but the lack of common ground with prior experience can slow down the learning process, especially at first.  I recommend the article [Learn Vim for the Last Time](https://danielmiessler.com/study/vim/) by Daniel Miessler, which makes the case for why to use Vim better than I can.

Instead of a settings menu, Vim uses a configuration file, allowing maximum flexibility in customization. But if you are not used to mucking around in configuration files, getting your settings just right will take trial and error. On my system with Neovim, the location in is ~/.config/nvim/init.vim. Learning to use the default configuration is helpful when you are working on new or random machines, but selective customization is key to an ergonomic and productive flow.

## Plugin Management {#plugins}

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

### Syntax Checking {#lsp}

The first time opening a file in .tex format, Vim will prompt you to install the Latex language server using the command `:LspInstallServer`. This is a helper function provided by the [vim-lsp-settings](https://github.com/mattn/vim-lsp-settings) plugin, and requires the vim-lsp plugin to also be present. The plugin is a one-stop-shop for programming languages, and will prompt the user when a new server is available that supports a given file type (confirm [Y] for installation). Most of the features available are for programming, rather than writing *per se*. The tab completion features of [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) are of debatable value here, but it is part of my overall setup.

The real value for Latex editing is from syntax checking, which prints compilation errors to the screen in real time, allowing the user to catch common syntax errors as they occur, rather than at compilation time. In the following example {{figref()}} I have commented out the closing center block, and the language server helpfully informs me I have mismatching environments. This is also useful for catching missing braces, and other common formatting errors.

| ![Syntax Checking \label{fig:1}][fig1] |
|:---:|
| {{ nfig() }} Compiler errors display on screen. |

[fig1]: /lsp_syntax_check.png "Compiler errors display on screen." 

### Compiling and Viewing {#vimtex}

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

### Using Snippets {#snippets}

While working on my thesis, I used the [Lyx](https://www.lyx.org/) editor. I appreciated that the dropdown menus on Lyx listed an alternative hotkey for each command, and I was quickly able to transition to hotkeys for producing most of the formatting blocks for figures, tables, etc. The breaking point came for me when I updated my Lyx edition to find the hotkeys still functional, but no longer listed on the dropdown menu. With no clear pathway to re-enabling this feature in the settings, the relevant hotkeys slowly began to fade from my memory. One day I began to wonder if I could not use the same editor for Latex as I did for other programming languages, and found NeoVim admirably suited to the task. The problem Latex creates is a sea of boilerplate code related to labels, sections, figures, tables, and math equations, with the actual content sandwiched in-between these formatting blocks. Using a snippet engine enables me to produce these repetitive blocks of code by using the tab key to expand short key sequences into larger code blocks.

Both `ultisnips` and `vim-snippets` include snippet libraries that cover the majority of common use cases for Latex. The best way to learn what is available is to peruse the list of defined snippets, which for `ultisnips` is in `/usr/share/vim/vimfiles/Ultisnips/tex.snippets`, and for `vim-snippets` is in `/usr/share/vim/vimfiles/snippets/tex.snippets` on my linux machine. While editing a latex document in insert mode, pressing the tab key after any of the short codes specified by the word `snippet` will expand to the code below up until the line `endsnippet`. For example, lines 53-56 of `Ultisnips/tex.snippets` define a snippet called `abs` that expands to the beginning and ending code blocks of an abstract environment {{ figref() }}. The `$0` is a special variable that places the user's cursor inside of the code block upon generation, allowing you to immediately begin filling the block with content, either manually or by pasting from the clipboard. 

| ![Abstract Snippet][snip_abs] |
|:---:|
| {{ nfig() }} Snippet triggered by `abs+<tab>`. |

[snip_abs]: /snip_abs.png "Abstract Snippet" 
I use the following lines to set snippet expansion to the tab key in my `init.vim`. For snippets with multiple user-defined fields, the `<ctrl>-j` combination moves the cursor forward by one field and `<ctrl>-k` moves it back by one.

```vim
let g:UltiSnipsExpandTrigger="<tab>"	% tab triggers expansion
let g:UltiSnipsJumpForwardTrigger="<c-j>" % move cursor foward one position
let g:UltiSnipsJumpBackwardTrigger="<c-k>" % move cursor back one position
```


While in insert mode, I type `abs` followed by the tab key:

![Abstract Snippet Before](/snip_abs_before.png)

This results in the following expansion, with my cursor centered within the block ready to enter content:

![Abstract Snippet After](/snip_abs_after.png)


The keywords `enum`, `item` and `desc` similarly create the opening and closing blocks for the enumerate, itemize and description environments, respectively. These lists default to having a single `\item`, and the keyword `it` expands to add an additional `\item`. For code blocks that include bracketed optional arguments, pressing the delete key will remove the optional parameters. For instance, the `pac` keyword expands to the line:

![Package Snippet Before](/snip_pac1.png)

The first cursor position snaps to the inside of the brackets for the optional package argument. Here the delete key will remove the brackets as well as the text. The forward trigger (`<ctrl>+j`) moves the cursor to the next user-defined parameter, in this case the package name (which I have changed to `fontspec`), and then advances the cursor to immediately after the closing brace:

![Package Snippet After](/snip_pac2.png)

The `fig` keyword is one of the more useful snippets in my workflow, because it includes a sensible level of default formatting, including centering, a label, and image width capped to 80% of page width. Here I have edited the default image name to `mypic.png` to illustrate that the figure caption and label both mirror the selected image name, with the start of the caption capitalized. This is not a useful feature to me for captions, but I use it regularly to set label names based on image file names:

![Figure Snippet](/snip_fig.png)

When there is user-defined text in snippet code blocks, the cursor moves through a predefined sequence of fields to receive input from the user. At this time, Vim in is insert mode, and anything you type will appear on the keyboard verbatim. For options, a sensible default appears (e.g. `width=0.8\linewidth`) whereas field names tend to be descriptive (e.g. `\label{fig:name}`. If you type something, this will overwrite the default value. Advancing the cursor either commits your changes or preserves the default as-is. Vim returns to command mode only after traversing through all the user-generated fields in the invoked snippet. If you want to amend the default in a field, you can press the tab key to keep the default and enter command mode, where you can edit it, but this does not cancel the rest of the generation process, and the forward trigger will still move the cursor to the next field in the call order if additional cursor points remain. Use the escape key if you want to abort midway through.

The `cha`, `sec`, `sub`, `ssub` and `par` keywords expand to chapter, section, subsection, sub-subsection and paragraph blocks respectively, starting with the cursor positioned inside the title braces. Variants ending in asterisks (e.g. `sec*`) expand to unnumbered blocks. Although the keyword `tab` will generate an empty tabular environment, I prefer to the use the `gentbl` keyword, followed by the dimensions of the table in format `WIDTHxHEIGHT`. For producing a table with three rows and columns, I would enter the following text in insert mode, followed by the tab key to trigger snippet expansion:

![Table Generator Snippet Trigger](/snip_gentbl.png)

This expands to the following code, with cursor points at each blank field:

![Table Generator Snippet Result](/snip_gentbl2.png)

When typing freely, I tend to forget the forward slashes at the end of rows, so starting with a syntactically correct table is helpful for me. Similarly, the characters `tr` followed by a number will expand to a table row with the corresponding number of fields.

The keywords `eq`, `eql` and `eq*` expand to a numbered equation block, without and with a label, and an unnumbered equation block respectively. The `frac` keyword expands to `\frac{}{}`, with the added benefit of moving the cursor from the numerator to the denominator on forward trigger, and `sum` works similarly. The characters `ls` expand to `\left{} \right{}`, and for the delimiters `(`, `|`, `{` and `[` the pattern `ls*` expands to `\left* \right*`, using the appropriate closing delimiter, and placing the cursor in the center of the block.

While the shorter `fig` and `tab` keywords expand to the block environments, using the complete words `figure` and `table` will expand to references including the appropriate leading word, so:

```tex
figure
```
becomes:
```tex
Figure~\ref{fig:}
```
Similarly, the `cite` keyword expands to a cite block with reference fields for a `.bib` file, and `url` expands to `\url{}`. Here the purpose is less to reduce keystrokes than the use of the less ergonomic `\` `{` and `}` keys, and the occurrence of dangling delimiters or missing end blocks. Along this vein, the lone character `b` followed by the tab key expands to:

```tex
\begin{something}

\end{something}
```
The first insert will overwrite the word "something" in the begin and end braces, and the next will place content inside the block. These snippets cover the common use cases for my latex workflow, but they are only a selection.

The only use case I have found for writing my own snippets is to generate templates for frequently-used document formats like cover letters. Typing the word `memo` followed by tab will expand to a cover letter with my name, the current date, and a link to an image of my signature. Compared to saving a blank letter as a .tex document, and copying and saving a new version for each letter, storing the template as a snippet is easier to use, and I do not risk overwriting my blank copy accidentally. I recommend checking out [jdhao's blog](https://jdhao.github.io/2019/04/17/neovim_snippet_s1/) if you are interested.

On my system, an interesting hiccup occurs when installing `vim-snippets`, where invoking a snippet triggers a duplicate resolution menu. For example, pressing tab after `eq` provokes the response:

![Duplicate Snippet Menu](/vim_snippets_dupe.png)

The reason this occurred is that the snippet file was copied into two places on my system, once in the home directory under `.vim/plugged/vim-snippets/snippets` and again in the general file system in `/usr/share/vimfiles/snippets`. If you run into a similar issue with duplicate snippets, you can change the name of the file in your home directory to a name other than `tex.snippets`, which should prevent it from associating with .tex files. For instance, on my machine I use the following line of code to the resolve the issue:

```fish
cd ~/.vim/plugged/vim-snippets/snippets
mv tex.snippets tex.snippets.bk
```

### Delimiter Management

The [vim-surround](https://github.com/tpope/vim-surround) plugin offers extended control over features surrounding words and sentences. While website authors working with html and css are the target audience, I often find that I want to surround a word with quotes, backticks, brackets or braces *after the fact*, and the `vim-surround` plugin allows me to wrap a word using a few keystrokes instead of fiddling around with the arrow keys or the mouse. It also enables changing surrounding delimiters, for example from quotes to braces, especially useful for modifying longer sentences or even paragraphs. I use the default key mapping as described on their web page.


## Key Maps {#keymaps}

The following lines are a curated selection from config files I found on the web. The comments above indicate the intended effect.
