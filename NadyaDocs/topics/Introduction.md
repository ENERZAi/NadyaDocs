# Nadya

**Nadya** is the programming language for writing high performance applications in quick and easy way.
Nadya lets programmers write fast and safe programs without knowing details about low level optimizations.

It was primarily designed to write AI computing kernels in easy & extensible way. However,
we are developing it further to be used for general purpose programming, especially where performance critical
computations are required.

### Design purpose ###
1. Easy & soft learning curve for starters writing HPC programs
2. Complete & stand-alone
3. Intelligent optimization pipeline
4. Fully customizable
5. Memory safety

Nadya is aiming to be general programming language specialized for high-performance computing. 

Nadya is solution between high-level optimizing tools & languages (such as OpenAI Triton, Mojo, XLA) and low-level
programming languages (such as Rust, C, C++, Assembly, ...)



### Why Nadya?

#### Why did we end up developing Nadya

Our primary goal was to create framework that can accelerate AI workloads by optimizing the computations.
To do so, we had to write optimized operations in C & assembly one by one for all operations we are willing to support.
There were too many of them, and it was very challenging to make them faster than compute kernels from other state of the art
inference frameworks. Therefore, we started thinking of new programming language so we can benefit from stronger compiler optimizations.

__Previous works on building optimizing compilers__

Researches on optimizing compiler has been developing into way that compiler is in charge of everything, while
programmers don't need to care much about optimizations. As result, lots languages & frameworks has been developed.
(such as OpenAI Triton, Mojo, XLA, JAX (Pallas))

However, compilers are not key to every kind of optimization. There are parts that humans can  do better, while there are parts
compilers can do better. This kind of situation usually occurs since program itself lacks safety guarantees. Compiler optimizations
must be rock-solid, so it should always preserve original behavior. This makes compiler optimizations very conservative.

On the other hand, there were researches for optimizing low-level programming language in better way. Most of them were
done with LLVM IR, and they try to infer high-level context information from low-level information. To get better grasp of
the program, they often make assumptions on the program. For instance, polyhedral optimization framework implemented in LLVM (polly)
tries to derive loop conditions using set of hyperplanes which is derived from the loop condition. However, this requires loop body
to be in restrictive form, without any side-effects that polyhedral analyzer cannot model.
There were also attempts to capture dynamic information while code is running. llvm-bolt for example captures dynamic information from the program, and
uses it for link optimization.

__How Nadya stacks up__

We thought if we could build language that is friendly for both programmer and compiler. Semantics of Nadya has been designed
 to be compiler-friendly, so that compiler can capture enough information without performing expensive analysis.
Moreover, we decided to make it low-level enough so programmers, who knows intention of their program better than anyone.
So programmers can optimize the implementation themselves when required. This capability is maximized
with integrated code-generation capability, which will be explained later. We believe this makes Nadya to benefit
both from optimizations from programmer, and compiler. As a result, we can write very fast & efficient implementations without
going too low-level. 


#### Utilizing high-level information for compiler optimization
Nadya is designed to be friendly for both compilers and programmers. Programmers can express their code using high-level
expressions, while compilers can grasp more information about what programmer is meant to do, and using those for optimizations.

For example, in C, it's difficult to assume how this addition can be optimized.
There exists auto-vectorizer such as slp-vectorizer or loop vectorizer, but compiler must derive those high-level information
by analyzing low-level implementations

```c
int add(int* lhs, int* rhs, int rowSize, int columnSize){
  // From this context, compiler must derive innermost loop is vectorizable.
  // Deriving high-level information from low-level implementation 
  // can be challenging
  for(int i = 0; i < rowSize; ++i){
     for(int j =0; j < columnSize; ++j){
        lhs[i*columnSize + j] = 
            lhs[i*columnSize + j] + rhs[i*columnSize + j];
     }
  }
  
  return 0;
}
```

