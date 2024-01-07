# Algorithm

Algorithm module implements some common & useful algorithms

## Components
### Bisect
#### lowerBound tens find
Calculates lower bound (first value equal or greater than input) index of the 1D sorted tensor 

__Example__
```c#
module core.Algorithm.Bisect as bisect
let tens = tensor((5,), {0,2,5,7,10})
let ans = bisect.lowerBound tens 5
print(ans) // Prints 2
```
__Program output__
```Bash
2
```

#### upperbound tens find
Calculates upper bound (first value greater than input) index of the 1D sorted tensor.
If all elements of '_tens_' is greater than '_find_', prints -1

```c#
module core.Algorithm.Bisect as bisect
let tens = tensor((5,), {0,2,5,7,10})
let ans = bisect.upperBound tens 5
print(ans) // Prints 3
```
__Program output__
```Bash
3
```
