# Modules

In Nadya, _module_ is group of code, which may include function & value definitions, declarations. Module allows
grouping them in different purposes that supports different functionalities. This helps to avoid conflicts in the program
and improve code readability.

## Contents
1. <a anchor="declaring-module"> Declaring module </a>
2. <a anchor="calling-module"> Calling module </a>

## Declaring Module

Module is automatically declared & defined when you build '.ndy' file. Single '.ndy' file corresponds to single module.
Name of the file becomes name of the module. For instance, when you create file 'foo.ndy', you are declaring module named 'foo'.

Each module can be imported by another module with given module name. It is also used to identify project hierarchy in 
_nadya-project.yml_ file in 
<a href="Project-configuration.md">__Project configuration__</a>

Following file named 'fibonacci.ndy' declares module 'fibonacci'. which named function named 'fib'.
Value (closure) 'fib' would be able to be called by other modules when they import 'fibonacci'.

__fibonacci.ndy__
```c#
// File : fibonacci.ndy
let rec fib n = 
    if(n == 0){
        0
    } else if (n == 1){
        1
    }else {
        fib (n-1) + fib(n-2)
    }
    
print(fib <| 10) // Prints '55' to stdout

```

## Calling Module

Calling module can be done with ease by simply using _module_ keyword. _module_ keyword is located at top of the file for 
loading modules required to implement the rest of the program. 

<note>Only modules in higher hierarchy in nadya-project.yml file can be imported to avoid cyclic module dependency. See <a href="Project-configuration.md">Project configuration</a>
 for more information.</note>

For example, 'fibonacci.ndy' can be imported in other file as follows.

```c#
// File : main.ndy
module fibonacci

let fibonacci_10 = fibonacci.fib <| 10
print(fibonacci_10) // Prints '55'
```

Modules can be organized in directories. If some module is under the directory, its relative path from the project root to module 
needs to be specified when it is imported.

In Nadya, directory where _nadya-project.yml_ exists becomes project root. Therefore, you need to write path relative to location of the module from
the path where _nadya-project.yml_ exists.

Accessing values & functions declared in modules can be done by writing following syntax.

```BNF
<path to the module from project root> `.` <name of function or value>
```

If you find accessing value in different module with import path is too verbose, you can either import module using alias.

```c#
// File : main.ndy
module fibonacci as f // Import fibonacci with alias named 'f'. Now, module 'fibonacci' can be called with 'f'.

let fibonacci_10 = f.fib <| 10
print(fibonacci_10) // Prints '55'
```


For instance, if directory looks as follows,
module _alpha_ and _beta_ are required to be called after 'example' when importing module from _entry.ndy_.

__Project structure (example)__
```
root
├── example
│   ├── alpha.ndy
│   └── beta.ndy
├── entry.ndy
└── nadya_project.yml
```

alpha and beta can be called from entry as follows. When accessing the value defined in _Example.alpha_ and _Example.beta_, 
you need to specify name of the value or function after _Example.alpha_ and _Example.beta_ respectively, or name of the alas if you
imported modules using alias.

```c#
// File : entry.ndy
module Example.alpha
module Example.beta


// access member named 'value' in Example.alpha
let alpha_value = Example.alpha.value
// access member named 'value' in Example.beta
let beta_value = Example.beta.value

// Call first-class function defined in 'Example.alpha' named 'foo'.
!{
    let argument = 10.0f
    Example.alpha.foo(argument)
}
```

When calling 'alpha' from 'beta', you do not need to specify 'example' in its import path since path is relative from the root path.

```c#
// File : beta.ndy
module Example.alpha

// access member named 'value' in alpha
let alpha_value = Example.alpha.value
```

__Attaching alias__

If alias is attached, you can omit verbose expression pointing to relative path of the module when accessing values.
This is useful when path to the module is complicated

```c#
// File : entry.ndy
module Example.alpha as a
module Example.beta as b


// access member named 'value' in Example.alpha
let alpha_value = a.value
// access member named 'value' in Example.beta
let beta_value = bvalue

// Call first-class function defined in 'Example.alpha' named 'foo'.
!{
    let argument = 10.0f
    a.foo(argument)
}
```
