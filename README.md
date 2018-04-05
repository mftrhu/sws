# sws - simple website

> A simple static website generator, heavily inspired by sw & co

sws fills a niche that didn't really need to be filled: that of static website generators.  It does so following the footsteps of [sw][sw] and [swx][swx], but adding a simple plugin system for when some more complexity is required.

The base script (`sws`) behaves like `sw`, turning a nested folder structure containing markup files into something that can be viewed by a browser, but it can do more than that: it can be templated using a very simple syntax that allows for easy embedding of shell snippets, and sample plugins to handle blogs/simple wikis are included in the distribution.

It does everything with (mostly/hopefully) POSIX-sh-compliant shell scripts, sed, grep, awk and ed, and less than four hundred lines of code between the core script and plugins. As for the why - I wanted something simple, but not quite as simple as `sw`.

[sw]: https://github.com/jroimartin/sw
[swx]: https://3hg.fr/swx/swx_en.html

## Table of contents

- [Installation](#installation)
- [Usage](#usage)
  - [Metadata syntax](#metadata-syntax)
  - [Configuration](#configuration)
  - [Templates](#templates)
- [Extending sws](#extending-sws)
  - [sws internals](#sws-internals)
    - [Functions](#functions)
    - [Variables](#variables)
  - [Hooks](#hooks)
  - [Sample plugins](#sample-plugins)
    - [`blog`](#blog)
    - [`wiki`](#wiki)
- [License](#license)

## Installation

sws does not need to be installed, as it is just a shell script.  It can be copied/symlinked into a directory somewhere in `$PATH`, if wanted.


## Usage

```shell
$ sws SOURCE DESTINATION
```

After loading `_config`, generating the templates and loading the contents of the `_plugins` directory (if any), sws will go through each file in `SOURCE` and either copy them or convert them to HTML to put them in the `DESTINATION` directory.

This can be automated - especially when dealing with multiple sub-sites with their own plugins - by putting the calls in a Makefile.


### Metadata syntax

sws supports a very simple syntax for metadata, in the style of the [refer][refer] troff preprocessor. Metadata fields are single-line, and begin with a percent sign (`%`) immediately followed by an uppercase letter, a space, and some data.

Conventionally, the following meanings have been assigned to the various letters, and they are used by sws for the same purpose:

- `%T`: title of the work;
- `%A`: author;
- `%D`: date of publication;
- `%K`: keywords/tags, space-separated;
- `%X`: summary.

[refer]: https://en.wikipedia.org/wiki/Refer_%28software%29


### Configuration

sws can be configured by setting environment variables, either by exporting them or by adding them to the `_config` file of the source directory. It uses the following variables:

- `MARKDOWN`: path to/command to convert markdown source into HTML;
- `find_opts_pre` and `find_opts_post`: options to be passed to the `find` command tasked with finding all the files to parse inside the source directory;
- `head_tmpl`, `foot_tmpl` and `plugins_foldr`: paths pointing at the `_header` and `_footer` files, or `_plugins` directory, if they are not contained within the same directory as the `_config` file.


### Templates

sws uses a very simple templating system, processing files with an awk script to turn them into shell functions.

Lines starting with `%` are passed through without any processing, allowing the embedding of shell code for loops and conditionals, while lines that do not are simply echoed out.  Variables can be embedded with the `{{variable}}` syntax, and likewise for subshells (`{{(shell)}}`, but they cannot contain the `}` character).

**Template example:**

```html
<!DOCTYPE html>
<html lang="{{lang}}">
  <head>
    <title>{{site_title}}</title>
% for style in $css; do
    <link rel="stylesheet" type="text/css" href="/css/{{style}}.css">
% done
  </head>
```


## Extending sws

sws will load (source) all the files or symlinks inside the `_plugins` subdirectory of the main directory.  They can modify or extend how sws works, either by overriding its functions or by adding hooks.

### sws internals

#### Functions

- `get_meta FILE FIELD`: given a filename, returns the contents of `FIELD` if present.
- `sws_skip FILE`: given a filename, returns `true` if that file should not be handled by sws (by default this means that `FILE`'s basename starts with an underscore).
- `sws_handle FILE`: given a filename, either copies it to the destination directory or invokes `sws_markup` and `sws_page` to create the corresponding page. It also sets the `name` and `bare` variables.
- `sws_markup FILE`: strips the metadata from `FILE` and either pipes it to a document converter or copies it straight to the output.
- `sws_page`: expects HTML markup on stdin, and invokes the header and footer templates while taking care of setting the `page_title` and `prefix` variables.

#### Variables

- `src`: the source directory.
- `dst`: the destination directory.
- `from`: the path of the current file being handled.
- `name`: the path where the file will be put after being processed.
- `bare`: the basename of the current file/page, minus the extension.
- `prefix`: relative path leading to the root of the site, used for example to link to CSS and other assets.


### Hooks

Hooks are called in FIFO order, and can be added by using the `add_hook` function to the `before`, `during` or `after` phase.

**Syntax:**

```shell
add_hook PHASE FUNCTION
```

`before` and `after` are executed only once per sws call, immediately before and after processing the files in the source directory.  The `during` hook is called, once for every file, *after* copying the file/creating a page from it.


### Sample plugins

A few sample plugins are included in the sws distribution.

#### `blog`

A simple weblog plugin, it handles the creation of a chronological archive, a RSS feed and of an index page containing the last published article and links to the previous ones.

It can be minimally configured by setting the `blog_show_older` variable, which defaults to ten and which is the number of items (besides the very last article) that will be linked from the index page.

#### `wiki`

A *very* simple static wiki compiler, preprocessing markdown sources to handle wikilinks and appending a list of backlinks to each page. An index of all the pages in the wiki is also created under `all.html`.

The syntax for wikilinks is `[[Description|Target]]`, with the description part being optional. Wikilinks pointing at inexistent pages are not linked and instead flagged with a superscript `?`.

---

It overrides `sws_markup`, ignoring HTML files to pipe markdown files into the `wikify` function (a thin wrapper around an Awk script) before getting to the markdown processor.

#### `tags`

Handles the creation of tag pages for a wiki or blog, extracting tag information from the `%K` metadata field, listing pages in reverse chronological order (newer first).


## License

Copyright (c) 2018 mftrhu <mftrhu+github@inventati.org>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
