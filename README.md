Heroku buildpack: TeX
=====================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks)
for working with TeX documents. It bundles a working TeX Live environment into your Heroku app and doesn't do anything else with it. Additionally, this buildpack will move latex packages in the bin/tex_dependencies subfolders into locations where they can be detected by latex.


    $ ls

    $ heroku create --buildpack git://github.com/holiture/heroku-buildpack-tex.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom build pack... done
    -----> TeX app detected
    -----> Fetching TeX Live 20120511
    ...

As this currently does not auto-detect new fonts that have been added to the bin/tex_dependencies subfolders, the bin/compile shell script will have to be updated.
For example, this script adds the [hfbright font package](http://www.ctan.org/tex-archive/fonts/ps-type1/hfbright). The compile script then adds the command:

    updmap-sys --enable Map=hfbright.map

This can be useful if you simply want to play around with TeX Live without
having to build or install it yourself. You can pull it up on your instance
easily in bash:

    $ heroku run bash

Multipacks
----------

More likely, you'll want to use it as part of a larger project, which needs to
build PDFs. The easiest way to do this is with a [multipack](https://github.com/ddollar/heroku-buildpack-multi),
where this is just one of the buildpacks you'll be working with.

    $ cat .buildpacks
    git://github.com/heroku/heroku-buildpack-python.git
    git://github.com/holiture/heroku-buildpack-tex.git

    $ heroku config:add BUILDPACK_URL=git://github.com/ddollar/heroku-buildpack-multi.git

This will bundle TeX Live into your instance without impacting your existing
system. You can then call out to executables like `pdflatex` as you would on
any other machine.

Auto-build
----------

Another potential use is simply for building a specific document. This can be
useful if you're working with a document pipeline that outputs LaTeX documents,
but not often enough to need it integrated into a larger system. Rather than
having to build and install TeX Live yourself, you can use this buildpack to
do it for you.

This funcionality is provided by a special `autobuild` branch. To activate it,
simply append `#autobuild` to the buildpack URL and make sure you have a
`document.tex` at the root of your repository. It can reference other .tex files
as necessary, as long as the main file is called `document.tex`.

    $ ls
    document.tex

    $ heroku create --buildpack git://github.com/holiture/heroku-buildpack-tex.git#autobuild

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom build pack... done
    -----> TeX app detected
    -----> Fetching TeX Live 20120511
    -----> Building document.tex
           Wrote 3 pages to document.pdf

This will output a PDF file, which you can download it using your browser as
`document.pdf` at the root of your app's URL. For example, if your document app
is called `silent-night-1234`, you'd be able to find your completed document at
`http://silent-night-1234.herokuapp.com/document.pdf`.
