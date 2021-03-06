Heapusage
=========
Heapusage is a light-weight tool for finding memory leaks in Linux and macOS applications, and
more generally checking heap usage. By providing a small but important subset of Valgrind's memcheck
functionality it can be a useful alternative to Valgrind for debugging memory leaks in certain
environments (such as large applications which cannot be run at Valgrind slowdown speed, embedded
systems on CPU architectures not supported by Valgrind).

Supported Platforms
===================
Heapusage is primarily developed and tested on Linux, but basic
functionality should work in macOS / OS X as well. Current version has been tested on:
- OS X El Capitan 10.11
- Ubuntu 16.04 LTS
- (Raspbian / Raspberry Pi 3 - based on fix for [issue #1](https://github.com/d99kris/heapusage/issues/1))

Limitation: On macOS / OS X this tool relies on code injection using DYLD_INSERT_LIBRARIES,
which generally does not work with third-party applications in a standard system. Using it on
(your own) applications built from source should work fine though.

Installation
============
Pre-requisites (Ubuntu):

    sudo apt install git cmake build-essential

Download the source code:

    git clone https://github.com/d99kris/heapusage && cd heapusage

Generate Makefile and build:

    mkdir -p build && cd build && cmake .. && make -s

Optionally install in system:

    sudo make install

Usage
=====

General usage syntax:

    heapusage [-m minsize] [-n] [-o path] PROG [ARGS..]
    heapusage --help
    heapusage --version

Options:

    -m <minsize>
           minimum leak in bytes for detailed reporting

    -n     no symbol lookup (faster)

    -o <path>
           write output to specified file path, instead of stderr

    PROG   program to run and analyze

    [ARGS] optional arguments to the program

    --help display this help and exit

    --version
           output version information and exit

Example checking heap usage of test program 'hutest01' with heapusage installed on system:

    heapusage ./hutest01

Example checking heap usage of test program 'hutest01' without heapusage being installed on system and
saving the output to a log file:

    ./heapusage -o /tmp/heapusage.log ./hutest01

Advanced Usage (Library)
========================

For more detailed controls of heap usage logging, one may manually load and setup the
underlying libheapusage library as described in this section.

Linux
-----

General usage syntax:

    HU_FILE=<path> [HU_NOSYMS=0|1] LD_PRELOAD="/path/to/libheapusage.so" <application>

Example using libheapusage (without system install) with gedit logging to file heaplog.txt:

    HU_FILE="heaplog.txt" LD_PRELOAD="./libheapusage.so" gedit

macOS / OS X
------------

General usage syntax:

    HU_FILE=<path> [HU_NOSYMS=0|1] DYLD_INSERT_LIBRARIES="/path/to/libheapusage.1.dylib" DYLD_FORCE_FLAT_NAMESPACE=1 <application>

Example using libheapusage (without system install) with hutest01 application:

    HU_FILE="heaplog.txt" DYLD_INSERT_LIBRARIES="./libheapusage.dylib" DYLD_FORCE_FLAT_NAMESPACE=1 ./hutest01

Output
======
Example output:

    $ ./heapusage ./hutest01
    ==22648== Heapusage - https://github.com/d99kris/heapusage
    ==22648== 
    ==22648== HEAP SUMMARY:
    ==22648==     in use at exit: 12221 bytes in 4 blocks
    ==22648==   total heap usage: 5 allocs, 1 frees, 13332 bytes allocated
    ==22648== 
    ==22648== 6666 bytes in 3 block(s) are lost, originally allocated at:
    ==22648==    at 0x00007fdca672199d: malloc + 49
    ==22648==    at 0x000000000040080d: main + 55
    ==22648==    at 0x00007fdca6376830: __libc_start_main + 240
    ==22648==    at 0x0000000000400709: _start + 41
    ==22648== 
    ==22648== 5555 bytes in 1 block(s) are lost, originally allocated at:
    ==22648==    at 0x00007fdca672199d: malloc + 49
    ==22648==    at 0x00000000004007e8: main + 18
    ==22648==    at 0x00007fdca6376830: __libc_start_main + 240
    ==22648==    at 0x0000000000400709: _start + 41
    ==22648== 
    ==22648== LEAK SUMMARY:
    ==22648==    definitely lost: 12221 bytes in 4 blocks
    ==22648== 

The corresponding file and line number of the stacktrace addresses can be determined
using addr2line on Linux (the equivalent tool for macOS is atos):

    $ addr2line -f -e ./hutest01 0x000000000040080d
    main
    tests/hutest01.c:29

Technical Details
=================
Heapusage intercepts calls to malloc/free/etc and logs each memory allocation and free. At
process termination it outputs logging of all allocations not free'd.

Alternatives
============
There are many memory leak checkers available for Linux and macOS, here are some of them:
- LeakSanitizer
- Mtrace
- Valgrind

License
=======
Heapusage is distributed under the BSD 3-Clause license. See LICENSE file.

Keywords
========
linux, macos, os x, heap usage, finding memory leaks, alternative to valgrind.

