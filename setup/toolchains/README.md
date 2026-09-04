## Build the MINIX toolchain

In theory, you can modify and rebuild the MINIX kernel directly inside MINIX without a cross-toolchain. However, I could not find an easy way to get the rebuilt kernel back out of MINIX and boot it under QEMU.

Back at university, I remember simply using `dd` to write the new kernel image and booting from that.

For this project I decided to build the MINIX/i386 cross-toolchain on Linux instead.

Get the MINIX source from GitHub:

```bash
git clone https://github.com/Stichting-MINIX-Research-Foundation/minix.git minix
cd minix
git checkout R3.3.0
```

The kernel source code is under:

```text
/minix/minix/kernel
```

You can start the toolchain build with:

```bash
sh build.sh -mi386 -O ../obj tools
```

`build.sh -h` will show the available build options. For simplicity, I target i386.

### First build problem

The first problem I hit was a multiple definition error:

```text
/usr/bin/ld: compat.o:(.bss+0x0): multiple definition of `debug_file'; arch.o:(.bss+0x0):
```

It looks like there is a duplicate definition of the global variable `debug_file`. It is quite common to see this sort of thing in legacy code.

After some quick research, the reason turned out to be an old GCC compatibility issue. Older GCC versions defaulted to `-fcommon`, while modern GCC defaults to `-fno-common`.

The old behaviour allowed uninitialised globals such as `debug_file` to be merged by the linker.

To be honest, the best way would be to fix the source code. For now, adding `-fcommon` gets past it:

```bash
export HOST_CFLAGS="-O -fcommon"
sh build.sh -mi386 -O ../obj tools
```

Hint: do not remove `../obj`.

### ASTWriter.cpp

The next problem I encountered was another no matching function error:

```text
Serialization/ASTWriter.cpp:4048:28: error: no matching function for call to
'llvm::BitstreamWriter::EmitRecordWithBlob(unsigned int&,
clang::ASTWriter::RecordData&,
llvm::SmallVectorTemplateCommon<std::pair<unsigned int, unsigned int>, void>::pointer)'
```

After a lengthy look at the log and source code, I found the problem was related to the `data()` helper in `ASTWriter.cpp`.

```bash
grep -n 'data(' \
external/bsd/llvm/dist/clang/lib/Serialization/ASTWriter.cpp
```

The MINIX code has its own `data()` helper:

```cpp
template <typename T>
static StringRef data(const SmallVectorImpl<T> &v)
```

It returns a `StringRef`, which is what `EmitRecordWithBlob()` expects.

The problem is that modern C++ also has `std::data()`. With a newer compiler, the call can end up resolving to `std::data()` instead, which returns a pointer. This is why the argument no longer matches `EmitRecordWithBlob()`.

There are several ways to get around this, such as forcing C++11:

```bash
HOST_CXXFLAGS="-O2 -std=gnu++11"
```

Given this is an old LLVM/Clang codebase, I did not really want to force a different C++ standard across the whole build just for this.

I simply renamed the MINIX `data()` helper to `blobData()` and changed its calls.

From:

```cpp
template <typename T, typename Allocator>
static StringRef data(const std::vector<T, Allocator> &v)

template <typename T>
static StringRef data(const SmallVectorImpl<T> &v)
```

to:

```cpp
template <typename T, typename Allocator>
static StringRef blobData(const std::vector<T, Allocator> &v)

template <typename T>
static StringRef blobData(const SmallVectorImpl<T> &v)
```

I have included the fixed file in the `src` directory.

If successful:

```text
===> Summary of results:

build.sh command:    build.sh -mi386 -O ../obj tools
build.sh started:    21:48:57 CDT 2026

MINIX version:       3.3.0
MACHINE:             i386
MACHINE_ARCH:        i386
Build platform:      Linux 7.0.0-30-generic x86_64

...

TOOLDIR path:        minix/../obj/tooldir.Linux-7.0.0-30-generic-x86_64
DESTDIR path:        minix/dev/minix/../obj/destdir.i386
RELEASEDIR path:     minix/../obj/releasedir
Updated makewrapper: minix/../obj/tooldir.Linux-7.0.0-30-generic-x86_64/bin/nbmake-i386
build.sh ended:      22:20:52 CDT 2026
```

I have included the fixed file in the `src` directory.

## Build MINIX

Once the toolchain is built:

```bash
export HOST_CFLAGS="-O -fcommon"
sh build.sh -mi386 -O ../obj -U distribution
```

If successful:

```text
build.sh command:    build.sh -j12 -mi386 -O ../obj -U distribution
build.sh started:    22:51:07 CDT 2026
MINIX version:       3.3.0
MACHINE:             i386
Build platform:      Linux 7.0.0-30-generic x86_64
Successful make distribution

build.sh ended:      22:58:35 CDT 2026
```