#  Unions

Unions provide support for values that can be one or more kind of format. Each format can have different types and values.
This is useful for cases where single value is required to pack multiple kind of states, with possibly different kind of types.

#### Syntax

```BNF
`type` <name> `=` ( '|' <name> 'of' <type> )+
```


#### Example
```Haskell
type Option<a> = 
    | Some of a
    | None
```

<note>  
This feature is yet to come in Nadya native
</note>


Union definition


## Defining Union
<note> Union only works for compile time code in current version </note>

Unions provide support for value that can have multiple structures, including different types and names.
This feature can be used where it is required for single value to have possibly multiple kinds of structures.


```Haskell
// Declare union
type Figure =
    | Circle of f32         // Circle only needs single value 'radius'
    | Rectangle of f32*f32  // Rectangles needs two values 'height' and 'width'
    | Sphere of f32         // Sphere only needs single value 'radius'
    | Cube of f32*f32*f32   // Cube needs three values 'height', 'width' and 'depth'
    
// Construct union values
let circle = Circle 3.0f // Define circle with radius '3'
let rectangle = Rectangle (2.0f, 3.0f) // Define rectangle with height 2.0f, and width 3.0f
let sphere = Sphere 3.0f // Define sphere with radius '3'
let cube = Cube (2.0f, 3.0f, 4.0f) // Define cube with height 2.0f, width 3.0f and depth 4.0f

// Results
print(circle) // Prints Circle 3.000000
print(rectangle) // Prints Rectangle (2.000000,3.000000)
print(sphere) // Prints Sphere 3.000000
print(cube) // Prints Cube (2.000000,3.000000,4.000000)
```

__Program output__
```Bash
Circle 3.000000
Rectangle (2.000000,3.000000)
Sphere 3.000000
Cube (2.000000,3.000000,4.000000)
```

## Pattern matching
Pattern matching is procedure that determines which kind of value that union value has, and extracting the value inside the union value.
Think of union value has certain kind of 'pattern' (which is kind of structure that union value has) and 'pattern matching' as
identifying the actual pattern that union value has.

```Haskell
// Declare union
type Figure =
    | Circle of f32         // Circle only needs single value 'radius'
    | Rectangle of f32*f32  // Rectangles needs two values 'height' and 'width'
    | Sphere of f32         // Sphere only needs single value 'radius'
    | Cube of f32*f32*f32   // Cube needs three values 'height', 'width' and 'depth'
    
// Construct union values
let circle = Circle 3.0f // Define circle with radius '3'
let rectangle = Rectangle (2.0f, 3.0f)
let sphere = Sphere 3.0f
let cube = Cube (2.0f, 3.0f, 4.0f)

// Matching patterns
// This function returns size of area (if figure is 2D) or volume (if figure is 3D)
let pi = 3.1415f
let getAreaOrVolume x = 
    match x with
    | Circle a -> pi*a*a
    | Rectangle (a, b) -> a*b
    | Sphere a -> (4.0f/3.0f)*pi*a*a*a
    | Cube (a,b,c) -> a*b*c
    
// Accumulating results
print("Area of circle : " + toStr(getAreaOrVolume circle)) // Prints 28.273500
print("Area of rectangle : " + toStr(getAreaOrVolume rectangle)) // Prints 6.000000
print("Volume of sphere : " + toStr(getAreaOrVolume sphere)) // Prints 113.093994
print("Volume of cube : " + toStr(getAreaOrVolume cube)) // Prints 24.000000
```



__Program output__
```Bash
Area of circle : 28.273499
Area of rectangle : 6.000000
Volume of sphere : 113.093994
Volume of cube : 24.000000
```

Patten matching can be used instead 'if/else' statement.
Pattern can be a value literal, and '_' indicates any other case.

Two implementations are equivalent.
```Haskell
let a = 30
if(a == 30){
    print("true")
} else {
    print("false")
}
```

```Haskell
let a = 30
match a with
| 30 -> print("true")
| _ -> print("false")
```