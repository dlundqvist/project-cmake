# project-cmake

Emacs extension to the project package for supporting CMake as build system.

## Description

This package is an extension to Emacs' own `project` package [(see link)](https://www.gnu.org/software/emacs/manual/html_node/emacs/Projects.html) .  Emacs' `project` understands more or less the source coude structure of a repository, it can help you find files and open shells at the right locations. However, its `project-compile` minimalistic function does not really work and does not understand modern build systems.

The package `project-cmake` incorporates the required logic to understand that a project is to be configured, built and tested using CMake and CTest.  It also is capable of recognizing different build kits, on platforms that support it.  For instance, on Windows it can scan for [MSYS2 / MINGW64](https://www.msys2.org/) environments and used the versions of CMake and the compiler supported by it. It can also detect and configure installations of Linux distributions via the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install), using the installed copy of CMake and Linux C/C++ compilers to build the software, and eventually it will also support Microsoft compilation environments.

`project-cmake` also adds new key bindings to the `project-prefix-map`, with the following default assignments

   C-x p C   -  project-cmake-configure
   C-x p m   -  project-cmake-build
   C-x p t   -  project-cmake-test
   C-x p s  -  project-cmake-shell

`project-cmake` can also help LSP servers by providing them with the right configuration flags on how to locate a project's build structure and build flags.  At this moment, this integration is only provided for `eglot` (see [project webage](https://github.com/joaotavora/eglot)), via the function `project-cmake-eglot-integration` which hooks into `eglot-ensure` and updates `eglot-server-programs`.

`project-cmake-shell` is a wrapper around `project-shell` that enables using shells and environments appropriate to the development kit. For instance, if building with MINGW64 under the UCRT64 ABI, it will invoke MSYS's copy of `bash` with appropriate arguments (namely login and interactive shell without the readline library).

## Usage

Let me briefly describe how I use this project. I have checked out this repository at a common location for my libraries, say `~/src/project-cmake`. I then make the project available to Emacs with the help of `use-package`, as follows
````
(use-package project-cmake
    :load-path "~/src/project-cmake/"
    :custom
    ;; I work on Windows and build using MSYS2
    (project-cmake-default-kit 'msys2-ucrt64)
    :config
    (require 'eglot)
    (project-cmake-scan-kits)
    (project-cmake-eglot-integration))
````
For this to work, I have first downloaded [MSYS2](https://www.msys2.org/), installing a rather minimal set of packages which consists of the following ones (and others that my project depends on)
````
mingw64-ucrt-w64-x86_64-gcc
mingw64-ucrt-w64-x86_64-cmake
mingw64-ucrt-w64-x86_64-clang-extra-tools
````

For the `eglot` integration I use a very minimalistic configuration, as `project-cmake` already takes care of configuring the C/C++ language server for me:
````
(use-package eglot
  :ensure t
  :hook
  ((c-mode . eglot-ensure)
   (c++-mode . eglot-ensure))
  )
````

Note that in order for `eglot` to work with [`clangd`](https://clangd.llvm.org/), it may need the `compile_commands.json` generated by CMake at configuration time. When CMake has failed to configure or you did not have yet time to do it, `clangd` will not find that file. In that case you might have to restart the language server by calling `eglot-shutdown`, configuring the project with `project-cmake-configure` and reloading the C/C++ files.
