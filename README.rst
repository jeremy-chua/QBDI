Introduction
============
.. intro

QuarkslaB Dynamic binary Instrumentation (QBDI) is a modular, cross-platform and cross-architecture 
DBI framework. It aims to support Linux, macOS, Android, iOS and Windows operating systems running on 
x86, x86-64, ARM and AArch64 architectures. Information about what is a DBI framework and how QBDI 
works can be found in the user documentation introduction (:ref:`user-introduction`).

QBDI modularity means it doesn't contain a preferred injection method, rather QBDI is meant to be 
used in conjunction with an external injection tool. QBDI examples include a linux and macOS 
injector for dynamic executables and there are also bindings to use `Frida <https://frida.re>`_ 
as an injector.

x86-64 support is mature however SIMD memory access are not yet reported. ARM support is in 
progress but already sufficient to execute simple CLI program like *ls* or *cat*. x86 and AArch64 
are planned, but currently unsupported.

A current limitation is that QBDI doesn't handle signals, multi-threading and C++ exception 
mechanisms.

.. role:: green
.. role:: yellow
.. role:: orange
.. role:: red

=======   =====================   ======================   =================================
CPU       Operating Systems       Execution                Memory Access Information
=======   =====================   ======================   =================================
x86-64    Linux, OS X, Windows    :green:`Supported`       :yellow:`Partial (only non SIMD)`
ARM       Linux, Android          :yellow:`Partial`        :red:`Unsupported`
AArch64   Linux, Android          :red:`Unsupported`       :red:`Unsupported`
x86       Linux, Windows, OS X    :red:`Unsupported`       :red:`Unsupported`
=======   =====================   ======================   =================================

.. intro-end

Compilation
===========
.. compil

To build this project the following dependencies are needed on your system: cmake, make (for Linux
and macOS), ninja (for Android), Visual Studio (for Windows) and a C++11 toolchain for the platform of
your choice.

The compilation is a two-step process:

* A local binary distribution of the dependencies is built.
* QBDI is built using those binaries.

The current dependencies which need to be built are LLVM and Google Test. This local built of 
LLVM is required because QBDI uses private APIs not exported by regular LLVM installations and 
because our code is only compatible with a specific version of those APIs. This first step is 
cached and only needs to be run once, subsequent builds only need to repeat the second step.

QBDI build system relies on CMake and requires to pass build configuration flags. To help with 
this step we provide shell scripts for common build configurations which follow the naming pattern 
``config-OS-ARCH.sh``. Modifying these scripts is necessary if you want to compile in debug or 
cross compile QBDI.

Linux
-----

x86-64
^^^^^^

Create a new directory at the root of the source tree, and execute the linux configuration script::

    mkdir build
    cd build
    ../cmake/config-linux-X86_64.sh

If the build script warns you of missing dependencies for your platform (in the case of a first 
compilation), or if you want to rebuild them, execute the following commands::

    make llvm
    make gtest

This will rebuild the binary distribution of those dependencies for your platform. You can
then relaunch the configuration script from above and compile::

    ../cmake/config-linux-X86_64.sh
    make -j4

Cross-compiling for ARM
^^^^^^^^^^^^^^^^^^^^^^^

The same step as above can be used but using the ``config-linux-ARM.sh`` configuration script 
instead. This script however needs to be customized for your cross-compilation toolchain:

* The right binaries must be exported in the ``AS``, ``CC``, ``CXX`` and ``STRIP`` environment 
  variables.
* The ``-DCMAKE_C_FLAGS`` and ``-DCMAKE_CXX_FLAGS`` should contain the correct default flags for 
  your toolchain. At least the ``ARM_ARCH``, ``ARM_C_INCLUDE`` and ``ARM_CXX_INCLUDE`` should be 
  modified to match your toolchain but more might be needed.

macOS
-----

Create a new directory at the root of the source tree, and execute the linux configuration script::

    mkdir build
    cd build
    ../cmake/config-macOS-X86_64.sh

If the build script warns you of missing dependencies for your platform (in the case of a first 
compilation), or if you want to rebuild them, execute the following commands::

    make llvm
    make gtest


This will rebuild the binary distribution of those dependencies for your platform. You can
then relaunch the build script from above and compile::

    ../cmake/config-macOS-X86_64.sh
    make -j4

Windows
-------

Building on windows requires a working bash interpreter, both Cygwin and the Windows Subsytem for 
Linux have been used successfully. It also requires a pure Windows installation of Python 2.

First, the ``config-win-X86_64.sh`` should be edited to use the generator (the ``-G`` flag) 
matching your Visual Studio installation. Then the following command should be run::

    mkdir build
    cd build
    ../cmake/config-win-X86_64.sh

If the build script warns you of missing dependencies for your platform (in the case of a first 
compilation), or if you want to rebuild them, execute the following commands::

    MSBuild deps\llvm.vcxproj
    MSBuild deps\gtest.vcxproj

This will rebuild the binary distribution of those dependencies for your platform. You can
then relaunch the build script from above and compile::

    ../cmake/config-win-X86_64.sh
    MSBuild /p:Configuration=Release ALL_BUILD.vcxproj

Android
-------

Cross-compiling for Android requires the Android NDK and has only been tested under Linux. The 
``config-android-ARM.sh`` configuration script should be customized to match your NDK installation 
and target platform:

* ``NDK_PATH`` should point to your Android NDK
* ``SDKBIN_PATH`` should be completed to point to the toolchain to use inside the NDK.
* ``API_LEVEL`` should match the Android API level of your target.
* The right binaries must be exported in the ``AS``, ``CC``, ``CXX`` and ``STRIP`` environment 
  variables (look at what is inside your ``SDKBIN_PATH``).

From that point on the Linux guide can be followed using this configuration script.

.. compil-end