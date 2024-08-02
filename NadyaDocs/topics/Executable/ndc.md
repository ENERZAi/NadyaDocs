# Nadya optimizing compiler

`ndc` is a compiler binary for `nadya` language. You can create a binary or library files by using `ndc`.
Internally, `ndc` use other nadya binaries and `clang`, so they are must be installed before use `ndc`.

For reference on command line interface for `ndc`, please refer to following document.

<a href="ndc-Command-Line.md">ndc command line interface reference</a>

## Creating Simple project in Nadya

Here is simple Nadya project. Let's go through how they can be compiled

```yaml
project:
  name: ndc-example
  version: 0.1.0

file:
  - Apple:
      - apple.ndy
  - Banana:
      - banana.ndy
  - main.ndy
```

```c#  
attr[ Extern ]
fun bigApple() -> i32 {
    1
} 

!{ apple() }
```

{ collapsible="true" collapsed-title="Apple/apple.ndy" default-state="expanded"}

```c#
module Apple.apple

attr[ Extern ]
fun superBanana() -> i32 {
    2 + Apple.apple.bigApple()
}

_end_
```

{ collapsible="true" collapsed-title="Banana/banana.ndy" default-state="expanded"}

```c#
module Banana.banana

attr[ Extern ]
fun main() -> i32 {
    print(3 + Banana.banana.superBanana())
    0
}

_end_
```

{ collapsible="true" collapsed-title="main.ndy" default-state="expanded"}

There are **three ways** compilation in `ndc`.

1. single file compilation
    - This compilation only compiles a file. In this case, you can't use any other modules without nadya core library.
2. full project compilation
    - This compilation compiles whole project. You can use your own other modules free, but you can't control some
      details like compiling specific file.
3. target file compilation
    - This compilation compiles a file and others which are related to target. You can also use your own other modules
      free, `ndc` finds appropriate files in `nadya-project.yml`.

For more details, refer to 
<a href="ndc-Command-Line.md">ndc command line interface reference</a>

### Single file compilation

In above example, we can't compile `banana.ndy` and `main.ndy` because they depend on other modules which are not core
libraries. So, now we can only compile `apple.ndy` for single file compilation.

```Shell
ndc-project$ ndc Apple/apple.ndy -o Apple/apple.o --lowering-level=object
# `-o` option is alias for `--output`.
```

Now we got an object file of `Apple/apple.ndy` and it includes function `bigApple() -> i32`. You can use this function
by another language by linking. Below is a example which is using `C++`.

```C++
#include <iostream>

extern "C" int bigApple();

int main() {
   std::cout << bigApple() << '\n';
   return 0;
}
```

{ collapsible="true" collapsed-title="test.cc" default-state="expanded"}

```Shell
ndc-project$ cpp test.cc Apple/apple.o -o test.o 
ndc-project$ ./test.o 
1
```

### Full project compilation

You can give a workspace directory which includes `nadya-project.yml` to `ndc`, if not, `ndc` will go up to the root
and find it. If there were not `nadya-project.yml`, it will print error message.

Now, we can compile `ndc-project`.

```Shell
ndc-project$ ndc -w . -o ndc-project.o
# `-w` option is alias for `--workspace` 
```

In `main.ndy`, we have **extern** `main` function, this executable file has entrypoint. So, we can executable object
file alone.

```Shell
ndc-project$ ./ndc-project.o
6
```

### Target file compilation

You can compile a target file. This compilation also need project file `nadya-project.yml` and target file should be
included. `ndc` will find all related modules and compile by appropriate order.

Now we only compile `Banana/banana.ndy` and make it to a shared library.

```Shell
ndc-project$ ndc -t Banana/banana.ndy --object=shared -o banana.so
```

We can call `superBanana` function in other languages. Below one is the `C++` example.

```C++
#include <iostream>

extern "C" int superBanana();

int main() { 
   std::cout << superBanana() << '\n';
}
```
{ collapsible="true" collapsed-title="test2.cc" default-state="expanded"}

Let's compile and execute object.

```Shell
ndc-project$ cpp test2.cc banana.so -o test2.o
ndc-project$ LD_LIBRARY_PATH=./ ./test2.o
3
```

This three ways are prepared in `ndc`. Basically, you will use these ways and add some specific options.

