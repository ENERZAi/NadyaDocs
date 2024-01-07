# Basics

This describes collection of basic building blocks for creating opus program. For each section,
link to detailed description is stated below.

## Modules {collapsible="true"}
Modules defined from another file, or core library module can be imported by 'module' keyword 
at the beginning of the source file.

Module allows programmers to reuse repeated implementations, separate source files
and opens way to develop structured program using Opus.

<code-block lang="c#">
module core.List.List as list // import module 'list'
// .. some implementation
let myList = [1;2;3]
let sortedList = list.sort myList
print(myList) // output: Op_Cons (1,Op_Cons (2,Op_Cons (3,Op_Empty )))
</code-block>

__Program output__
```
Op_Cons (1,Op_Cons (2,Op_Cons (3,Op_Empty )))
```


## Functions {collapsible="true"}
Functions in opus can be defined using following syntax

1. Function definition without template
```
fun <function name> (<arg0>  : <type0>, <arg1> : <type1>, ...) -> <return type> {
    // function body
}
```

2. Function with template

template arguments can be used to 'specialize' the function in the way that programmer
wants. It can be evaluated statically by the compiler and it would generate
binary in the way that programmer wants.
```
template </<targ0>, <targ1>, ... /> 
fun <function name> (<arg0>  : <type0>, <arg1> : <type1>, ...) -> <return type> {
    // function body
}
```

Here are real-world use case for defining simple function without any template specializations

<tabs>
    <tab title = "Simple add">
        <code-block lang="c#">
        // Example 1 : Simple add operation
        fun addOp(inputA : rtType(tensor&lt;f32, 4&gt;),inputB : rtType(tensor&lt;f32, 4&gt;)) -> rtType(tensor&lt;f32, 4&gt;) {
            inputA + inputB
        }
        </code-block>
    </tab>
    <tab title = "Op agnostic binary operation">
        <code-block lang="c#">
        // Example 2 : Binary operation that can vary the type of the operation
        template &lt;/opType/&gt;
        fun binaryOp(inputA : rtType(tensor&lt;f32, 4&gt;),inputB : rtType(tensor&lt;f32, 4&gt;)) -> rtType(tensor&lt;f32, 4&gt;) {
            ${ 
                if(opType == "add") {
                    !{
                        tensorA + tensorB
                    }ted
                } else if(opType == "sub") {
                    !{
                        tensorA - tensorB
                    }
                } else if(opType == "mul"){
                    !{
                        tensorA * tensorB
                    }
                } else if (opType == "div"){
                    !{
                        tensorA / tensorB
                    }
                }
            }
        }
        </code-block>
    </tab>
    <tab title = "Op and data type agnostic binary operation">
        <code-block lang="c#">
        // Example 3 : Binary operation that can vary data type and type of the operation
        template &lt;/tensorType, opType/&gt;
        fun binaryOpWithArbitraryDataType(inputA : tensorType),inputB : tensorType) -> tensorType {
            ${ 
                if(opType == "add") {
                    !{
                        tensorA + tensorB
                    }ted
                } else if(opType == "sub") {
                    !{
                        tensorA - tensorB
                    }
                } else if(opType == "mul"){
                    !{
                        tensorA * tensorB
                    }
                } else if (opType == "div"){
                    !{
                        tensorA / tensorB
                    }
                }
            }
        }
        </code-block>  
    </tab>
</tabs>

## Printing to standard output {collapsible="true"}
```print``` can print argument to standard output.
It automatically appends newline character at the end of the string
<code-block lang='c#'>
print("hello world") // print simple string
let number = 3.14f
print("The value of pi is " + toStr(number)) // use 'toStr' to convert value to string
print(number) // print the number itself
</code-block>

__Program output__
```
hello world
The value of pi is 3.140000
3.140000
```

## Values and Variables {collapsible="true"}

Read only values can be defined with keyword let. To define writable variable, use 'let mut'.

Values can be declared as follows
<code-block lang='c#'>
let a = 3 // 32bit integer value
let b = 3.14f // 32bit floating point value
let c = 3.1415f64 // 64bit floating point value
let d = true // boolean value
</code-block>

You can define multiple values at the same time using tuple based definition
<code-block lang='c#'>
// This defines all values within one line
let a, b, c, d = 3, 3.14f, 3.1415f64, true
</code-block>

Assigning new value to value is invalid
<code-block lang='c#'>
// This defines all values within one line
let a = 3
a &lt;- 4 // error : Cannot assign to immutable value
</code-block>

To make mutable variable, use 'mut' keyword
<code-block lang='c#'>
let mut a = 3
print(a) // prints '3'
a &lt;- 4 
print(a) // prints '4'
</code-block>

__Program output__
```
3
4
```

## References {collapsible="true"}
'&' symbol in opus means 'reference'. Reference allows one value to point to contents of another value. 
This lets programmers to give different name to same data. Modification to the referenced value
will affect all symbols pointing to modified data.

