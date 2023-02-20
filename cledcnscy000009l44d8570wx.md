# Functions in Go

A function is a block of statements which performs a specific task. A function is a well-organized and reusable code. It improves the code readability, maintainability and testability. The general function is:

```go
func function_name( [parameter list] ) [return_types]
{
  body of the function
}
```

### Declaring and calling functions

The function is declared using `func` the keyword.

```go
func sayCheeze() {
  fmt.Println("Cheeeeeeeeeeeeze")
}
```

Calling the function is pretty easy.

```go
sayCheeze()
```

```go
package main

import "fmt"

func sayCheeze() {
	fmt.Println("Cheeeeeeeeeeeeze")
}

func main() {
	sayCheeze()
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/sz4RmqzeqZZ)

### You may pass input parameters

```go
func addition(i int, j int) {
  fmt.Println(i + j)
}
```

We need to pass the declared number of parameters while calling the function.

```go
addition(1, 2)
```

When two or more consecutive parameters have the same type we can omit the type from all but the last.

```go
package main

import "fmt"

func addition(i, j, k int) {
	fmt.Println(i + j + k)
}

func main() {
	addition(10, 20, 30)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/Hd3WlDtdSbI)

### You may define the return type of the function

```go
func addition(i, j, k int) int {
   return i + j + k
}
```

```go
sum := addition(1, 2, 3)
```

A function can return any number of results.

```go
package main

import "fmt"

func addition(i, j int) (int, int, int) {
	return i, j, i + j
}

func main() {
	a, b, sum := addition(1, 2)
	fmt.Println(a, b, sum)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/hqkVQEkp_Ik)

### Named return values

Go's return values may be named. The named return values are treated as locally defined variables in the function.

```go
package main

import "fmt"

func division(i int) (quotient, remainder int) {
	quotient = i / 10
	remainder = i % 10
	return
}
func main() {
	quotient, remainder := division(125)
	fmt.Printf("Quotient: %d and Remainder: %d", quotient, remainder)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/4mOb0aPX7B2)

### Variadic functions

Variadic functions can be called with any number of trailing arguments. For example, `fmt.Println` is a common variadic function.

```go
package main

import "fmt"

func total(nums ...int) (total int) {
	for _, i := range nums {
		total += i
	}
	return total
}

func main() {
	fmt.Println(total(1, 2, 3, 4, 5))
	fmt.Println(total(1, 2, 3, 4, 5, 6, 7, 8, 9, 10))
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/g0EkydaW9k2)

***Thank you for reading this blog, and please give your feedback in the comment section below.***