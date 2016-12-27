# Control Flow Guard Showcase

[Control Flow Guard](<https://msdn.microsoft.com/en-us/library/windows/desktop/mt637065(v=vs.85).aspx>) (CFG) is Microsoft's implementation of [control flow integrity](https://www.microsoft.com/en-us/research/publication/control-flow-integrity/) in Visual Studio 2015. Control flow integrity (CFI) is an exploit mitigation like stack cookies, DEP, and ASLR. Like other exploit mitigations, the goal of CFI is to prevent bugs from turning into exploits. CFI works by reducing the ability of an attacker to redirect program execution to an attacker controlled destination.

We have created samples with specially crafted bugs to showcase Control Flow Guard's CFI implementation. These examples are an accompaniment to [our blog post about Control Flow Guard](TODO:URL). We also have a blog post about [clang's control flow integrity implementation](https://blog.trailofbits.com/2016/10/17/lets-talk-about-cfi-clang-edition/) and [the accompanying set of examples](https://github.com/trailofbits/clang-cfi-showcase).

The bugs in these examples are not statically identified by the compiler, but are detected at runtime via CFI. Where possible, we simulate potential malicious behavior that occurs without CFI protections.

Each example builds two binaries, one with CFG (e.g. `cfg_icall.exe`) and one without CFG (e.g. `no_cfg_icall.exe`).

# CFG Examples

* **cfg_icall** demonstrates control flow integrity of indirect calls. The example binary accepts a single command line argument (valid values are 0-3, but try invalid values with both binaries!). The command line argument shows different aspects of indirect call protection, or lack thereof.
* **cfg_vcall** shows an example of CFI applied to virtual function calls. This example demonstrates how CFI would protect against a type confusion or similar attack. Included in the example is a bad cast from one class to another unrelated class and simulation of a fully attacker controlled object.
* **cfg_valid_targets**  is like the **cfg_icall** example but uses the [SetProcessValidCallTargets API](<https://msdn.microsoft.com/en-us/library/windows/desktop/dn934202(v=vs.85).aspx>) to remove functions from the valid targets list.
* **cfg_guard_ignore** shows the use of [__declspec(guard(ignore))](<https://github.com/Microsoft/ChakraCore/blob/master/lib/Backend/JnHelperMethod.cpp#L164-L168>) to disable CFG for indirect calls inside a specific method.
* **cfg_guard_nocf** shows the difference between [__declspec(guard(ignore))](<https://github.com/Microsoft/ChakraCore/blob/master/lib/Backend/JnHelperMethod.cpp#L164-L168>) and [__declspec(guard(nocf))](<https://github.com/adobe/avmplus/blob/858d034a3bd3a54d9b70909386435cf4aec81d21/AVMPI/MMgcPortWin.cpp#L54-L75>): Function pointers referenced in ignored functions are not tracked by CFG, but function pointers from nocf functions are tracked by CFG.
* **cfg_guard_suppress** and **cfg_suppressed_export** work together to show how to use [__declspec(guard(suppress))](<http://www.codemachine.com/downloads/win10/ntdef.h>) to prevent an exported function from being called using `LoadLibrary` and `GetProcAddress`.

# Requirements

These examples were tested on Windows 10 with Visual Studio 2015, using Window SDK 10.0.14393.0.

Other platforms should work, but the minimum requirements for Control Flow Guard are Visual Studio 2015 and Windows 8.1 Update.

# Building

Start up a "VS2015 x64 Native Tools Command Prompt" and build via `nmake all`. The expected output is:

     Microsoft (R) Program Maintenance Utility Version 14.00.24210.0
     Copyright (C) Microsoft Corporation.  All rights reserved.
     
             cl.exe /W4 /nologo /Zi /EHsc /guard:cf /Fecfg_guard_nocf.exe  cfg_guard_nocf.cpp /link mincore.lib /guard:cf
     cfg_guard_nocf.cpp
             cl.exe /O2 /W4 /nologo /guard:cf /Fecfg_suppressed_export.dll cfg_suppressed_export.cpp /link mincore.lib /DLL /guard:cf
     cfg_suppressed_export.cpp
        Creating library cfg_suppressed_export.lib and object cfg_suppressed_export.exp
             cl.exe /W4 /nologo /Zi /EHsc /guard:cf /Fecfg_guard_suppress.exe  cfg_guard_suppress.cpp /link mincore.lib /guard:cf
     cfg_guard_suppress.cpp
             cl.exe /W4 /nologo /Zi /EHsc /guard:cf /Fecfg_guard_ignore.exe  cfg_guard_ignore.cpp /link mincore.lib /guard:cf
     cfg_guard_ignore.cpp
             cl.exe /W4 /nologo /Zi /EHsc /guard:cf /Fecfg_vcall.exe  cfg_vcall.cpp /link mincore.lib /guard:cf
     cfg_vcall.cpp
             cl.exe /W4 /nologo /Zi /EHsc /guard:cf /Fecfg_icall.exe  cfg_icall.cpp /link mincore.lib /guard:cf
     cfg_icall.cpp
             cl.exe /W4 /nologo /Zi /EHsc /guard:cf /Fecfg_valid_targets.exe  cfg_valid_targets.cpp /link mincore.lib /guard:cf
     cfg_valid_targets.cpp
             cl.exe /W4 /nologo /Zi /EHsc /Feno_cfg_vcall.exe  cfg_vcall.cpp /link mincore.lib
     cfg_vcall.cpp
             cl.exe /W4 /nologo /Zi /EHsc /Feno_cfg_icall.exe  cfg_icall.cpp /link mincore.lib
     cfg_icall.cpp
             cl.exe /W4 /nologo /Zi /EHsc /Feno_cfg_valid_targets.exe  cfg_valid_targets.cpp /link mincore.lib
     cfg_valid_targets.cpp
             cl.exe /W4 /nologo /Zi /EHsc /Feno_cfg_guard_ignore.exe  cfg_guard_ignore.cpp /link mincore.lib
     cfg_guard_ignore.cpp
             cl.exe /W4 /nologo /Zi /EHsc /Feno_cfg_guard_suppress.exe  cfg_guard_suppress.cpp /link mincore.lib
     cfg_guard_suppress.cpp
             cl.exe /W4 /nologo /Zi /EHsc  /Feno_cfg_guard_nocf.exe  cfg_guard_nocf.cpp /link mincore.lib
     cfg_guard_nocf.cpp

