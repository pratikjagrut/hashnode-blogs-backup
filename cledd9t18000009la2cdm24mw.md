# Pointers in Go

The pointer is a special variable in Golang that stores the memory address of other variables.

Variables are used to store some type of data. All variables are assigned a particular memory where they store data and this memory has a memory address that is in hexadecimal format. The number that starts with `0x` is hexadecimal like (`0x14` which is equivalent to 20 in decimal). Golang allows us to store this memory address in variables but only pointers will understand that the stored value is pointing to some memory whereas other variables will treat it just as a value.

## Declaring Pointer

The type `*T` is a pointer to a `T` value. Its zero value is `<nil>`.

```go
var pointer_name *Data_Type
var p *int = &variable
```

In the above snippet, you can see two symbols `*` and `&`, which are two operators.

`* Operator` is also called dereferencing operator which is used to declare an operator and also to access the value stored at the memory address pointer pointing to.

`& Operator` this operator is used to access the memory address of a variable.

Below is an example demoing the use of \* and &.

```go
package main

import "fmt"

func main() {
  i := 42

  var p *int = &i

  fmt.Println("The value of i is: ", *p)
  fmt.Println("The memory address of i is", p)
}
```

```go
The value of i is:  42
The memory address of i is 0xc00002c008
```

[***Run this code in Go Playground***](https://play.golang.org/p/DRmvVzN-gFK)

### Shorthand Declaration

In Golang shorthand declaration(:=) is used to narrow down the variable declaration. The Golang compiler will decide if the RHS variable is a pointer variable if we're assigning the memory address of the LHS variable using `&`.

```go
i := 42
p := &i
```

### Declaring pointer with the new function

Golang has a built-in allocating primitive function called `new`. The `new(T)` allocates memory, initializes it with a zero value of type T and returns the pointer to that memory. In Go terminology, it returns a pointer to a newly allocated zero value of type T.

**What is the difference between** `var p *int` and `p := new(int)`?

The expression `var p *int` only declares the pointer and does not initialize it with any memory address, i.e pointer does not point to any memory location. Whereas in `new(T)` function returns the pointer pointing to the memory location in the system.

```go
package main

import "fmt"

func main() {
  var p *int
  fmt.Println("The value of p", p)

  p1 := new(int)
  fmt.Println("The value of *p1 is: ", *p1)
  fmt.Println("The value of p1", p1)
}
```

```go
The value of p <nil>
The value of *p1 is:  0
The value of p1 0xc00002c008
```

In the above example, we can not dereference the pointer `p` because it has `<nil>` value. Trying to dereference it will throw an error.

```go
panic: runtime error: invalid memory address or nil pointer dereference
```

[***Run this code in Go Playground***](https://play.golang.org/p/C8gmExWxJ5B)

## Passing a pointer to the function

We can pass a pointer to the function as we pass other variables. We can create a pointer to the variable and then pass it to the function or we can just pass an address of that variable using `&`.

In the below example, we initialize a variable i with the value 10. We have a function `addTen(i *int)` which takes a pointer as an argument and then adds 10 by dereferencing the pointer hence mutating the original value.

```go
package main

import "fmt"

func addTen(i *int) {
  *i = *i + 10
}

func main() {
  i := 10
  p := &i
  fmt.Println(i)
  addTen(p) // Pass a pointer
  fmt.Println(i)
  addTen(&i) // Pass address using & operator
  fmt.Println(i)
}
```

```go
10
20
30
```

[***Run this code in Go Playground***](https://play.golang.org/p/gz-qxg5IT3Q)

## Pass-by-value vs Pass-by-pointer

When we write a function we decide parameters to be passed `by-value` or `by-pointer`.

**Pass by value**

Every time variable is passed as a parameter a new copy of that variable is created and passed to the function. This copy has a different memory location hence any changes to this copy will not affect the original variable.

In the below example, we pass the parameter by value.

```go
package main

import "fmt"

func add(i int) {
  i = i + 10
  fmt.Println("Variable copy from add function:", i)
}

func main() {
  i := 10
  fmt.Println("Original variable:", i)
  add(i)
  fmt.Println("Original variable:", i)
}
```

```go
Original variable: 10
Variable copy from add function: 20
Original variable: 10
```

[***Run this code in Go Playground***](https://play.golang.org/p/uSHAuPe4dEn)

**Pass by Pointer**

When we pass the parameter by-pointer Go makes a copy of that pointer to the new location. This pointer copy has the same memory address stored in it hence it is pointing to the original variable and any changes done using this pointer will change the value of the original variable.

```go
package main

import "fmt"

func add(i *int) {
  *i = *i + 10
  fmt.Println("Variable copy from add function:", *i)
}

func main() {
  i := 10
  fmt.Println("Original variable:", i)
  add(&i)
  fmt.Println("Original variable:", i)
}
```

```go
Original variable: 10
Variable copy from add function: 20
Original variable: 20
```

[***Run this code in Go Playground***](https://play.golang.org/p/ojMMRRkIeo3)

*In some way, we can say that pass-by-pointer is an implementation of pass-by-value.*

## Container types(slices, maps etc) and pointers

```go
package main

import (
	"fmt"
)

func modify(h map[int]int) {
	h[1] = 5
	fmt.Println("Modified map: ", h)

}

func main() {
	m := make(map[int]int)
	m[1] = 1
	m[2] = 2
	m[3] = 3
	fmt.Println("Original map: ", m)
	modify(m)
	fmt.Println("Original map: ", m)
}
```

In the above example, we're creating a map and passing it to the modify function. With the cursory look, it seems like we're passing an argument by-value because we're not passing the address of map m using & or by creating a pointer to map m. So this means the copy of map m should be created and the original map is untouched. But if we run this we get an output as follows.

```go
Original map:  map[1:1 2:2 3:3]
Modified map:  map[1:5 2:2 3:3]
Original map:  map[1:5 2:2 3:3]
```

[***Run this code in Go Playground***](https://play.golang.org/p/ORs5UkdECmS)

***We passed an argument by value then why did the original map was updated?***

When we create a map using `m := make(map[int]int)` the compiler makes a call to the `runtime.makemap` a function whose signature is as `func makemap(t *maptype, hint int, h *hmap) *hmap`.

So as we see, the return type of `runtime.makemap` is a pointer to the `runtime.hmap structure`. So when we passed a map to the function we actually passed a pointer to the `runtime.hmap` structure of map m and hence the original map was modified.

Instead of mapping if we try the above program using slice we will get a similar output because the slice variable stores the pointer to an underlying array.

***Thank you for reading this blog, and please give your feedback in the comment section below.***