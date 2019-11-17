TL;DR:

Typora requires Pandoc (≥ v1.20). If you do not have Pandoc or only old version of Pandoc installed on your machine, you could:

Download installer from **[Download Page](https://github.com/jgm/pandoc/releases/latest)** and install or update [Pandoc](http://pandoc.org/).

For windows users, if typora still saying it cannot found Pandoc, you may need try to restart your PC.

# Pandoc Integration

In Typora, import function and export function for some file formats (including docx, odt, rtf, epub, LaTeX and wiki) are powered by a 3rd party software named Pandoc. Those features requires Pandoc (≥ v1.16) to be installed.

Please note that install Pandoc is optional for Typora, if you do not need those advanced import/export support in typora, then you do not have to install Pandoc on your computer.

This document would show how to install Pandoc and use Typora with Pandoc for full import/export functions.

[TOC]

## What is Pandoc

[Pandoc](http://pandoc.org/) is a universal document text converter. Typora use it to support file import/export features for several file types. 

## Install Pandoc

### For Mac User

Briefly speaking, there are two recommended ways.

##### Install from downloaded package installer

Download a package installer from pandoc's [download page](https://github.com/jgm/pandoc/releases/latest), open it and follow the instructions for installation.

![Install pandoc on OS X](img/Snip20160502_1.png)

#####  Install from homebrew

For developers using [homebrew](http://brew.sh/), installing pandoc can be one line from terminal:

```sh
brew install pandoc
```

### For Windows User

Download the `pandoc-*-window.msi` from pandoc's [download page](https://github.com/jgm/pandoc/releases/latest), open it and follow the instructions for installation.

![Install pandoc on Windows](img/pandoc-win.PNG)

## Use Pandoc

After Pandoc is installed, then you could import supported file types by clicking File -> Import from menubar, or simple drag and drop a file into typora. Export function is also fully functional from menubar. Pandoc will run in backgrounds for those tasks and then exit automatically, so you may not feel it.

## FAQ

#### Which version of pandoc is supported ?

Versions ≥ 1.16 is required. The latest version, the better. So updating pandoc is encouraged if your pandoc version is too old.

 #### Can typora work without pandoc ?

Yes, only import and export (other than html/PDF file types) needs it.

#### Which file types can be imported or exported by typora ?

Import supports file with extesion: .docx, .latex, .tex, .ltx, .rst, .rest, .org, .wiki, .dokuwiki, .textile, .opml, .epub.

Export supports file formats of: HTML, PDF (these two does not need Pandoc), Docx, odt, rtf, Epub, LaTeX, Media Wiki.

Pandoc should support more file types which typora did not integrate, detailed info is [here](http://pandoc.org/).

#### What's the difference between exporting by typora and exporting by using pandoc from command line ?

Exporting by typora is also powered by Pandoc, yet typora will not convert directly from markdown to target file type, instead, it converts to an inner format pandoc can read and then write as target file type, so, in detail:

- If you run pandoc from command line, then you need to specify its markdown parser from (pandoc Markdown, [CommonMark](http://commonmark.org/), [PHP Markdown Extra](https://michelf.ca/projects/php-markdown/extra/), [GitHub-Flavored Markdown](https://help.github.com/articles/github-flavored-markdown/)), while exporting using typora, typora will pass its inner AST to pandoc for file conversion. In other words, the format of blocks or inline elements in exported output will always be consistent as what you see in typora and exported HTML/PDF. Yet, the styling maybe different.
- Some markdown syntax invented by pandoc Markdown, like citations, are not supported when you exporting from typora, since only markdown syntax typora support will be correctly exported. But we may support more extended markdown syntax in future.
- If you use typora for exporting, then`[TOC]` will be correctly exported for all file types. Highlight and underline will be supported for LaTeX, rtf, Epub, wiki formats and sometimes Docx. Yet, they are only supported for HTML based file formats in raw Pandoc. Other block and inline elements is basically both supported by raw pandoc and typora+pandoc.

#### Can all block/inline element types be exported correctly ?

Task list is not supported yet. Underline and highlight for `.docx` is supported only if they are not used inside or outside other inline styles. Underline and highlight is not support for OpenOffice(`.odt`). Embedded .gif file is not support for LaTeX. Other block or inline elements can basically be exported. But the styles cannot be 100% match when import/export.

#### How to uninstall pandoc for mac ?

This is the instruction from [pandoc's official site](http://pandoc.org/installing.html):

> If you later want to uninstall the package, you can do so by downloading [this script](https://raw.githubusercontent.com/jgm/pandoc/master/osx/uninstall-pandoc.pl) and running it with `perl uninstall-pandoc.pl`".

#### Found bug and unsupported syntax for exporting ?

Contact us <hi@typora.io>, better to provide a sample `.md` file, so we could reproduce the bug. 

If you found it is a bug/feature request for pandoc, you could contact the community via [pandoc-discuss](https://groups.google.com/forum/#!forum/pandoc-discuss).

