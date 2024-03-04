# Introduction to Nadya

Nadya is the language for writing high performance applications quick and easy, 
without knowing too much about hardware optimization techniques.

Its was primarily designed to write ML layers in easy & extensible way. However,
it can also be used for any other general usages.

__Design purposes__
1. Easy & slow learning curve for starters writing HPC programs
2. Complete & stand-alone
3. Intelligent optimization pipeline
4. Fully customizable

### Why Nadya?
__Functional-style programming pattern__

Functional programming has many advantages.
It is concise, clear, and prevents mistakes effectively
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


__Straight forward arithmetic operations__

Nadya has built-in support for tensor arithmetics, with broadcasting semantics
Nadya will generate fast code automatically for every cases
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
        let ref refA = a
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
