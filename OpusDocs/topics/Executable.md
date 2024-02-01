# Executable

## opusc

`opusc` is a compiler binary for `opus` language. You can create a binary or library files by using `opusc`.
Internally, `opusc` use other opus binaries and `clang`, so they are must be installed before use `opusc`.

### Example

Here is simple project for example.

```yaml
project:
  name: opusc-example
  version: 0.1.0

file:
  - Apple:
      - apple.opus
  - Banana:
      - banana.opus
  - main.opus
```

```c#  
attr[ Extern ]
fun bigApple() -> i32 {
    1
} 

!{ apple() }
```

{ collapsible="false" collapsed-title="Apple/apple.opus"}

```c#
module Apple.apple

attr[ Extern ]
fun superBanana() -> i32 {
    2 + Apple.apple.bigApple()
}

_end_
```

{ collapsible="false" collapsed-title="Banana/banana.opus"}

```c#
module Banana.banana

attr[ Extern ]
fun main() -> i32 {
    print(3 + Banana.banana.superBanana())
    0
}

_end_
```

{ collapsible="false" collapsed-title="main.opus"}

There are **three ways** compilation in `opusc`.

1. single file compilation
    - This compilation only compiles a file. In this case, you can't use any other modules without opus core library.
2. full project compilation
    - This compilation compiles whole project. You can use your own other modules free, but you can't control some
      details like compiling specific file.
3. target file compilation
    - This compilation compiles a file and others which are related to target. You can also use your own other modules
      free, `opusc` finds appropriate files in `opus-project.yml`.

#### Single file compilation

In above example, we can't compile `banana.opus` and `main.opus` because they depend on other modules which are not core
libraries. So, now we can only compile `apple.opus` for single file compilation.

```Shell
opusc-project$ opusc Apple/apple.opus -o Apple/apple.o --lowering-level=object
# `-o` option is alias for `--output`.
```

Now we got an object file of `Apple/apple.opus` and it includes function `bigApple() -> i32`. You can use this function
by another language by linking. Below is a example which is using `C++`.

```C++
#include <iostream>

extern "C" int bigApple();

int main() {
   std::cout << bigApple() << '\n';
   return 0;
}
```

{ collapsible="false" collapsed-title="test.cc"}

```Shell
opusc-project$ cpp test.cc Apple/apple.o -o test.o 
opusc-project$ ./test.o 
1
```

#### Full project compilation

You can give a workspace directory which includes `opus-project.yml` to `opusc`, if not, `opusc` will go up to the root
and find it. If there were not `opus-project.yml`, it will print error message.

Now, we can compile `opusc-project`.

```Shell
opusc-project$ opusc -w . -o opusc-project.o
# `-w` option is alias for `--workspace` 
```

In `main.opus`, we have **extern** `main` function, this executable file has entrypoint. So, we can executable object
file alone.

```Shell
opusc-project$ ./opusc-project.o
6
```

#### Target file compilation

You can compile a target file. This compilation also need project file `opus-project.yml` and target file should be
included. `opusc` will find all related modules and compile by appropriate order.

Now we only compile `Banana/banana.opus` and make it to a shared library.

```Shell
opusc-project$ opusc -t Banana/banana.opus --object=shared -o banana.so
```

We can call `superBanana` function in other languages. Below one is the `C++` example.

```C++
#include <iostream>

extern "C" int superBanana();

int main() { 
   std::cout << superBanana() << '\n';
}
```

{ collapsible="false" collapsed-title="test2.cc"}

Let's compile and execute object.

```Shell
opusc-project$ cpp test2.cc banana.so -o test2.o
opusc-project$ LD_LIBRARY_PATH=./ ./test2.o
3
```

This three ways are prepared in `opusc`. Basically, you will use these ways and add some specific options.

### template arguments

Template arguments can be delivered by `opusc` when `target compilation`. Let's check another simple example.

```yaml
project:
  name: opusc-example2
  version: 0.1.0

file:
  - ...
  - target.opus
  - ...
```

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
opusc-example2$ opusc -t target.opus -o target.so --function \
                externFunc -a "{\"50\"}" --object=shared
# This option takes a name of function.

# -a is alias for --template-arguments. 
# This option takes template arguments of -f option.

# -a option needs --function
```

Now we can find function `target_50` in `target.so` created by above command.

### Cross-platform compilation

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

### Command line options

#### General Options

- `--version`

  Print version of `opusc`.

- `--help, -h`

  Print command line options.

- `--input, -i <file>`

  Input file (for single file compilation).

- `--output, -o <file>`

  Output file or dir.

- `--target, -t <file>`

  target file (for target compilation).

- `--workspace, -w <file>`

  workspace directory.

- `--function <function name>`

  specify function name for target compilation

- `--template-arguments, -a <values>`

  Template arguments for `--function`

- `--lowering-level=<level>`

  Lowering level ( `compiletime`, `runtime`, `opus-dialect`, `llvm-dialect`, `llvmir`, `object`, `final` -> default)
  This option
  is only for `--lowering-level=final`

- `--ll <uint>`

  Alias for lowering
  level. (`0=compiletime`, `1=runtime`, `2=opus-dialect`, `3=llvm-dialect`, `4=llvmir`, `5=object`, `6=final`)

- `--object`

  Object type (`shared` : shared library, `static` : static library, `executable` : executable file). This options
  needs `--lowering-level=final` or `--ll=6`, if not, this option will be ignored.

- `--print-log=<value>`

  Print internal task and execution of `opusc`. If `value` were `stderr/stdout`, print log to stderr/stdout, or not,
  print to `<value>` file.

- `--verbose, -v`

  Alias for `--print-log=stderr`.

- `-l<library>`

  External libraries for link.

- `-L<dir>`

  Add directories which libraries are exists.

- `-g, --O1, --O2, --O3`

  Optimization level

- `-fuse-ld=<linker>`

  Choose linker

- `-fopenmp`, `-fno-openmp`

  Enable parallel compilation and link with `libomp`. This option requires `libomp.so` in library paths. You can add
  paths by using option `-L<dir>`.

- `-fopt-vector-size=<uint>`

  Make `opus-opt` vectorize to size `<uint>`. This priority is higher than `--cpu` or x86 instruction options
  like `-mavx512f`, `-mavx`, etc... .

- `fopt-stack-threshold=<uint>`

  Make `opus-opt` change stack threshold size to `<uint>`

- `fopt-full-pass="<pass1> <pass2> ..."`

  Ignore default `opus-opt` pass and replace to given pass.

- `-fpic`, `-fPIC`

  Choose reallocation model

#### Pass Options

- `WClang,<arg1>,<arg2>,...` : Pass options to clang internally. `opusc` doesn't check any problem in the arguments.
- `WLinker,<arg1>,<arg2>,...` : Pass options to linker internally. `opusc` doesn't check any problem in the arguments.

#### Target option

- `--triple=<value>` : target triple for cross compilation
- `--cpu=<value>` : target cpu. For a list of available CPUs for the target use "--cpu=help"

- Instruction sets for x86 cpu
    - `-mavx`
    - `-mavx2`
    - `-mavx512f`
    - `-mavx512dq`
    - `-mavx512ifma`
    - `-mfma`
    - `-msse2`
    - `-msse3`
    - `-mssse3`
    - `-msse4.1`
    - `-msse4.2`