Note only l-value (which means value has been already assigned to some symbol) can be referenced.
r-value (immediate value that hasn't been assigned to any other symbol) cannot be referenced.

<code-block lang='c#'>
let mut value = 3
let mut valueRef1 = &value
print(value) // prints '3'
valueRef1 &lt;-4
print(value) // prints '4'
</code-block>

__Program output__
```
3
4
```

Assigning to r-value is invalid
<code-block lang='c#'>
let mut valueRef = &(value + 3) // error: Can only reference from identifier or load expression
</code-block>

__Program output__
```Bash
./example.opus:30:23: error: Can only reference from identifier or load expression
let mut valueRef = &(value + 3)
                      ^
Opus project compilation failed. You got 1 error
```

Note that immutable values cannot be bound as reference to mutable let binding and vice-versa.
For more information on relation between mutability and reference, please refer to 
<a href="Mutability-and-reference.md">Mutability & Reference</a>

## Tensor {collapsible="true"}
Tensor is used to describe chunk of numeric data with same type.
Tensor is composed 3 properties. Shape, data type, and actual data.
Tensors can be used for complex numeric operations, references,

### Defining Tensors

Read-only tensors can be defined with 'let' keyword. When defining mutable tensor, put 'mut' keyword after 'let'.
(Exactly same as other kind of values)
<code-block lang='c#'>
let myIntegerTensor = tensor((2,3), {1,2,3,4,5,6}) // Create 32bit constant integer tensor shaped (2,3)
print(myIntegerTensor) // Prints 'tensor((2,3),{1,2,3,4,5,6})'
let myFloatTensor = tensor((3,2), {1.0f, 2.0f, 3.0f, 4.0f, 5.0f, 6.0f}) // Create 32bit constant floating point tensor shaped (3,2)
print(myFloatTensor) // Prints 'tensor((3,2,),{1.000000,2.000000,3.000000,4.000000,5.000000,6.000000})'
let mut mutableTensor = tensor((3,), {1i16,2i16,3i16}) // Create 16bit mutable integer tensor shaped (3,)
print(mutableTensor)
</code-block>

__Program output__
```Bash
tensor((2,3),{1,2,3,4,5,6})
tensor((3,2,),{1.000000,2.000000,3.000000,4.000000,5.000000,6.000000})
tensor((2,3),{1,2,3})
```

### Load & store

To load and store the value from tensor, we need to provide tuple containing i32 typed values

__Loading from tensor__
<code-block lang='c#'>
let myIntegerTensor = tensor((2,3), {1,2,3,4,5,6}) // Create 32bit constant integer tensor shaped (2,3)
print(myIntegerTensor[(0, 1)]) // Prints 2
print(myIntegerTensor[(0:1:1, 0:2:1)]) // Prints tensor((1,2),{1,2})
</code-block>

__Program output__
```Bash
2
tensor((1,2),{1,2})
```

__Storing to tensor__

When storing to tensor, there can be 2 cases
1. Storing tensor to tensor
2. String scalar to tensor

When storing tensor to tensor, the squeezed shape of stored tensor
must equal shape of tensor to store. 

(Where 'squeeze' means eliminating all dimension with single element i.e. (1,5,1,3) is squeezed to (5,3))

<code-block lang='c#'>
let mut myTensor = tensor((2,3), {1,2,3,4,5,6}) // Create 32bit constant integer tensor shaped (2,3)
myTensor[(0, 1)] &lt;- 10 // Stores scalar to tensor
print(myTensor) // Prints tensor((2,3),{1,10,3,4,5,6})
let other = tensor((2,), {10,11,12})
// Stores tensor to tensor
myTensor[(1:2:1, 0:2:1)] &lt;- other // Ok. Squeezed shape of other (2,) matches squeezed shape of update range (2,)
print(myTensor) // Prints tensor((2,3),{1,10,3,10,11,6})
</code-block>

__Program output__
```Bash
tensor((2,3),{1,10,3,4,5,6})
tensor((2,3),{1,10,3,10,11,6})
```

For more information, please refer to 
<a href="Load-Store-rules.md">Load & Store semantics</a>

## Closures (Lambda functions) { collapsible="true" }
<note>
This feature is currently only available in compile time code
</note>

Closures are special kind of functions that is treated as an 'value'.
It captures state of the program when it was instantiated, and uses it when closure is called.

Closure can be created as simple _let_ keyword.
```BNF
'let' <symbol_name> <param_name>+ '=' <expression>
```
Or, it can be created as r-value with _lambda_ keyword
```BNF
'lambda' '(' <param_name> '(' ',' <param_name>* ')' '{' <expression> '}'
```

When calling closure, use pipeline operator '<|' to pass each argument. 
You can also can omit '<|' and separate arguments with whitespace as syntactic sugar.

<code-block lang='c#'>
let myFun1 a b = a + b
let result1 = myFun1 2 3 // You can also call it by myFun1 &lt;| 2 &lt;| 3
print(result1) // prints 5
let a = 2
// myFun2 automatically captures 'a'
let myFun2 b = a + b // equivalent to 2 + b
let result2 = myFun2 3
print(result2) // prints 5
// You can also create closure as r-value (right hand side value)
print(lambda(a, b) { a + b } &lt;| 2 &lt;| 3) // Prints 5
</code-block>

__Program output__
```Bash
5
5
5
```

Closures can be partially instantiated, and can be carried around

<code-block lang='c#'>
let myFun a b = a + b
let partialMyFun = myFun &lt;| 3
print(myFun) // Prints lambda(a) { lambda(b) { (a + b ) }}
let res = partialMyFun &lt;| 2
print(res) // Prints 5
</code-block>

__Program output__
```Bash
lambda(a){
lambda(b) { (a + b) }}
5
```

