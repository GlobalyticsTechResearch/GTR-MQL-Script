# mql-zmq

ZMQ binding for the MQL language (both 32bit MT4 and 64bit MT5)

- [mql-zmq](#mql-zmq)
  - [Introduction](#introduction)
  - [Files and Installation](#files-and-installation)
  - [About string encoding](#about-string-encoding)
  - [Notes on context creation](#notes-on-context-creation)
  - [Usage](#usage)

## Introduction

This is a complete binding of the [ZeroMQ](http://zeromq.org/) library
for the MQL4/5 language provided by MetaTrader4/5.

Traders with programming abilities have always wanted a messaging solution like
ZeroMQ, simple and powerful, far better than the PIPE trick as suggested by the
official articles. However, bindings for MQL were either outdated or not
complete (mostly toy projects and only basic features are implemented). This
binding is based on latest 4.2 version of the library, and provides all
functionalities as specified in the API documentation.

This binding tries to remain compatible between MQL4/5. Users of both versions
can use this binding, with a single set of headers. MQL4 and MQL5 are basically
the same in that they are merged in recent versions. The difference is in the
runtime environment (MetaTrader5 is 64bit by default, while MetaTrader4 is
32bit). The trading system is also different, but it is no concern of this
binding.

## Files and Installation

This binding contains three sets of files:

1. The binding itself is in the `Include/Zmq` directory. *Note* that there is a
   `Mql` directory in `Include`, which is part of
   the [mql4-lib](https://github.com/dingmaotu/mql4-lib). Previous `Common.mqh`
   and `GlobalHandle.mqh` are actually from this library. At release 1.4, this
   becomes a direct reference, with mql4-lib content copied here verbatim. It is
   recommended you install the full mql4-lib, as it contains a lot other
   features. But for those who want to use mql-zmq alone, it is OK to deploy
   only the small subset included here.

2. The testing scripts and zmq guide examples are in `Scripts` directory. The
   script files are mq4 by default, but you can change the extension to mq5 to
   use them in MetaTrader5.

3. Precompiled DLLs of both 64bit (`Library/MT5`) and 32bit (`Library/MT4`)
   ZeroMQ (4.2.0) and libsodium (1.0.11) are provided. Copy the corresponding
   DLLs to the `Library` folder of your MetaTrader terminal. If you are using
   MT5 32bit, use the 32bit version from `Library/MT4`. **The DLLs require that
   you have the latest Visual C++ runtime (2015)**.

   *Note* that if you are using **MT5 32bit**, you need to comment out the
   `__X64__` macro definition at the top of the `Include/Mql/Lang/Native.mqh`. I
   assume MT5 is 64 bit, since their is no way to detect 32 bit by native
   macros, and to define pointer related values a macro like this is required.
   
   *Note* that these DLLs are compiled from official sources, without any
   modification. You can compile your own if you don't trust these binaries. The
   `libsodium.dll` is copied from the official binary release. If you want to
   support security mechanisms other than `curve`, or you want to use transports
   like OpenPGM, you need to compile your own DLL.
   
   *Note* for WINE users, if the default binaries do not work for you, you can
   try the binaries in the `Library/VC2010` directory. The new binaries are a
   little newer (libzmq 4.2.2 and libsodium 1.0.36). They are compiled with
   Visual C++ 2010 Express SP1 (using the Windows SDK 7.1), and supposed to be
   more compatible to WINE than the VS2015 version. They depend on VC2010
   runtime (msvcr100.dll and msvcp100.dll). I have actually tested the old and
   the new DLLs on WINE 2.0.3 (Debian Jessie PlayOnLinux 32bit with MetaTrader4
   build 1090) and they both work. So it is *not* guarenteed but it is nice to
   have an alternative. The new libzmq.dll only runs on vista or newer windows
   because I turned on the `using poll` option. This improves performance a
   little bit. Since MetaTrader4 officially no longer supports Windows XP, I
   assume this would not be a problem.

## About string encoding

MQL strings are Win32 UNICODE strings (basically 2-byte UTF-16). In this binding
all strings are converted to utf-8 strings before sending to the dll layer. The
ZmqMsg supports a constructor from MQL strings, the default is _NOT_
null-terminated.

## Notes on context creation

In the official guide:

> You should create and use exactly one context in your process. Technically,
> the context is the container for all sockets in a single process, and acts as
> the transport for inproc sockets, which are the fastest way to connect threads
> in one process. If at runtime a process has two contexts, these are like
> separate ZeroMQ instances.

In MetaTrader, every Script and Expert Advsior has its own thread, but they all
share a process, that is the Terminal. So it is advised to use a single global
context on all your MQL programs. The `shared` parameter of `Context` is used
for sychronization of context creation and destruction. It is better named
globally, and in a manner not easily recognized by humans, for example:
`__3kewducdxhkd__`

## Usage

You can find a simple test script in `Scripts/Test`, and you can find examples
of the official guide in Scripts/ZeroMQGuideExamples. I intend to translate all
examples to this binding, but now only the hello world example is provided. I
will gradually add those examples. Of course forking this binding if you are
interested and welcome to send pull requests.