This could have be written in simpler form in Nadya. Nadya compiler captures shape, rank and data type of the 
data, and automatically creates vectorized implementation for adding two input parameters.
```c#
// mut& means value is captured as a mutable borrow (reference)
fun add(mut &lhs : tensor<i32, 2>, rhs : tensor<i32, 2>) -> i32 {
   // In Nadya, shape of tensor is implicitly included in tensor object
   lhs <- lhs + rhs
   0
}
```

So, we decided to develop a new programming language, called Nadya.
Nadya was designed to convey maximum amount of information to the compiler.


#### Metaprogramming (Code generation)

In many cases, programmers wish to separate implementation of function that performs same role, depending on the environment,
target device, and more. In this case, programmers are usually required to separate the implementation, or divided them into 
multiple function calls, which can be quite annoying.
If one is writing some computational function for optimizations, it becomes very complicated. For instance, XNNPACK framework
from Google uses different assembly & C implementations for each input conditions, and target device.

You may ask, since Nadya semantics is transparent between architectures, it's enough for deploying single implementation for
multiple conditions. However, for the best fine-tuned performance, it's useful for programmers to separate the implementation for certain level.

For example, for ARM based CPUs, performing matrix multiplication with 3~4 tiled loops is more efficient. On the other hand,
for X86 CPUs, aggressively separating loops deeper, and performing packing is more efficient. These kind of optimizations
that compiler is difficult to handle, may be done by the programmer.

For programmers to perform this in single implementation, we introduce metaprogramming capability, where programmers can program
Nadya code that constructs Nadya abstract syntax tree, which could be compiled later.

This is one of the strongest, and unique feature of Nadya.
Nadya can create its own 'AST' (Abstract syntax tree) and combine them to build new code.
This allows programmers to write very flexible code that can be adopted in many different scenarios.
For example, Programmers can make Nadya generate code that uses different set of algorithms depending on
situations, such as program inputs and target hardware.

```c#
// Generate code creating default matmul, or tiled matmul depending on input size
// expression stores generated code by each function

let mut expression = !{0}

if(inputSize < threshold) {
    expression <- generateDefaultMatmul();
} else {
    expression <- generateTiledConv();
}

// Use generated code for building gemm algorithm
// !{ code } represents generated code
//! ${ code } represents instantiation of result into the generated code.
let gemm = !{ alpha * ${expression}  + beta * bias }

// ...
```

In Nadya "Expression", "Type" and "Function object" can be treated as value. In other words, programmers can 
combine these as building block for building more complicated program. This opens way for to generate different code
with different hyper-parameters, which greatly reduces cost of implementing separate implementation that does similar
function.

#### Intelligent compiler optimizations

**Builtin support for hardware intrinsics**

Programmer doesn't need  to be aware of specific intrinsics that's dependent on target hardware and platform,
Nadya gracefully supports them, maximizing capability of target hardware.

<code-block lang="c#">
let tensorA = tensor((2, 4), 0.0f)
let tensorB = tensor((1, 4), 1.0f)
// Results in (2, 4) tensor. Utilizes optimized SIMD operations internally
// ex) AVX on amd64, Neon or SME for ARM chips
let sum = tensorA + tensorB
</code-block>

**Automatic loop optimizations**

Nadya can unroll, parallelize, fuse loops without violating program safety.
These process can be done automatically, or can be instructed by programmer.
This takes reduces effort for optimizing the code for specific hardware target.

```c#
// Nadya compiler can automatically parallelize the loop
attr [ Parallel : true ]
for(idx from 0 to 100 step 2){
    // Loop contents
}
```

**Memory optimizations**

Nadya can automatically reduce unnecessary load & stores and other memory operations by aggressively
removing & optimizing them. This includes automatic in-placing, hoisting memory allocations, and many more.
Memory load & stores take lots of portion in optimization, and Nadya can help programmers doing so.


#### Memory safety

Nadya is designed to minimize human errors with memory. There is no explicit `malloc` or `free` exposed to programmer.
Nadya compiler itself looks for suitable location where memory should be freed, and takes care of it
to prevent memory leaks.

#### Functional-style programming pattern

Functional programming has many advantages.
It is concise, clear, and prevents mistakes effectively.

