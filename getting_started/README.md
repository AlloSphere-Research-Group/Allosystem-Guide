# Getting Started

##Supported Systems

AlloSystem can be run and built using the following platforms:

* Linux (GCC)
* OS X (GCC and LLVM)
* Windows (Mingw and MSVC)

##Download
AlloSystem is currently distributed in source form only, so you have to build it yourself. You can download the AlloSystem sources using git with the command:

    git clone https://github.com/AlloSphere-Research-Group/AlloSystem.git
For more information and to download git, visit the [Git homepage](http://git-scm.com/).


##Installation
You might not need to install AlloSystem, as it can be used directly from the sources without installing. The AlloSystem build system provides application building facilities (see section *Application building facilities* below). If you choose to install the AlloSystem libraries, then they will be available as any other system libraries and you will need to use AlloProject or build your own IDE projects (see sections on *AlloProject* and *IDEs* below). To decide whether you want to install or use the application building facilities in AlloSystem, read the rest of the chapter.

You will also need to get some of the dependencies described below in the *Dependencies* section.

###OS X
1. Homebrew: You can get AlloSystem and its dependencies using the [homebrew package manager](http://brew.sh). There is no need to download the sources from git if you use this
method. To install using homebrew run on the terminal:


    $ brew tap AlloSphere-Research-Group/AlloSystem
    $ brew install AlloSystem

2. Command line: You can install the libraries by running in the AlloSystem
sources root directory:


    $ sudo make install

###Linux
You can install the libraries by running in the AlloSystem sources root directory:

    $ sudo make install

###Windows Mingw
You can install the libraries by running in the AlloSystem sources root directory:

    $ sudo make install

###Windows MSVC
Not supported, you must manually copy libraries or use the other methods
described below.

##Dependencies

AlloSystem is a complex package that depends on other libraries for functionality.

To build AlloSystem you will first need the cmake build system. You can download cmake from http://www.cmake.org/, from the system's package manager on Linux systems, or using MacPorts or Homebrew on OS X.

Different parts of AlloSystem have different dependencies:
* Allocore:
 * *Mandatory dependencies*:
   * Pthreads, LibUSB and Udev on Linux system (no mandatory dependencies need to be installed on other systems)
 * *Optional dependencies*:
   * OpenGL
   * GLUT
   * GLEW
   * Freetype
   * Assimp
   * Freeimage
   * APR
   * Portaudio
   * Libsndfile
* Alloutil:
 * Allocore
 * Lua 5.1 or Luajit
* AlloGLV:
 * Allocore
 * GLV

The smallest and simplest AlloSystem can consist of the Allocore library only without any optional dependencies. This system could allow you to create interactive applications, but will lack a some functionality like font rendering, audio, stereographic rendering or GL control widgets.

If you build Allocore without having all the dependencies, be aware that the library will build without issues, but if you try to build code that uses the functions that are not available, because the headers have not been copied, building will fail with a message like:

    fatal error: allocore/graphics/al_Image.hpp: No such file or directory

In this particular case, because freeimage was not available, image and texture functionality is not available.

A script called *install dependencies.sh* is provided within each module
to get its dependencies. The scripts should run on Linux, OS X and Windows,
although they might not work under certain circumstances.

Apart from the library dependencies, some of the examples within these packages have other additional dependencies like Gamma, libsndfile and mysql. If you don't have these, some of the examples may not run.


##Application building facilites
The AlloSystem build system allows building applications directly from the
command line without the need to install the libraries system wide. This method is only supported on the command line. You should use the application building facilities if:
* You are prototyping many small applications
* You are developing AlloSystem
* You want to work on the command line, or
* You need to update AlloSystem from git often

To create and run an application, place your source file anywhere within
the AlloSystem source tree and type (in the root AlloSystem directory):

    ./run.sh path/to/file.cpp

You can also build and run applications comprised of multiple files by placing
them all in a single directory and running:

    ./run.sh path/to/directory

If you need to add compiler flags in your application (e.g. for dependencies
linking), you can create a file called textttflags.txt in the same folder as your
sources containing them.

###Debugging on the commandline
If you need to run your application in the debugger, you can run it like:

    ./debug.sh path/to/file.cpp

This will build debug version of the AlloSystem libraries and of the application, and will run it in the debugger. If the application crashes or is interrupted, the debugger shell will open allowing for inspecting memory and
setting breakpoints. If you need to use a different debugger to the default gdb, edit the *debug.sh* script's variable *DEBUGGER*.

##AlloProject
The AlloProject repository contains a set of scripts that simplifies getting AlloSystem, GLV and Gamma and building applications that use them. You should use AlloProject if:
* You need to tie your code to particular versions of AlloSystem, Gamma
and GLV
* You want to use installed versions of AlloSystem, Gamma and GLV in a
command line setting
* You want to generate Xcode projects for an existing set of sources
* You need to use a different version of AlloSystem than the one installed, or
* You want to create projects that include and build Gamma and GLV in addition to AlloSystem


To create an AlloProject, clone the AlloProject git repository (optionally
give it a name, otherwise it will be called AlloProject):

    git clone https://github.com/AlloSphere-Research-Group/AlloProject.git new_project_name

If you need the AlloSystem, Gamma and GLV sources (if you don’t have
them installed in your system), run:

    ./initproject.sh

AlloProject allows you to build any number of applications within the
project directory, just like AlloSystem (see section 1.4). To build and run
an application comprised of a single source file run:

    ./run.sh path/to/file.cpp

To build and run all the files contained in a directory as a single application,
run:

    ./run.sh path/to/directory

The AlloProject scripts will take care of all the linking and building of
AlloSystem, Gamma and GLV. If you need additional compiler flags, use the
*flags.txt* file as in AlloSystem.

AlloProject can generate an Xcode project for a file or files within a directory by running:

    ./makexcode.sh src/simpleApp.cpp

##Creating your own IDE projects
Although AlloProject (see section on *AlloProject* above) can assist in creating IDE projects, you might want to create your own IDE projects that use AlloSystem. In this case the only requirement is that the AlloSystem library and headers are
installed where your system will find them, and that you add linking options to the desired AlloSystem modules (e.g. liballocore, liballloutil, liballoGLV).

##Reporting Issues

We are interested in hearing what didn't work for you so we can fix it. Please file bug reports (and feature requests) in the [AlloSystem github tracker](https://github.com/AlloSphere-Research-Group/AlloSystem/issues).

