# Variables, Data Types And Constants in Go

## Variable

A variable is a symbolic name given to the storage location which contains some value that can be changed at any time during the execution of the program. A variable must be defined with the type of data or value it is holding.

## Data types

There are several data types in Go.

```go
bool        int         uint        float32     complex64
string      int8        uint8       float64     complex128
byte        int16       uint16
rune        int32       uint32
error       int64       uint64
```

## Zero value

In some programming languages variable holds a `null` or `undefined` value when not initialized, Go gives it a zero value of its data type. A `boolean` variable if not initialized, get the `false` value and an `integer` variable gets `0` value, `string` variable will get `nil` value.

## Declaring a variable

The `var` the statement declares a list of variables. The type is at the last of the statement.

```go
var varName dataType
```

E.g.:

```go
var i, j, k int
var f float32
var u uint
```

**Variables with initializers**

We can initialize variables while declaring.

```go
var varName dataType = value
```

```go
var i int = 10
var s string = "hello"
var ok bool = true
```

*We can omit the type of variable, it will take the type of initializers*

```go
var i = 42           // int
var f = 3.142        // float64
var g = "hello"      // string
```

**Short-hand declaration**

This is the most commonly used variable declaration inside functions.

```go
varName := value
```

E.g.:

```go
i := 10
s := "hello"
ok := true
```

*When declaring a variable without specifying it’s type the variable’s type is inferred from the value on the right-hand side.*

```go
var i int
j := i            // j is an int
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

**Assign or Re-assign**

You can assign or re-assign value to a declared variable as:

```go
varName = value
```

E.g.:

```go
i = 12
s = "world"
ok = false
```

**Multiple variable declarations**

Multiple variables of the same type with the single var statement

```go
var i, j, k int
var x, y, z = 0.867 + 0.5i, 4.65, true
```

Multiple variables of the different types with the single var statement

```go
var (
    i int = 10
    s string = "hello"
    ok bool = true
)
```

## Constants

Go supports constants of character, string, boolean, and numeric values. Constants are declared using the `const` keyword.

E.g:

```go
const str string = "constants"
```

**Untyped and Typed**

Constants can be declared with or without a type in Go. If we are declaring a literal constant, then we are declaring constants that are untyped and unnamed.

E.g.:

```go
const str string = "constants" //Typed constant
const i = 10 //Untyped constant, literal declaration
const f = 3.14
```

The constants on the LHS of the declaration are named constants and the literal values on the RHS are unnamed constants.

Typed constants don’t use the same type system as variables, they've their implementation for representing the values that we associate with them.

In the case of a typed constant, the declared type is used to associate the type’s precision limitations or kind of constant.

In the case of an untyped constant, the literal value will determine what kind/type the constant takes

Every GO compiler has the flexibility to implement constants as they wish, within the mandatory set of [requirements](http://golang.org/ref/spec#Constants).

***Thank you for reading this blog, and please give your feedback in the comment section below.***