<tabs>
    <tab title="Functional">
        <code-block lang="c#">
let myFun a b = a + b
print(myFun 3 5) // prints '8'
let subFun = myFun 3 // 'subset' of function can be defined as a value
print(subFun 5) // prints '8'
        </code-block>
    </tab>
    <tab title="Pattern matching">
        <code-block lang="c#">
let val = Some 3
let output = 
    match val with
    | Some a -> a
    | None -> 0
    print(output) // Prints 3
        </code-block>
    </tab>
</tabs>


#### Straight forward arithmetic operations

Nadya has built-in support for tensor arithmetics, with integrated broadcasting semantics
Nadya compiler can effectively optimize them to generate fast code automatically for every case.

<tabs>
    <tab title="Easy tensor arithmetics">
        <code-block lang="c#">
        let a = tensor((2,3), {1,0f,2,0f,3,0f,4.0f,5.0f,6.0f}) // 32bit floating point tensor shaped (2,3)
        let b = tensor((3,), {1.0f, 2.0f, 3.0f}) // 32bit floating point tensor shaped (3,)
        // Tensor arithmetics (automatically broadcasted)
        print(a + b) // tensor((2,3), {2.0f, 4.0f, 6.0f, 5.0f, 7.0f, 9.0f})
        </code-block>
    </tab>
    <tab title="Reference">
        <code-block lang="c#">
        let mut a = tensor((2,3), {1,0f,2,0f,3,0f,4.0f,5.0f,6.0f}) // 32bit floating point tensor shaped (2,3)
        let mut refA = &a
        refA[0, 0] &lt;- 3.0f // Modifies both a & refA
        print(a) // prints tensor((2,3), {3,0f,2,0f,3,0f,4.0f,5.0f,6.0f})
        </code-block>
    </tab>
    <tab title="Bitwise operations">
        <code-block lang="c#">
        // Bit operations can be performed on both tensors & scalar values
        let a, b = 3, 4
        print(a | b)
        print(a & b)
        let tensorA = tensor((2,), {1, 2})
        let tensorB = tensor((2,), {3, 4})
        print(tensorA | tensorB)
        print(tensorA & tensorB)
        </code-block>
    </tab>
</tabs>

<seealso>
    <category ref="references">
        <a href="https://www.indeed.com/career-advice/career-development/functional-programming-languages">What is functional programming?</a>
    </category>
</seealso>

### Upcoming features

Nadya is in early-development phase, and new features are being developed and added actively.
We are currently working on many features aiming to build very powerful language.
Expect Nadya getting better and better overtime, with following new features!

1. **Methods on structs**
   * User struct can have methods, that lets programmer to use oop-like patterns
   * Planned to be supported on version 0.3.x
2. **Ownership system**
   * This is one of the feature we're putting our most effort on. This ensures memory safety without using garbage collector (which is bad for performance), and lets programmer seamlessly program in Nadya without worrying about memory leaks.
   * Planned to be supported on version 0.3.x
3. **Type trait system**
   * Similar to 'Concept' on C++20, programmers can add restrictions or conditions on type 
   * Programmers can provide required implementations expected to be implemented by types inheriting trait (Something similar to an interface in C# or Java)
   * Planned to be supported from version 0.3.x
4. **Smarter optimization algorithm regarding fusion**
   * Fusing tensor operations can reduce useless loops, or control flow hazards which is beneficial for both CPUs and GPUs.
   * Planned to be supported from version 0.4.x
5. **Memory access boundary checking**
    * If programmer accesses memory out of bounds, and if compiler can detect it, compiler will notify programmer.
6. **Virtual machine support for running compiled bytecode regardless of target environment or platform**
7. **Runtime library supporting advanced library functions**
   * Planned to be supported from version 0.3.x
8. **Debugging & stack unwinding**
    * In order to improve debugging experience, we are adding stack tracking, and gdb support
9. **GPU support**
   * Builtin Support for platforms such as Vulkan & cuda without using new kind of syntax or semantics
