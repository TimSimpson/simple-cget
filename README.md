    cget init --std=c++14
    cget install zdlib

    cget install -f recipes.txt
    cget install ./example1/libmd5
    cget install ./example1/md5app
# simple-cget

I've been trying to learn cget and Paul Fultz CMake workflow for awhile now.
Paul made some examples but I'm going even simpler with these.

## Walk-through ##

Navigate to the root of this directory.

Create your cget directory:

    cget init --std=c++14

Install zdlib, which is a zero dependency library:

    cget install ./zdlib

This will put the library files in `cget/lib` and a bunch of .cmake files in a directory called `cget/lib/cmake/zd`.

To rebuild, use `-U`:

    cget install -U ./zdlib

Now build zdapp, which uses zdlib:

    cget build ./zdapp

To install it, do this:

    cget install ./zdapp
