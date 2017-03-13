# McSema [![Slack Chat](http://empireslacking.herokuapp.com/badge.svg)](https://empireslacking.herokuapp.com/)

McSema lifts x86 and amd64 binaries to LLVM bitcode modules. McSema support both Linux and Windows binaries, and most x86 and amd64 instructions, including integer, FPU, and SSE operations.

McSema is separated into two conceptual parts: control flow recovery and instruction translation. Control flow recovery is performed using the `mcsema-disass` tool, which uses IDA Pro to disassemble a binary file and produces a control flow graph. Instruction translation is performed using the `mcsema-lift` tool, which converts the control flow graph into LLVM bitcode.

## Build status

|       | master |
| ----- | ------ |
| Linux | [![Build Status](https://travis-ci.org/trailofbits/mcsema.svg?branch=master)](https://travis-ci.org/trailofbits/mcsema) |

## Features

* Translates 32- and 64-bit Linux ELF and Windows PE binaries to bitcode, including executables and shared libraries for each platform.
* Supports a large subset of x86 and x86-64 instructions, including most integer, FPU, and SSE operations. Use `mcsema-lift --list-supported --arch x86` to see a complete list.
* Runs on both Windows and Linux, and can translate Linux binaries on Windows and Windows binaries on Linux.
* Output bitcode is compatible with the LLVM 3.8 toolchain.
* Translated bitcode can be analyzed or [recompiled as a new, working executable](docs/McsemaWalkthrough.md) with functionality identical to the original.
* McSema runs on Windows and Linux and has been tested on Windows 7, 10, Ubuntu 14.04, and Ubuntu 16.04.

## Using Mcsema

Why would anyone translate binaries *back* to bitcode?

* **Binary Patching And Modification**. Lifting to LLVM IR lets you cleanly modify the target program. You can run obfuscation or hardening passes, add features, remove features, rewrite features, or even fix that pesky typo, grammatical error, or insane logic. When done, your new creation can be recompiled to a new binary sporting all those changes. In the [Cyber Grand Challenge](https://blog.trailofbits.com/2015/07/15/how-we-fared-in-the-cyber-grand-challenge/), we were able to use mcsema to translate challenge binaries to bitcode, insert memory safety checks, and then re-emit working binaries.

* **Symbolic Execution with KLEE**. [KLEE](https://klee.github.io/) operates on LLVM bitcode, usually generated by providing source to the LLVM toolchain. Mcsema can lift a binary to LLVM bitcode, [permitting KLEE to operate on previously unavailable targets](https://blog.trailofbits.com/2014/12/04/close-encounters-with-symbolic-execution-part-2/).

* **Re-use existing LLVM-based tools**. KLEE is not the only tool that becomes available for use on bitcode. It is possible to run LLVM optimization passes and other LLVM-based tools like [libFuzzer](http://llvm.org/docs/LibFuzzer.html) on [lifted bitcode](docs/McsemaWalkthrough.md).

* **Analyze the binary rather than the source**. Source level analysis is great but not always possible (e.g. you don't have the source) and, even when it is available, it lacks compiler transformations, re-ordering, and optimizations. Analyzing the actual binary guarantees that you're analyzing the true executed behavior.

* **Write one set of analysis tools**. Lifting to LLVM IR means that one set of analysis tools can work on both the source and the binary. Maintaining a single set of tools saves development time and effort, and allows for a single set of better tools.

## Documentation

 - [How to use mcsema: A walkthrough](docs/McsemaWalkthrough.md)
 - [Using Mcsema with libFuzzer](docs/UsingLibFuzzer.md)
 - [Navigating the source code](docs/NavigatingTheCode.md)
 - [Life of an instruction](docs/LifeOfAnInstruction.md)
 - [How to implement the semantics of an instruction](docs/AddAnInstruction.md)
 - [Limitations](docs/Limitations.md)

## Getting help

If you are experiencing problems with McSema, or just want to learn more and contribute, then ask for help in the `#tool-mcsema` channel of the [Empire Hacking Slack](https://empireslacking.herokuapp.com/). Alternatively, you can join our mailing list at [mcsema-dev@googlegroups.com](https://groups.google.com/forum/?hl=en#!forum/mcsema-dev) or email us privately at mcsema@trailofbits.com.

## Dependencies

| Name | Version | 
| ---- | ------- |
| [Git](https://git-scm.com/) | Latest |
| [CMake](https://cmake.org/) | 2.8+ |
| [Google Protobuf](https://github.com/google/protobuf) | 2.6.1 |
| [LLVM](http://llvm.org/) | 3.8 |
| [Clang](http://clang.llvm.org/) | 3.8 |
| [Python](https://www.python.org/) | 2.7 | 
| [Python Package Index](https://pypi.python.org/pypi) | Latest |
| [python-protobuf](https://pypi.python.org/pypi/protobuf) | 2.6.1 |
| [IDA Pro](https://www.hex-rays.com/products/ida) | 6.7+ |

## Getting and building the code

### On Linux

### Step 1: Install dependencies

```shell
sudo apt-get update
sudo apt-get upgrade

sudo apt-get install \
     git \
     cmake \
     libprotoc-dev libprotobuf-dev libprotobuf-dev protobuf-compiler \
     python2.7 python-pip \
     llvm-3.8 clang-3.8 \
     realpath

sudo pip install --upgrade pip
sudo pip install 'protobuf==2.6.1'
```

#### Note: Using IDA on 64 bit Ubuntu

If your IDA install does not use the system's Python, you can add the `protobuf` library manually to IDA's zip of modules.

```
# Python module dir is generally in /usr/lib or /usr/local/lib
touch /path/to/python2.7/dist-packages/google/__init__.py
cd /path/to/lib/python2.7/dist-packages/              
sudo zip -rv /path/to/ida-6.X/python/lib/python27.zip google/
sudo chown your_user:your_user /home/taxicat/ida-6.7/python/lib/python27.zip
```

### Step 2: Clone and enter the repository

```shell
git clone git@github.com:trailofbits/mcsema.git --depth 1
```

The Linux bootstrap script supports two configuration options:

  * `--prefix`: The installation directory prefix for mcsema-lift. Defaults to the directory containing the bootstrap script.
  * `--build`: Set the build type. Defaults to `Debug`.

```shell
cd mcsema
./bootstrap.sh --build Release
```

### Step 3: Build and install the code

```shell
cd build
make
sudo make install
```

## On Windows

### Step 1: Install dependencies

Download and install [Chocolatey](https://chocolatey.org/install). Then, open Powershell in *administrator* mode, and run the following:

```shell
choco install -y --allowemptychecksum git cmake python2 pip wget unzip 7zip
choco install -y microsoft-visual-cpp-build-tools --installargs "/InstallSelectableItems Win81SDK_CppBuildSKUV1;Win10SDK_VisibleV1"
```

### Step 2: Clone and enter the repository

Open the Developer Command Prompt for Visual Studio, and run:

```shell
cd C:\
if not exist git mkdir git
cd git

git clone https://github.com/trailofbits/mcsema.git --depth 1
```

```shell
cd mcsema
bootstrap
```

### Step 3: Build and install the code

```
Help wanted!
```

## Try it Out

If you have a binary, you can get started with the following commands. First, you recover control flow graph information using `mcsema-disass`. For now, this needs to use IDA Pro as the disassembler.

```shell
mcsema-disass --disassembler /path/to/ida/idal64 --arch amd64 --os linux --output /tmp/ls.cfg --binary /bin/ls --entrypoint main
```

Once you have the control flow graph information, you can lift the target binary using `mcsema-lift`.

```shell
mcsema-lift --arch amd64 --os linux --cfg /tmp/ls.cfg --entrypoint main --output /tmp/ls.bc
```

There are a few things that we can do with the lifted bitcode. The usual thing to do is to recompile it back to an executable.
```shell
clang-3.8 -o /tmp/ls_lifted generated/ELF_64_linux.S /tmp/ls.bc -lpthread -ldl -lpcre /lib/x86_64-linux-gnu/libselinux.so.1
```

## FAQ

### How do you pronounce McSema and where did the name come from?

McSema is pronounced 'em see se ma' and is short for machine code semantics.
