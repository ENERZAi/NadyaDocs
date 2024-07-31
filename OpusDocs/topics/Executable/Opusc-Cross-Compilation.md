# Cross compilation

`opusc` is a compiler binary for `opus` language. You can create a binary or library files by using `opusc`.
Internally, `opusc` use other opus binaries and `clang`, so they are must be installed before use `opusc`.

## Cross-platform compilation

For cross-platform compilation, you need `target triple` (and `lld` if you need). `target triple` has the general
format `<arch>-<sub>-<vender>-<sys>-<env>`. More details are
available [here](https://clang.llvm.org/docs/CrossCompilation.html).

Below file is a single opus file `main.opus`, and we'll compile it to `aarch64-unknown-linux-gnu`.

Default linker (usually `ld`) usually doesn't support emulating various architectures, so we will use `lld` in this
case. You can easily install this
binary by package managers(`apt`, `dnf`, etc ...) or build `LLVM`.

```C#
// main.opus
attr[ Extern ]
fun main() -> i32 {
    let a = tensor((16,), 1f16)
    let b = tensor((16,), 2f16)
    let c = a * b
    cast<i32>(c[(0,)])
}

!{ main() }
```

{ collapsible="true" default-state="expanded" collapsed-title="main.opus"}

There are two compilation way to another platform, using `--sysroot` or using cross compile library.

- using `--sysroot`

If you have all libraries in directory which have `lib`, `include` directory, you can use this directory for **system
root directory** by using `--sysroot` option. System directory requires
libraries(`libc.so`, `libgcc.a`, `libgcc_s.so`, (`libomp.so`)) and binaries(`Scrt1.o`, `crti.o`, `crtbeginS.o`...). It
is recommended to prepare using a docker, qemu and `apt` package manager.

- using cross compile library

We can install some cross compile library by using package manager simply. For `opusc` we need
only `libgcc-{version}-dev-{platform}-cross` and easily install by using `apt` package manager.

## x86_64

For example, we'll compile the single opus file to `x86_64-unknown-linux-gnu` and `znver4` (amd zen4 architecture). We
need some tools and options for this compilation.

Below is the example directory. `opus_0.1.0.deb` is package file for install and `cross.dockerfile` is for docker
image.

```Shell
~/test $ tree
.
├── main.opus
├── cross.dockerfile
└── opus_0.1.0.deb
```

### using `--sysroot`

For using system root, we will build a docker image which has `x86_64` root. `libgcc-12-dev` is requirements for
compilation. `opus_0.1.0.deb` file has dependency in `LLVM` repository, so add it.

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
directory for opus install and compilation result.

```Shell
~/test $ docker build -t opus-cross -f cross.dockerfile ./ 
~/test $ docker run -it --rm -v $(pwd):/app/cross opus-cross /bin/bash
```

Now install opus and compile main.opus.

```Shell
/app/cross# apt install ./opus_0.1.0_amd64.deb -y
# install opus, clang, lld, mlir-tools ...etc
/app/cross# opusc test.opus -o test.out --triple=x86_64-unknown-linux-gnu -mcpu=znver4 -march=znver4 -fuse-ld=lld --sysroot=/app/x86_64-linux-gnu
```

Now the compilation is completed, and we can execute `test.out` in `x86_64` device which is amd zen4 architecture.

However, this method is highly intuitive but is not recommended due to number of disadvantages.

1. must be prepared qemu and docker configuration
2. docker image is fat because of unnecessary files from cross-platform image.

### using cross compile library

There are `libgcc-{version}-dev-{platform}-cross` in `apt` package manager, so you can just install and compile.

```Shell
~/test $ docker run -it --rm -v $(pwd):/app/cross ubuntu:22.04
...
/app/cross# apt update
/app/cross# apt install libgcc-12-dev-amd64-cross
/app/cross# apt install wget software-properties-common -y && \
            wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc && \
            add-apt-repository "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-17 main"
/app/cross# apt install ./opus_0.1.0.deb -y
# install opus, clang, lld, mlir-tools ...etc
```

The `clang` in `opusc` just find the location of `libgcc` and other library files, so just compile `main.opus`.

```Shell
/app/cross # opusc test.opus -o test.out --triple=x86_64-unknown-linux-gnu -mcpu=znver4 -march=znver4 -fuse-ld=lld
```

We highly recommend this method because of simplicity, but the intervention of other libraries cannot be predicted, so
we can't guarantee that there are not pollution.

## aarch64

Using system root is almost same with `x86_64` cross compilation, so skip this and show another one.

We'll compile `main.opus` for `aarch64-unknown-linux-gnu` and `cortex-a77` which has fp16 features. Compilation using
library, we need `libgcc-{version}-dev-arm64-cross`, so just install it.

```Shell
~/test $ docker run -it --rm -v $(pwd):/app/cross ubuntu:22.04
...
/app/cross# apt update
/app/cross# apt install libgcc-12-dev-arm64-cross
/app/cross# apt install wget software-properties-common -y && \
            wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc && \
            add-apt-repository "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-17 main"
/app/cross# apt install ./opus_0.1.0.deb -y
# install opus, clang, lld, mlir-tools ...etc
```

Arm devices are more complicate to compilations because of architecture. In `main.opus`, there are some `f16`
operations, so we need to notify to compiler that the device can use `fp16` operations. cortex-a77 has fp16 instruction
sets and is `armv8.2-a` architecture, so `-march` option will be `armv8.2-a+fp16` at least. (If `+fp16` were not given,
compiler just manipulation fp16 tricky, so very slow)

```Shell
/app/cross# opusc test.opus -o test.out --triple=aarch64-unknown-linux-gnu -march=armv8.2-a+fp16 -fuse-ld=lld 
```

Now, we can execute the `test.out` in `cortex-a77` device. However, this `elf` file has `fp16` instructions so, not
every device can execute this file.

## aarch64 android

It is almost same as aarch64, but we need to specify the clang which is built for android cross compilation.

We'll compile `main.opus` for `aarch64-unknown-linux-android34`. (`-march`, `-mcpu` also usable but skip in this case.)

```Shell
~/test $ docker run -it --rm -v $(pwd):/app/cross ubuntu:22.04
...
/app/cross# apt update
/app/cross# apt install libgcc-12-dev-arm64-cross
/app/cross# apt install wget software-properties-common unzip -y && \
            wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc && \
            add-apt-repository "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-17 main"
/app/cross# apt install ./opus_0.1.0.deb -y
# install opus, clang, lld, mlir-tools ...etc

# android ndk download
/app/# wget https://dl.google.com/android/repository/android-ndk-r26b-linux.zip
/app/# unzip android-ndk-r26b-linux.zip 
/app/# export NDK_PATH=$(pwd)/android-ndk-r26b
```

```Shell
/app/cross# opusc test.opus -o test.out --triple=aarch64-unknown-linux-android34 \
            -mandroid-clang=$NDK_PATH/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android34-clang
```

If you want to compile for another architecture or android version, you need to select the clang which is built for
appropriate architecture or version. Now you can execute `test.out` in android device.

