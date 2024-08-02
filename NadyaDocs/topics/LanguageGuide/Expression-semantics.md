# Nadya language guide

#### Contents

1. <a anchor="learning-basic-concepts">Basics</a>
2. <a anchor="code-organization">Code organization</a>
3. <a anchor="literals">Literals</a>
4. <a anchor="control-flow">Control flow</a>
5. <a anchor="functions">Functions</a>
6. <a anchor="unions-pattern-matching">Unions & pattern matching</a>
7. <a anchor="helper-features">Helper features</a>

## Learning basic concepts 
1. <a href="Basics.md">__Basics__</a>

    This document goes through basic concepts used in Nadya programming language. Before going through rest of the document, It's recommended to read this first to get a brief view how Nadya program can be written and organized.
2. <a href="ndc.md">__Nadya Optimizing Compiler__</a>

    This document explains how to use Nadya Optimizing Compiler (Named `ndc`) with simple examples. 
3. <a href="Load-Store-rules.md">__Load & store semantics for tensor__</a>

   __Tensor__ is fundamental building block for writing fast implementations that calculates data. 
   This document explains how load & store rules are modeled for tensor in Nadya.

## Code organization
1. <a href="Modules.md">__Modules__</a>
   
    Modules provide efficient way for organizing code into separate parts that performs separate functions. This helps to avoid collisions and improves code readability
2. <a href="Virtual-and-Native-environment.md"> __Virtual and native environment__ </a>
    
    Nadya has two kind of worlds that programmers can express their creativity. Virtual environment and native environment. 
    Code written in virtual environment runs virtually on interpreter (or Nadya virtual machine) to support execution on any kind of platform.
    Virtual environment also has capability to generate native code.
    On the other hand, Native environment is compiled down to binary, and runs natively on target device. It is required to be compiled separately
    depending on the target. However, it brings significantly faster performance, and integrates intelligent optimization pipeline that makes
    Nadya run really fast when compiled with right options.

## Literals
1. <a href="Literals.md">Literals</a>
    
    Learn which literals are supported in Nadya, and how to use them

2. <a href="Tensors.md">Tensors</a>
    
    Tensor is collection of data of specific type, used for efficient computation. Learn how to used tensors
    How to initialize, load and store to tensors, and how they are used for computation.

## Control flow
1. <a href="Let-bindings.md">Let bindings</a>

    Learn how to define value or variable using let bindings. Also, see some useful use-cases for let binding
    
2. <a href="Loops.md">Loops</a>

    Learn how to use loops in Nadya, and how to optimize them

3. <a href="Conditionals.md">Conditionals</a>

    Learn how to use conditional expressions in Nadya in many different ways.


## Functions
1. <a href="Closures.md">Closures</a>

    Closures are fundamental building block in functional programming. 
    Learn how to use closures in Nadya in proper way. See how they can be organized, composed and treated in different use cases 
2. <a href="First-class-functions.md">First class functions</a>
    
    First class functions are used to build functions native code, portable for use in other languages such as C/C++. 
    Also, native code functions are customizable using virtual environment. Learn how to do so, and how to use them in efficient way.

## Unions & pattern matching
1. <a href="Union.md">Union</a>

    Unions provide support for values that can be one or more kind of format. Each format can have different types and values.
    This is useful for cases where single value is required to pack multiple kind of states, with possibly different kind of types.
    Learn how they can be useful in various cases.

## Helper features
1. <a href="References.md">Reference</a>

    Reference in Nadya means assigning different name to same value. Using them without consideration can cause complicated code, but sometimes,
    they can be used to squeeze out optimal performance. If it is used with caution, it can be very useful. Learn how to use them in proper & efficient way
    without polluting readability, and conciseness of your code.

2. <a href="Attributes.md">Attributes</a>
    
    Attributes can attach additional information to Nadya code when necessary. They are typically used for optimization information, or special traits when necessary.
    Learn which types of attributes are available in Nadya, and how they can be used.

3. <a href="Compiler-builtins.md">Compiler builtin functions</a>
    
    Nadya has some compiler builtin functions that can call hardware instructions directly. In most cases, this feature is not necessary. However, these can be useful when
    programmer wants to guarantee that special hardware intrinsics are used.
