# Arrays And Slices

## Array

The array is a collection of the same type of element in a contiguous memory location. The array has a fixed length i.e the number of elements to be stored is fixed before memory allocation.

### Array Declarations

An array type definition specifies the length and type of element. The array `[n]T` is of length `n` and type `T`.

```go
// This array can hold 3 integers.
var i [3]int
```

Arrays can be indexed in the usual way, to access the *n<sup>th</sup>* element we can do `a[n]`.

```go
i[0] = 1
i[1] = 10
i[2] = 100
```

**Array literal**

```go
j := [3]string{"Go", "is", "fun"}
```

We can make a compiler to compute the length of an array.

```go
j := [...]string{"Go", "is", "fun"}
```

In both cases, the type of `j` is `[3]string`.

```go
package main
import "fmt"
func main() {
var i [3]int
fmt.Println("Values of array, i: ", i)
fmt.Println("Length of array, i: ", len(i))
i[0] = 1
i[1] = 10
i[2] = 100
fmt.Println("Updated values of array, i: ", i)
j := [3]string{"Go", "is", "fun"}
fmt.Println("Values of array, j: ", j)
fmt.Println("Length of array, j: ", len(j))
k := [...]string{"Go", "is", "fun"}
fmt.Println("Values of array, k: ", k)
fmt.Println("Length of array, k: ", len(k))
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/-mGgVI4V03b)

## Slices

Arrays are a bit inflexible, so slices are widely used in GO. Slices are built on arrays giving them more flexibility, power and convenience.

The slice type is `[]T`, specified without length and `T` is the element type.

### Slice formation

A slice literal is like an array literal without the length.

```go
package main
import "fmt"
func main() {
i := []int{1, 2, 3, 4, 5}
fmt.Println(i)
fmt.Println(len(i))
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/yQUe7knUqgC)

#### Creating a slice with make

Slices can be created with the built-in make function; this is how you create dynamically-sized arrays.

```go
func make([]T, len, cap) []T
```

Where,

* \[\]T: Type of underlying array
    
* len: Length of the underlying array
    
* cap: Capacity of the underlying array, cap is optional
    

```go
package main
import "fmt"
func main() {
b := make([]int, 3, 5)
fmt.Println("Length: ", len(b))
fmt.Println("Capacity: ", cap(b))
fmt.Println("Values: ", b)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/UqAJvWdDX0N)

If the capacity argument is omitted then capacity defaults to length.

```go
package main
import "fmt"
func main() {
b := make([]int, 3)
fmt.Println("Length: ", len(b))
fmt.Println("Capacity: ", cap(b))
fmt.Println("Values: ", b)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/FZSQFmrl3nq)

##### Length and capacity

The length of a slice is the number of elements it contains.

The capacity of a slice is the number of elements in the underlying array, counting from the first element in the slice.

Length: `len(s)`

Capacity: `cap(s)`

#### Creating a slice from an array

A slice can be formed by specifying 2 indices, high bound and low bound separated by a colon, `s[low : high]`. This selects a half-open range which includes the first element but excludes the last one.

