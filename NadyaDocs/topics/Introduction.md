# Nadya

**Nadya** is the programming language for writing high performance applications in quick and easy way.
Nadya lets programmers write fast and safe programs without knowing details about low level optimizations.

Its was primarily designed to write AI computing kernels in easy & extensible way. However,
we are developing it further to be used for general purpose programming, especially where performance critical
computations are required.

### Design purpose ###
1. Easy & soft learning curve for starters writing HPC programs
2. Complete & stand-alone
3. Intelligent optimization pipeline
4. Fully customizable
5. Memory safety

### Why Nadya?

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

**Integrated code generation**

One of the strongest, and unique feature of Nadya is builtin code-generation support.
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
let gemm = !{ alpha * ${expression} } + beta * bias}
// ...
```

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

### Upcomming features

Nadya is in early-development phase, and new features are being developed added actively.
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