### template arguments

Template arguments can be delivered by `ndc` when `target compilation`. Let's check another simple example.

```yaml
project:
  name: ndc-example2
  version: 0.1.0

file:
  - ...
  - target.ndy
  - ...
```
{ collapsible="true" collapsed-title="nadya-project.yml" default-state="expanded"}

```C#
template </ manglingInfo : string />
attr[ Extern : "target_" + manglingInfo ]
fun externFunc() -> i32 {
   50
}
```

Just skipped other modules, but it isn't important that they are exists or not in this topic. If you compile this code
as target, you **never get any runtime code or function**, because the function `externFunc` isn't called by any caller.
You can make a caller in command line.

```Shell
ndc-example2$ ndc -t target.ndy -o target.so --function \
                externFunc -a "{\"50\"}" --object=shared
# This option takes a name of function.

# -a is alias for --template-arguments. 
# This option takes template arguments of -f option.

# -a option needs --function
```

Now we can find function `target_50` in `target.so` created by above command.


## Cross-platform compilation

For cross-platform compilation, you need `target triple` (and `lld` if you need). `target triple` has the general
format `<arch>-<sub>-<vender>-<sys>-<env>`. More details are
available [here](https://clang.llvm.org/docs/CrossCompilation.html).


Below file is a single nadya file `main.ndy`, and we'll compile it to `aarch64-unknown-linux-gnu`.

Default linker (usually `ld`) usually doesn't support emulating various architectures, so we will use `lld` in this
case. You can easily install this
binary by package managers(`apt`, `dnf`, etc ...) or build `LLVM`.

```C#
// main.ndy
attr[ Extern ]
fun main() -> i32 {
    let a = tensor((16,), 1f16)
    let b = tensor((16,), 2f16)
    let c = a * b
    cast<i32>(c[(0,)])
}

!{ main() }
```

{ collapsible="true" default-state="expanded" collapsed-title="main.ndy"}

There are two compilation way to another platform, using `--sysroot` or using cross compile library.

- using `--sysroot`

If you have all libraries in directory which have `lib`, `include` directory, you can use this directory for **system
root directory** by using `--sysroot` option. System directory requires
libraries(`libc.so`, `libgcc.a`, `libgcc_s.so`, (`libomp.so`)) and binaries(`Scrt1.o`, `crti.o`, `crtbeginS.o`...). It
is recommended to prepare using a docker, qemu and `apt` package manager.

- using cross compile library

We can install some cross compile library by using package manager simply. For `ndc` we need
only `libgcc-{version}-dev-{platform}-cross` and easily install by using `apt` package manager.

### x86_64 (64bit Intel & AMD processors)

For example, we'll compile the single nadya file to `x86_64-unknown-linux-gnu` and `znver4` (amd zen4 architecture). We
need some tools and options for this compilation.

Below is the example directory. `nadya_0.1.0.deb` is package file for install and `cross.dockerfile` is for docker
image.

```Shell
~/test $ tree
.
├── main.ndy
├── cross.dockerfile
└── nadya_0.1.0.deb
```

#### using `--sysroot`

For using system root, we will build a docker image which has `x86_64` root. `libgcc-12-dev` is requirements for
compilation. `nadya_0.1.0.deb` file has dependency in `LLVM` repository, so add it.

```Docker
FROM --platform=x86_64 ubuntu:22.04 AS cross

RUN apt update
RUN apt install libgcc-12-dev

FROM ubuntu:22.04 AS base 

COPY --from=cross / /app/x86_64-linux-gnu

RUN apt update
RUN apt install wget software-properties-common -y

RUN bash -c "wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc"
RUN add-apt-repository "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-17 main"
```

{ collapsible="true" default-state="expanded" collapsed-title="cross.dockerfile"}

Now build this dockerfile to image and run. (If there were not qemu configuration, it will be failed.) We will bind a
directory for nadya install and compilation result.

```Shell
~/test $ docker build -t nadya-cross -f cross.dockerfile ./ 
~/test $ docker run -it --rm -v $(pwd):/app/cross nadya-cross /bin/bash
```

Now install nadya and compile main.ndy.

```Shell
/app/cross# apt install ./nadya_0.1.0_amd64.deb -y
# install nadya, clang, lld, mlir-tools ...etc
/app/cross# ndc test.ndy -o test.out --triple=x86_64-unknown-linux-gnu -mcpu=znver4 -march=znver4 -fuse-ld=lld --sysroot=/app/x86_64-linux-gnu
```

Now the compilation is completed, and we can execute `test.out` in `x86_64` device which is amd zen4 architecture.

However, this method is highly intuitive but is not recommended due to number of disadvantages.

1. must be prepared qemu and docker configuration
2. docker image is fat because of unnecessary files from cross-platform image.

#### using cross compile library

There are `libgcc-{version}-dev-{platform}-cross` in `apt` package manager, so you can just install and compile.

```Shell
~/test $ docker run -it --rm -v $(pwd):/app/cross ubuntu:22.04
...
/app/cross# apt update
/app/cross# apt install libgcc-12-dev-amd64-cross
/app/cross# apt install wget software-properties-common -y && \
            wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc && \
            add-apt-repository "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-17 main"
/app/cross# apt install ./nadya_0.1.0.deb -y
# install nadya, clang, lld, mlir-tools ...etc
```

The `clang` in `ndc` just find the location of `libgcc` and other library files, so just compile `main.ndy`.

```Shell
/app/cross # ndc test.ndy -o test.out --triple=x86_64-unknown-linux-gnu -mcpu=znver4 -march=znver4 -fuse-ld=lld
```

We highly recommend this method because of simplicity, but the intervention of other libraries cannot be predicted, so
we can't guarantee that there are not pollution.

### aarch64 (64bit ARM processors)

Using system root is almost same with `x86_64` cross compilation, so skip this and show another one.

We'll compile `main.ndy` for `aarch64-unknown-linux-gnu` and `cortex-a77` which has fp16 features. Compilation using
library, we need `libgcc-{version}-dev-arm64-cross`, so just install it.

```Shell
~/test $ docker run -it --rm -v $(pwd):/app/cross ubuntu:22.04
...
/app/cross# apt update
/app/cross# apt install libgcc-12-dev-arm64-cross
/app/cross# apt install wget software-properties-common -y && \
            wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc && \
            add-apt-repository "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-17 main"
/app/cross# apt install ./nadya_0.1.0.deb -y
# install nadya, clang, lld, mlir-tools ...etc
```

Arm devices are more complicate to compilations because of architecture. In `main.ndy`, there are some `f16`
operations, so we need to notify to compiler that the device can use `fp16` operations. cortex-a77 has fp16 instruction
sets and is `armv8.2-a` architecture, so `-march` option will be `armv8.2-a+fp16` at least. (If `+fp16` were not given,
compiler just manipulation fp16 tricky, so very slow)

```Shell
/app/cross# ndc test.ndy -o test.out --triple=aarch64-unknown-linux-gnu -march=armv8.2-a+fp16 -fuse-ld=lld 
```

Now, we can execute the `test.out` in `cortex-a77` device. However, this `elf` file has `fp16` instructions so, not
every device can execute this file.

### aarch64 (android)

It is almost same as aarch64, but we need to specify the clang which is built for android cross compilation.

We'll compile `main.ndy` for `aarch64-unknown-linux-android34`. (`-march`, `-mcpu` also usable but skip in this case.)

```Shell
~/test $ docker run -it --rm -v $(pwd):/app/cross ubuntu:22.04
...
/app/cross# apt update
/app/cross# apt install libgcc-12-dev-arm64-cross
/app/cross# apt install wget software-properties-common unzip -y && \
            wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc && \
            add-apt-repository "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-17 main"
/app/cross# apt install ./nadya_0.1.0.deb -y
# install nadya, clang, lld, mlir-tools ...etc

# android ndk download
/app/# wget https://dl.google.com/android/repository/android-ndk-r26b-linux.zip
/app/# unzip android-ndk-r26b-linux.zip 
/app/# export NDK_PATH=$(pwd)/android-ndk-r26b
```

```Shell
/app/cross# ndc test.ndy -o test.out --triple=aarch64-unknown-linux-android34 \
            -mandroid-clang=$NDK_PATH/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android34-clang
```

If you want to compile for another architecture or android version, you need to select the clang which is built for
appropriate architecture or version. Now you can execute `test.out` in android device.
