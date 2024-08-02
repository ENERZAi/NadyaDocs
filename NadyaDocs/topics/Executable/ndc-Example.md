# Example

Here is simple project for example.

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

## Single file compilation

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

## Full project compilation

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

## Target file compilation

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

## template arguments

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
