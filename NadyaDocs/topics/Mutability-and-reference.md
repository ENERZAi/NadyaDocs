# Mutability and reference

## Mutability
Mutability allows programmers to mutate values after it was initialized. Using this property, users can extend their code easier than before.

To minimize potential human-errors caused by mutable values, nadya treats all values as immutable unless value was initialized with 'let mut'.

If programmer attempts to mutate any immutable value, interpreter will report ```err_store_to_immutable_value``` error.

<code-block lang='c#'>
let immutableValue = 1 // This value is immutable.
immutableValue &lt;- 3 // Error! emits : "Cannot assign to immutable value "1""
let mut mutableValue = 1 // This value is mutable.
mutableValue &lt;- 3 // Ok.
print(mutableValue) // Prints '3'
</code-block>

### Assigning the value
Any value used in nadya can be mutable and they can be assigned if they are mutable. But, there is one thing that must be kept in mind.
**Destination value and original value must have compatible type.**
Nadya cannot assign to value that has different type (for example, we can't assign string to integer).

<code-block lang='c#'>
let mut dstValue = (3, 5)
dstValue &lt;- (3, 4, "str") // Error! We cannot assign value typed (i32, i32, string) to value typed (i32, i32)
dstValue &lt;- (1, 2) // Ok. Now, dstValue will be (1, 2)
</code-block>

## Reference
Reference allows one value to point to contents of another value. This lets programmers to give different name, create sub-tensor without copying the value.

In Nadya values can be divided into two categories. l-value and r-value. l-value are values stored in the environment while r-value are values that are temporary and can be destructed afterwards by garbage collector.

To reference some value, we must reference l-values. Therefore, interpreter must check whether value given inside is l-value or r-value.

<code-block lang='c#'>
let valueRef1 = &value // OK since value is l-value
let valueRef2 = &(value + 3) // Error since reference target was r-value
</code-block>
Reference is especially useful when it is used with tensors. Same rule is applied for tensor as a value. We can build tensor reference using subset of the original tensor, and treat it just like any other tensors. When tensor reference is modified (and if it was mutable), corresponding contents of the original tensor will be modified as well.

<code-block lang='c#'>
let tensorRef1 = &tensor // reference of the tensor
let tensorRef2 = &tensor[(3, 5)] // Tensor shaped (1, 1) pointing to tensor[3,5]
let tensorRef3 = &tensor[(2:4:1, 5)] // Tensor shaped (2, 1) pointng to the given region
</code-block>

### Mutability and reference
There are several rules to be stated between handling reference with mutable variables.

**Restrictions**
1. Immutable values cannot be bound as reference to mutable let binding.
    1. Binding immutable value to mutable let binding without reference is allowed with deep copy
    2. Binding to mutable let without reference always deep-copies the value
    3. Binding to immutable let with reference is allowed
2. Mutable values cannot be bound as reference to immutable let binding
    1. Binding mutable value to immutable let binding without reference is allowed with deep copy
    2. Binding to immutable let without reference deep-copies the value only if init value is mutable
3. Reference can only be attached to tensor or numeric values (all integers, all floating points and boolean) which implements 'ReferenceInterface'. Otherwise, interpreter reports ```err_reference_to_non_numeric_value``` exception.

__Examples__
<code-block lang='c#'>
let mut value = 1 // mutable value
let constValue = 2 // constant value
let constValueCopy1 = value // OK. Allowed by deep copy
let constValueCopy2 = constValue // OK. Allowed by shallow copy (much cheaper to compute)
let mut valueCopy1 = value // OK. Allowed by deep copy
let mut valueCopy2 = constValue // OK. Allowed by deep copy
let constValueRef1 = &value // Error! Cannot bind mutable reference to immutable let binding
let constValueRef2 = &constValue // OK. no copy required.
let mut mutableValueRef1 = &value // OK. Allowed by reference (no copy, value and mutableValueRef points to same object)
let mut mutableValurRef2 = &constValue // Error! Cannot bind const value to mutable value as reference
let mut stringValue = "some string"
let mut stringRef = &stringValue // Error! cannot create reference from non-numeric typed values
</code-block>

### Passing reference to closure & constructors
When reference is passed to closure & constructors, value will be deep-copied, and passed argument won't be reference anymore.
If passed argument was mutable, it will be set to 'constant'.

<code-block lang='c#'>
let foo x = x + 1
let mut arg = 3
let mut argRef = &arg
foo &lt;| argRef // This will deep copy the argRef, decoupling with 'arg'
</code-block>


## Passing arguments to function parameters
In nadya, values can be passed to functions in multiple ways. This contains passing values as mutable or constant, or reference or non-reference.
Following is the rule for matching function argument to parameters.

**Rules**
* Parameter is immutable value
    * Can accept both immutable and mutable values
    * Always pass argument without copy
* Parameter is mutable value
    * Can accept both immutable and mutable values
    * Always pass argument with copy
* Parameter is mutable reference
    * Only accept if argument is mutable reference & argument is not copied


<code-block lang='c#'>
// immutable parameter
fun foo(param : tensor&lt;i32>) -> i32 {
	// Do something
	0
}
// mutable parameter
fun bar(mut param : tensor&lt;i32>) -> i32 {
	// Do something
	0
}
// mutable reference parameter
fun goo(mut &param : tensor&lt;i32>) -> i32  {
	// Do something
	0
}
let immutableTensor = tensor({1,2,3}, (3,))
let mut mutableTensor = tensor({1,2,3}, (3,))
// function foo
foo(immutableTensor) // OK : argument and parameter uses same memory
foo(mutableTensor) // OK : argument and parameter uses same memory
foo(&immutableTensor) // Error! : function does not accept reference
foo(&mutableTensor) // Error! : function does not accept reference
// function bar
bar(immutableTensor) // OK : copy required (separates memory between argument and parameter)
bar(mutableTensor) // OK : copy required
bar(&immutableTensor) // Error! : function does not accept reference
bar(&mutableTensor) // Error! : function does not accept reference
// function goo
goo(immutableTensor) // Error! : function only accepts mutable reference
goo(mutableTensor) // Error! : function only accepts mutable reference
goo(&immutableTensor) // Error! : function only accepts mutable reference
goo(&mutableTensor) // OK : argument and parameter uses same memory
</code-block>
