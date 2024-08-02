# Load &amp; Store semantics

### Tensor load
Loading from tensor copies values from the original tensor, and creates new tensor that might have different shape and mutability.
When accessing tensor with load expression, there can be two types of index values. They have different behavior when determining shape of the tensor.
1. i32 type
   Accesses one index from the corresponding dimension. Dimension with i32 type is squeezed, and will not be visible in output tensor.
2. range type
   Accesses range of values from the corresponding dimension. Dimension with range type is appended to output shape, even if range has size of one.

When all index values are i32 type, resulting type is scalar value.

For example,
<code-block lang='c#'>
let src = tensor((2,3), {1,2,3,4,5,6})
let loadedScalar = src[(1,2)] // Scalar value (6)
let loadedTensor1 = src[(0, 0:3:1)] // tensor shaped (3,). First dimension as been squeezed out
let loadedTensor2 = src[(0:1:1, 0:3:1)] // tensor shaped (1,3). First dimension is alive since it was given as range
</code-block>

### Tensor store
Storing to tensor updates the destination tensor using the given value to store. Storing tensor does not modify source tensor at all, but modifies destination tensor.

<code-block lang='c#'>
let mut dst = tensor((2,3), {1,2,3,4,5,6})
let src = tensor((1,2,1), {7,8})
// Case 1
dst[(0,0)] &lt;- 10 // dst is updated to {10,2,3,4,5,6}
// Case 2
let range = (0:2:1,1)
dst[(0:2:1,1)] &lt;- src // dst is updated to {1,7,3,4,8,6}
// Case 3
dst[(0,0:2:1)] &lt;- src // dst is updated to {7,8,3,4,5,6}
// Case 4
dst &lt;- src // Error! : if src and dst must have same shape
let src2 = tensor((2,3), {11,12,13,14,15,16})
dst &lt;- src2 // OK : dst and src2 has same shape
</code-block>

When all store index values are i32 type, only integer can be stored to tensor and only one value can be updated. (See case 1 in the example)

If at least one store index value is range, then store operation expects right hand side (toStore) to be tensor with compatible shape. 'Compatible' here means shape of the destination pointed by indices and shape of tensor to store has identical shape when they are squeezed. (When all '1's in the shape has been removed).
This makes accessing indices in store operation very flexible.
Case 2 and Case 3 shows such examples.
source tensor shape of (2,) while store region of destination tensor has shape of (2,1) and (1,2) each. They are both compatible since source tensor and store region has shape of (2,) when squeezed.
Same holds with variation of source shape.

#### Storing without specifying range

Tensors can be stored directly without range or index. 
In this case, interpreter expects source and destination shapes to be identical (same rank, same dimensions).
<code-block lang='c#'>
let mut tensorA = tensor((2,3), {1,2,3,4,5,6})
let tensorB = tensor((2,3), {6,7,8,9,10,11})
tensorA &lt;- tensorB // OK tensor can be updated
print(tensorA) // Prints tensor((2,3), {6,7,8,9,10,11})
</code-block>

Updating tensor with reference can be done as follows
<code-block lang='c#'>
let mut tensorA = tensor((2,3), {1,2,3,4,5,6})
let mut tensorB = &tensorA
tensorB &lt;- tensor((2,3), {6,7,8,9,10,11}) // OK tensor can be updated
print(tensorA) // Prints tensor((2,3), {6,7,8,9,10,11})
</code-block>