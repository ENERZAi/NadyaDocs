# opusc

`opusc` is a compiler binary for `opus` language. You can create a binary or library files by using `opusc`.
Internally, `opusc` use other opus binaries and `clang`, so they are must be installed before use `opusc`.

## Cross-platform compilation

For cross-platform compilation, you need `target triple` (and `lld` if you need). `target triple` has the general
format `<arch>-<sub>-<vender>-<sys>-<env>`. More details are
available [here](https://clang.llvm.org/docs/CrossCompilation.html).

Below file is a single opus file `main.opus`, and we'll compile it to `aarch64-unknown-linux-gnu`.

```C#
attr [ Extern ]
fun myFunc() -> i32 {
   30
}

!{ myFunc() }
```
{ collapsible="true" collapsed-title="main.opus" default-state="expanded"}

Default linker usually doesn't support emulating `aarch64`, we will use `lld` in this case. You can easily install this
binary by package managers(`apt`, `dnf`, etc ...) or build `LLVM`.

```Shell
$ opusc -i main.opus -o main_aarch64.so --triple=aarch64-unknown-linux-gnu \
  --fuse-ld=lld --object=shared
# --triple option for specifiy target triple
# --fuse-ld option for choosing linker
```

Move `main_aarch64.so` to another appropriate device and link it.

```C++
#include <iostream>

extern "C" int myFunc();

int main() {
   std::cout << myFunc() << '\n';
   return 0;
}
```
{ collapsible="true" collapsed-title="main.cc" default-state="expanded"}

```Shell
aarch64-device$ cpp main.cc main_aarch64.so -o main.o 
aarch64-device$ LD_LIBRARY_PATH=./ ./main.o 
30
```

If you want to specify cpu for using efficient intrinsics, use `--cpu` option. For a list of available CPUs for the
target use `--cpu=help`. Below example is for `cortex-a72`.

```Shell
$ opusc -i main.opus -o main_aarch64.so --triple=aarch64-unknown-linux-gnu \
  --fuse-ld=lld --object=shared --cpu=cortex-a72
```

