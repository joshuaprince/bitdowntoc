# BitDownToc

This library / program / website generates Markdown table of contents (TOC).
It supports **Gitlab** and **GitHub** styles, and can generate anchors to comply with **Bitbucket Server**
(and its lack of proper markdown support) and even https://dev.to.
Thanks to small comments (in HTML or liquid tags), it can also detect previously generated TOC,
so you can run the tool each time you change your README without worries.

Note that it supports English, French, and most **Latin languages**, but not Cyrillic or Chinese!

TOC (generated by this tool, of course, using the `github` profile):

<!-- TOC start -->
- [Usage](#usage)
  * [Online](#online)
  * [Locally (jar)](#locally-jar)
- [TOC flavors (profiles)](#toc-flavors-profiles)
- [TOC placement](#toc-placement)
- [About the code](#about-the-code)
  * [Motivation](#motivation)
  * [Kotlin all the way !](#kotlin-all-the-way-)
  * [Build and run](#build-and-run)
<!-- TOC end -->

## Usage

### Online

Go to https://derlin.github.io/bitdowntoc !

### Locally (jar)

The JVM jar can be found in the releases. Download it locally, and use:
```bash
# BitBucket Server
java -jar bitdowntoc-jvm-*.jar readme.md --inplace
# GitLab
java -jar bitdowntoc-jvm-*.jar --no-anchors readme.md --inplace
# GitHub
java -jar bitdowntoc-jvm-*.jar --no-anchors --no-concat-spaces readme.md --inplace
```

The tool will output the transformed file depending on the following options (mutually exclusive):

* default: output to stdout;
* `-i`/`--inplace`: replace input file;
* `-o`/`--output`: output to the specified file.

**IMPORTANT**: do not use bash redirects with the same file (input = output), it won't work as you expect !

If you have a doubt, run the tool with `-h` or `--help`:
```text
Usage: cli [OPTIONS] INPUTFILE

Options:
  --version                        Show version and exit
  --indent-chars TEXT              Characters used for indenting the toc
                                   (default: '-*+')
  --concat-spaces / --no-concat-spaces
                                   Whether to trim heading spaces in generated
                                   links (GitLab style) or not (GitHub style)
                                   (default: true)
  --anchors / --no-anchors         Whether to generate anchors below headings
                                   (BitBucket Server) (default: true)
  --comment-style [HTML|LIQUID]    Language to use for generating comments
                                   around TOC and anchors (default: HTML)
  --trim-toc / --no-trim-toc       Whether to indent TOC based on the
                                   registered headings, or based on the actual
                                   heading levels (default: true)
  --oneshot / --no-oneshot         Whether to add comments so this tool can
                                   regenerate/update the toc and anchors
                                   (false) or not (true). (default: false)
  --max-level INT                  Maximum heading level to include to the toc
                                   (< 1 means no limit). (default: '-1')
  -p, --profile [BITBUCKET|GITHUB|GITLAB|DEVTO]
                                   Load default options for a specific site
  -i, --inplace                    Overwrite input file
  -o, --output-file PATH           Write the output to a file instead of the
                                   console
  -h, --help                       Show this message and exit

Arguments:
  INPUTFILE  Input Markdown File
```

## TOC flavors (profiles)

This tool should work properly for GitLab, GitHub, BitBucket Server, and dev.to.
There is no support for BitBucket.org, (which prefixes anchor links with `#markdown-header-`),
as the latter can autogenerate TOCs using the `[TOC]` annotation.
Simply choose your flavor using the profile option (`-p`/`--profile`).

In case you are interested, here are the relevant option differences:

* *BitBucket Server* → generate anchors
* *GitLab* → concat spaces, do not generate anchors
* *GitHub* → do not concat spaces, do not generate anchors
* *dev.to* → generate anchors, comment style = liquid

Note that if you render your Markdown using another processor, the *BitBucket Server* is the way to go.
As the anchors are manually added to the markdown, the TOC will work as long as `<a>` tags with a `name` parameter
are supported. If you are working with a platform supporting liquid tags (e.g. forem), use *dev.to* instead.

## TOC placement

You can control where the table of content will be inserted by adding the marker (on its own line):
```text
[TOC]
```

Note that headers above the marker will be ignored. The option "*trim toc*" (turned on by default) means that if
you have e.g. only level-2 headers below the marker, the TOC will be indented as if those were level 1 headers.

## About the code

### Motivation

I got the motivation from the lack of existing tools supporting **BitBucket Server**.
As of version 6, the BitBucket processor doesn't insert any ID or name to the HTML headers generated from markdown, meaning there is no way
of targeting a specific header without manually adding an anchor of the form:
```html
<a name="some-heading"></a>
```

I found [this blog](https://rderik.com/blog/generate-table-of-contents-with-anchors-for-markdown-file-vim-plugin/)
mentioning a Vim plugin doing it for you, but this requires Vim (obviously) and the anchors support leaves in a
specific branch that is unlikely to be maintained.

I am quite fond of [GitHub Wiki TOC generator](https://ecotrust-canada.github.io/markdown-toc/)
(you should see the inspiration here), and wanted something similar but more flexible
(not only targeting GitHub and supporting symbols and diacritics).


### Kotlin all the way !

This project was also a great way to play with Kotlin MPP (Multi-Platform Projects).

The code is split between:
* a common module handling the TOC generation, and defining the default options;
* a JVM module with a [Clikt](https://ajalt.github.io/clikt/) CLI;
* a JS module that is imported into a static HTML page.

Tests are implemented in the common module, with one exception: some additional tests are located in the JVM module,
because I wanted to load test files and reading files is not supported in common...

### Build and run

To build the project, use the custom target `bitdowntoc`, which will produce the JVM fat jar and copy the compiled
js scripts into the directory `html/scripts`:
```bash
./gradlew bitdowntoc
```
The `html` folder can then be deployed as a static site.


To run tests, use `allTests`:
```bash
./gradlew allTests
```
