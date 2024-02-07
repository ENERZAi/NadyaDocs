# FAQ

## Opus Syntax

### 1. compile time list to runtime tuple & tuple type

`opus` support `rtTuple` and `rtTupleType`. You can use this feature directly like below.

```C#
template </ tupType : rtType />
fun func(first : i32 * u32) -> tupType {
    ...
} 

let a = rtTupleType()
let b = append(a, rtType(i32))
let c = rtTuple()
let d = append(c, !{2})
let e = append(d, !{1u32}) 

!{ func</ b />(${e}) }
```

In core library, `core.List.List` support `toRtTuple` and `toRtTupleType` You can just use these closures.

```C#
module core.List.List as list

template </ tupType : rtType />
fun func(first : i32 * u32) -> tupType {
    ...
} 

let a = [rtType(i32); rtType(u32); rtTensorType(rtType(i32), 3)]
let b = list.toRtTupleType <| a
let c = [!{2}, !{1u32}]

!{ func</ b />(${c}) }
```

## Compilation

### X86 target

#### 1. unknown reference `__extendhfsf2` `__truncsfhf2`

These functions are from `libgcc.a`, but if the version is too low, it may not be implemented. Upgrade libgcc version or
use clang-rt by option `--rtlib=clang-rt`.

### ARM target

#### 1. fp16 operations are slower than expected

If feature `+fp16` is not given in `-march` option, it could not be lowered to fp16 intrinsics. Be careful
**What is not** has the meaning of **no**.

