# Virtual and Native environment

Nadya has two different worlds. First one is virtual environment, and the other is native environment.
Two environments are intended to be feature compatible (there can be unimplemented features on early releases), but
they are different in purpose.

### Virtual environment

Nadya in virtual environment runs on virtual machine or interpreter, rather than native. This maximizes portability of 
Nadya code, without need to recompile them entirely for different platforms (x64, arm, .. etc) since it would run on any 
platform that runs Nadya virtual machine.

<note> Virtual machine is currently under development. 
Nadya in virtual environment runs on tree-traversing interpreter temporarily before virtual machine development is completed. </note>

Nadya in virtual environment also has one special functionality.
It can generate native code that can be compiled down to native environment. Nadya can build nadya expressions in native code in 
virtual environment, which results complete native code. Moreover, Nadya can compute computations that can be 
pre-computed statically, and integrate it to native environment to maximize efficiency. 

Virtual environment is optimal for following purposes

* Program is not sensitive in performance. (Execution time & memory consumption)
* Program is required to run in wide range of platforms without re-compiling
* Native code generation is required
  * This is major use case of virtual environment. You can use virtual environment for generating native code. 
  * Typically used for code optimization, or reducing binary size by computing parts that can be pre-computed before running natively to maximize efficiency.


### Native environment

Nadya in native environment runs natively after compiling towards specific architecture & platform. This maximizes performance
and efficiency of the program. Nadya compiler contains intelligent compiler passes that considers Nadya programming model that would
maximize performance with techniques such as cache optimization, and automatic code vectorization.


#### Writing code in virtual environment

By default, if code is not written inside scope of first-class functions (functions defined using 'fun' in outermost scope),
code is considered to run on virtual environment.

```c#
let a = 3
let b = 4

print(a + b)


fun foo(i32 a){
   a * ${a + b} // Native environment
}
 
```
