---
layout: default
title: Duff
---

# Duff

## Welcome

Duff is a Unix command-line utility for quickly finding duplicates in a given
set of files.

Duff is written in C, uses gettext where available, is licensed under the
[zlib/libpng](http://www.opensource.org/licenses/zlib-license.php) license and
should compile on most modern Unices.


## Disambiguation

This is *not* [DUFF](http://dff.sourceforge.net/), the Windows program.  This is
duff, the Unix command-line utility.

If you are using Windows and wish to find duplicate files, use
[DUFF](http://dff.sourceforge.net/).


## Documentation

Duff is documented in its [man page](man.html).


## Download

Duff is currently considered stable.  The latest release of duff is 0.5.2, which
was released on January 29, 2012.

[duff-0.5.2.zip](https://github.com/elmindreda/duff/archive/0.5.2.zip) (source archive)

[duff-0.5.2.tar.gz](https://github.com/elmindreda/duff/archive/0.5.2.tar.gz) (source archive)

See the [GitHub repository](https://github.com/elmindreda/duff) if you are
looking for a development snapshot.


## FAQ

### Hey, that is O(*n*<sup>2</sup>) right?

No, optimally it is O(*n* log *n*) and as of 0.5.2, duff is O(*n* log *n*).


### How does it work, then?

The basic idea (as of version 0.5.2) is:

 - Only compare files if they're of equal size.
 - Compare the beginning of files before calculating digests.
 - Only calculate digests if the beginning matches.
 - Compare digests instead of file contents.
 - Only compare contents if explicitly asked.


### What systems does duff build on?

Duff should build on any modern Unix system with a C89 compiler.  It uses
gettext where available, but will work without it.

The following distributions and package systems already carry duff packages:

 - ALT Linux
 - Arch Linux
 - Cygwin
 - Debian GNU/Linux
 - Fink
 - FreeBSD
 - Gentoo Linux
 - Homebrew
 - MacPorts
 - NetBSD (in WIP)
 - openSUSE
 - Parabola GNU/Linux
 - Slackware Linux
 - Ubuntu

There is also an [Ubuntu PPA for
duff](https://launchpad.net/~kamalmostafa/+archive/duff).

Please [let me know](mailto:public@elmindreda.org) if your system should be listed.


### How do you calculate the checksum?

It is a regular SHA message digest, calculated using (by default) SHA-1, or if
you prefer, SHA-256, SHA-384 or SHA-512.

The actual calculations are done using [sha](http://www.saddi.com/software/sha/)
by Allan Saddi.


### What is it good for?

Getting a list of duplicates in a given set of files, which can then be acted
upon by you or programs other than duff.  Note that duff itself never modifies
any files, but it's designed to play nice with tools that do.


### Is duff named after Tom Duff?

As much as I like [Duff's device](https://en.wikipedia.org/wiki/Duff's_device),
no, it isn't.  Duff stands for DUplicate File Finder.

It's not named after the beer, either.


### Shouldn't duff also do *x*?

I don't know, but you're welcome to write a patch to make it do *x*, or
send me an email explaining why you think duff should do *x*.  If
I like your patch or email, a future version of duff will probably do *x*.
Several people have already had success using this method.


## Examples

### Normal mode

Shows normal output, with a header before each cluster of duplicate files, in
this case using recursive search (with the `-r` flag) into the `images`
directory.

    elmindreda@balthasar~$ duff -r comics
    2 files in cluster 1 (43935 bytes, digest ea1a856854c166ebfc95ff96735ae3d03dd551a2)
    comics/Nemi/n102.png
    comics/Nemi/n58.png
    3 files in cluster 2 (32846 bytes, digest 00c819053a711a2f216a94f2a11a202e5bc604aa)
    comics/Nemi/n386.png
    comics/Nemi/n491.png
    comics/Nemi/n512.png
    2 files in cluster 3 (26596 bytes, digest b26a8fd15102adbb697cfc6d92ae57893afe1393)
    comics/Nemi/n389.png
    comics/Nemi/n465.png
    2 files in cluster 4 (30332 bytes, digest 11ff80677c85005a5ff3e12199c010bfe3dc2608)
    comics/Nemi/n380.png
    comics/Nemi/n451.png

The header can be customized (with the `-f` flag) for example outputing only the
number of files that follow:

    elmindreda@balthasar~$ duff -r -f '%n' comics
    2
    comics/Nemi/n102.png
    comics/Nemi/n58.png
    3
    comics/Nemi/n386.png
    comics/Nemi/n491.png
    comics/Nemi/n512.png
    2
    comics/Nemi/n389.png
    comics/Nemi/n465.png
    2
    comics/Nemi/n380.png
    comics/Nemi/n451.png


### Excess mode

Duff can report all but one file from each cluster of duplicates (with the `-e`
flag).  This can be used in combination with for example `rm` to remove
duplicates, but should *only* be done if you don't care *which* duplicates are
removed.

    elmindreda@balthasar~$ duff -re comics
    comics/Nemi/n58.png
    comics/Nemi/n491.png
    comics/Nemi/n512.png
    comics/Nemi/n465.png
    comics/Nemi/n451.png


### Acting as a filter

Duff can read file and directory names from stdin and so is able to function as
a filter in pipelines.

To handle file names containing whitespace and other special characters, duff
can use null-terminated input and output (with the `-0` flag).  This should be
used whenever duff is used as a filter and the other programs in the pipeline
are able to output and/or parse null-terminated text.

Here, `echo` is used for demonstration purposes, but it can of course be
replaced by another program like `mv` or `rm`.

    elmindreda@balthasar~$ duff -re0 documents | xargs -n1 -0 echo
    documents/Thinking in C++ - Second Edition - Volume 2/code/g++295.mac
    documents/Master Class Assembly Language/CH12/COMMON/GRAPH.H
    documents/Master Class Assembly Language/CH18/DEMO/COMMON/GRAPH.H

File and directory names are read from stdin if none are specified on the
command line.  Here, `find` is used for finding files and duff is used as
a filter on the resulting list of file names.  Also note the use of `xargs` and
`echo` to turn the null characters into newlines: 

    elmindreda@balthasar~$ find . -name '*.pdf' -print0 | duff -0 | xargs -0 -n1 echo
    2 files in cluster 1 (1070259 bytes, digest a0a6806859ce209e7ecd3191b0d3343438801f93)
    documents/A Real-Time Cloud Modeling, Rendering and Animation System.pdf
    documents/A Real-Time Cloud Modeling, Rendering, and Animation System.pdf
    2 files in cluster 2 (32213588 bytes, digest b308ec6165679f210516ea802263964ca8a6b73c)
    documents/Terminators and Iron Men: Image-based lighting and physical shading at ILM.pdf
    documents/sigg2010_physhadcourse_ILM.pdf

This also works for directory names:

    elmindreda@balthasar~$ find projects -name .git -print0 | duff -r0 | xargs -0 -n1 echo
    2 files in cluster 1 (4061 bytes, digest 652f2780792e17f37797a56d56865b51753c8ab6)
    projects/duff-qsort/.git/objects/4d/4a9519eaf88b18fb157dfe5fae59c1c5d005c7
    projects/duff-jc/.git/objects/4d/4a9519eaf88b18fb157dfe5fae59c1c5d005c7
    2 files in cluster 2 (4088 bytes, digest e607f80e94b5b72fb6f444c208985b49b25d1186)
    projects/duff-qsort/.git/objects/89/4e786e16c1d0d94dfc08d6b475270fe1418d6a
    projects/duff-jc/.git/objects/89/4e786e16c1d0d94dfc08d6b475270fe1418d6a
    2 files in cluster 3 (4088 bytes, digest 6c4a7a408b98a4d678cba146d5a33d811a6be677)
    projects/duff-qsort/.git/objects/23/e5f25d0e5f85798dcfb368ecb2f04f59777f61
    projects/duff-jc/.git/objects/23/e5f25d0e5f85798dcfb368ecb2f04f59777f61


## License

Duff is licensed
[zlib/libpng](http://www.opensource.org/licenses/zlib-license.php), which is
similar in spirit to the new BSD license.


## Alternatives

No program is perfect for everyone and duff is no exception.  Here are a few
stable alternatives to duff:

 - [rdfind](http://rdfind.pauldreik.se/) by Paul Dreik
 - [fdupes](http://premium.caribe.net/~adrian2/fdupes.html) by Adrian Lopez


## Author

Duff is developed by [Camilla Berglund](mailto:public@elmindreda.org), with
bugfixes, suggestions, patches and encouragement contributed by various
people.

