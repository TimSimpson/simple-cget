# simple-cget

This is an example project / semi-tutorial on using [Cget](https://github.com/pfultz2/cget). It also is an example on best practices for writing [CMake](https://cmake.org/) files according to [Effective CMake](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiO4an007bXAhXENSYKHeycDGoQtwIIKDAA&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DbsXLMQ6WgIk&usg=AOvVaw0RtjvfdZFL46eWd5uwzsmw) and of course Cget which is written to work with well-behaved CMake projects.

I'd like to thank Paul Fultz II for letting me pick his brain on quite a few issues and delivering great presentations on both CMake and Cget which inspired me to clean up my CMake habits. Paul offers some [great examples of using Cget](https://github.com/pfultz2/cget-examples) however I realized as I was talking to him that my main hang-up was not understanding how Cmake config files and pkg-config worked (and also working in Windows where people mostly ignore it). So I wrote these examples to offer an even simpler introduction to his with explicit comments around some of the material I found exotic.

Disclaimer: much of what I write here should be taken as my own opinion (and with more than a few grains of salt).

## What is CMake? ##

CMake is a great build tool that is becoming the de facto standard of the C++ community. I recommend it often because:

* It's highly portable. It makes it easy to work on the same project on both Windows and Linux boxes using the best compilers and native tooling for each platform.
* It's high quality: once you get a build working, it's easy to keep it working.
* All the cool kids are using it (well, the really cool kids are probably using b2 or Meson but don't worry about that now).

Unfortunately, it's not perfect:

* The syntax and language comes from the Visual Basic / Fortran "long_commands_for_everything" school of design.
* CMake doesn't make it clear when you're not using best practices, particularly when it concerns how to export and reuse libraries from other projects. As a result, it's very easy to create CMake projects that build your code and libraries and seem to work but then don't play well with other people's CMake projects.
* I knew a professor who'd tell students learning to code "if it is long, it is wrong." Unfortunately some CMake best practices result in code filled with essential boilerplate which looks like cruft when you're learning the ropes.

I've tried to really reduce the amount of code and complexity in these examples to make it obvious how CMake is working.

## What is Cget? ##

Cget is what it says: "CMake Package Retrieval".

This brings up a more important question when learning Cget, which is "what is a CMake package anyway?" Googling this will tell you there's more than a few definitions out there. For this repo I've tried to go with Cget and [Effective CMake](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiO4an007bXAhXENSYKHeycDGoQtwIIKDAA&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DbsXLMQ6WgIk&usg=AOvVaw0RtjvfdZFL46eWd5uwzsmw)'s definition.

For now, let's just think of a CMake package as being the header files, library files, and executable binaries associated with a CMake project that we might want to export to be used by other projects.

Cget is a way to download and install these packages easily, as CMake itself offers no way of retrieving external projects (other than "ExternalProject_Add", which IMO should be avoided).

For Python programmers, consider a Python package (yes, I know the two are very different and some people hate when I make this metaphor but comparing the two can be instructive as the workflow I'm about to describe is the same).

When you first install Python on a system, it's pretty easy to start installing Python packages globally, but as most Python devs will tell you this isn't best practice because you can easily dirty your system.

For example, let's say you need to install package A version 1, then package B which depends on A v.1. Next you install package C, which depends on package A version 2- only now you have a problem, because package B can't work with package A version 2.

The solution Python has for this are called virtualenvs, which are basically separate Python distributions, i.e. unique directories containing full copies of the standard library where new packages can be installed to (well, there's typically a lot of symlinks involved to avoid waste but consider that an implementation detail). In the above example, you'd create two virtualenvs, one for package B and one for package C, and both of them would install the version of package A they needed. The virtualenv for B couldn't see package C at all, because it would effectively be a unique installation of Python.

Basically, CMake can work the same way. It's possible to git clone a project, tell CMake to "install" that project and make the resulting CMake package available on the system to all other CMake projects that might reference it.

Of course, this gets us into the same predicament as described above, so a better idea is to install the libraries, headers, and everything else important in a package to unique directories on your machine.

CMake offers a way to do this already in the form of toolchain files, which can set different directories to copy files related to packages as part of CMake's install process. Toolchain files have other powerful features, such as allowing you to specify which compiler to use and the various options that go with it. So if you're writing a project on a Linux machine, you can tell CMake to use one toolchain file and it will compile using gcc, then give it another toolchain file and it will compile the code to Javascript using emscripten.

With toolchain files we can emulate the same kind of isolation provided by tools like virtualenv in CMake. But it's cumbersome, and we'd have to download the source for each package manually.

Thankfully, Cget can easily create new directories for package stuff as well as the toolchain files that tell CMake to use them.

I call these `cget-envs`, which is a name I thought of and isn't endorsed by anyone. Think of it as a way of keeping all your C++ junk separated.


## Walk-through ##

Navigate to the root of this directory.

Create your cget directory:

    cget init --std=c++14

This creates a directory called `cget`. This directory is the `cget-env` , and puts a CMake toolchain there (you can override this default toolchain during initialization if you want).

This metaphor is a stretch, but I'd recommend you think of the cget directory / toolchain combo as equivalent to a Python virtualenv. As long as every call you make to CMake references the toolchain (which cget calls will do automatically) you can build and install software all day long and not worry about your system getting dirty.

This is not an official term but I'm going to call this an `cget-space` (a better name would be environment but that term gets overused).

Next, let's build a library. `zdlib` stands for "zero dependency library" and it's probably the simplest C++ library you can build and install with CMake.

Install it to the cget-space like this:

    cget install ./zdlib

This will put the library files in `cget/lib` and a bunch of .cmake files in a directory called `cget/lib/cmake/zd`.

These .cmake files are generated by the `install` commands in `zdlib/CMakeLists.txt`. This is core CMake functionality which lets other CMake projects find the library using the "find_package" command. Of course, it's possible to reuse CMake projects via other methods, such as integrated builds, but creating config files like this is apparently the best practice advocated by both Cget and [Effective CMake](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiO4an007bXAhXENSYKHeycDGoQtwIIKDAA&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DbsXLMQ6WgIk&usg=AOvVaw0RtjvfdZFL46eWd5uwzsmw).

To rebuild, use `-U`:

    cget install -U ./zdlib

Now build zdapp, which uses zdlib:

    cget build ./zdapp

To install it, do this:

    cget install ./zdapp

You can now run it with:

    ./cget/bin/zdapp