```go
package main
import "fmt"
func main() {
odds := [5]int{1, 3, 5, 7, 9}
fmt.Println("Length of odds array: ", len(odds))
fmt.Println("Capacity of odds array: ", cap(odds))
fmt.Println("Values of odds array: ", odds)
fmt.Println()
var o []int = odds[1:4]
fmt.Println("Length of o slice: ", len(o))
fmt.Println("Capacity of o slice: ", cap(o))
fmt.Println("Values of o slice: ", o)
fmt.Println()
m := odds[1:4]
fmt.Println("Length of m slice: ", len(m))
fmt.Println("Capacity of m slice: ", cap(m))
fmt.Println("Values of m slice: ", m)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/Z2e8QM8V2E3)

We can skip low or high bounds to use their defaults.  
For low bound default is `0` and for high bound default is `the length of an array`.

`a[:high]`, `a[low:]`

To create a slice of an entire array we can omit both high bound and low bound.

```go
odds := [5]int{1, 3, 5, 7, 9}
o := odds[:]
```

### Slice internals

A slice does not store any data, it just describes a section of an underlying array.

A slice consists of a pointer to the array segment, the length of an array segment and its capacity (the maximum length of the segment).

| ptr (\*Ele) |
| --- |
| Length (int) |
| Capacity (int) |

#### Relation of array and slice

When we create a slice we create a slice variable which stores the pointer, length and capacity of the underlying array. Slicing does not copy the elements, it creates a new slice variable pointing to the original array. Hence changing elements of the slice will make those changes to an underlying array and all other slices which share the same underlying array will be affected.

```go
package main
import "fmt"
func main() {
alphabet := [5]string{"A", "B", "C", "D"}
fmt.Println("Initial values of alphabet: ", alphabet)
i := alphabet[0:3]
fmt.Println("Initial values of i: ", i)
j := alphabet[1:4]
fmt.Println("Initial values of j: ", j)
i[2] = "P" // This changes will affect alphabet, i ,j slices
fmt.Println()
fmt.Println("Updated values of alphabet: ", alphabet)
fmt.Println("Updated values of i: ", i)
fmt.Println("Updated values of j: ", j)
}
```

```go
Initial values of alphabet:  [A B C D ]
Initial values of i:  [A B C]
Initial values of j:  [B C D]
Updated values of alphabet:  [A B P D ]
Updated values of i:  [A B P]
Updated values of j:  [B P D]
```

[***Run this code in Go Playground***](https://play.golang.org/p/iLR-Cs0cQkE)

### Append and copy functions

#### Append

```go
func append(s []T, x ...T) []T
```

The append function appends the data at the end of the slice. If the destination slice has enough capacity then it is re-sliced to accommodate the new element but if capacity is not enough then a new underlying array is created and allocated.

```go
package main
import "fmt"
func main() {
a := make([]int, 1)
fmt.Println("Length: ", len(a))
fmt.Println("Capacity: ", cap(a))
fmt.Println("Values: ", a)
fmt.Println()
a = append(a, 1, 2, 3)
fmt.Println("Length: ", len(a))
fmt.Println("Capacity: ", cap(a))
fmt.Println("Values: ", a)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/wuAsUZnf9kc)

**Appending one slice to another**

```go
package main
import "fmt"
func main() {
a := []string{"a", "b", "c"}
b := []string{"d", "e"}
a = append(a, b...)
fmt.Println("Length: ", len(a))
fmt.Println("Capacity: ", cap(a))
fmt.Println("Values: ", a)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/JEtqp9lsyD7)

#### Copy

Slice can not grow beyond the capacity of the underlying array if it tries to grow will cause a runtime panic error similar to an index out-of-bounds error.

```go
nums := [3]int{1, 2, 3} // array
n := nums[:] // slicing nums array
n[3] = 4 // this will throw runtime error
panic: runtime error: index out of range [3] with length 3
```

So one way to achieve the dynamic array is to create a new bigger slice and copy the content from the old slice to a new slice.

```go
package main

import "fmt"

func main() {
	nums := [3]int{1, 2, 3} // array
	n := nums[:]            // slicing nums array
	fmt.Println("Length: ", len(n))
	fmt.Println("Capacity: ", cap(n))
	fmt.Println("Values: ", n)
	fmt.Println()

	o := make([]int, len(n)+1)
	copy(o, n)
	o[3] = 4
	fmt.Println("Length: ", len(o))
	fmt.Println("Capacity: ", cap(o))
	fmt.Println("Values: ", o)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/7wTe1-2Jz91)

The copy function takes the destination slice and source slice as an argument and returns the number of elements copied.

```go
func copy(dst, src []T) int
```

##### Examples

**Copy from one slice to another slice**

```go
var odds = make([]int, 3)
n := copy(odds, []int{1, 3, 5}) // n == 3, odds == []int{1, 3, 5}
```

**Copy between the same slice**

```go
odds := []int{1, 3, 5}
n := copy(odds, odds[1:]) // n == 2, s == []int{3, 5, 5}
```

In this example, the values at indexes 0 and 1 are replaced by values at indexes 1 and 2.

**Copy from a string to a byte slice**

```go
var s = make([]byte, 5)
copy(s, "Hello, world!") // s == []byte("Hello")
```

In this example, the slice s has a capacity of 5 elements so the only string Hello is copied to it.

```go
package main
import "fmt"
func main() {
// Copy from one slice to another slice
var odds = make([]int, 3)
copy(odds, []int{1, 3, 5})
fmt.Println(odds)
// Copy between same slice
nums := []int{1, 2, 3, 4}
copy(nums, nums[1:])
fmt.Println(nums)
// Copy from a string to a byte slice
var s = make([]byte, 5)
copy(s, "Hello, world!")
fmt.Println(s)
fmt.Println(string(s))
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/0xzn5-ZlGgZ)

***Thank you for reading this blog, and please give your feedback in the comment section below.***