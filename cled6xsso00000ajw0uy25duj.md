# Looping Construct in Go

Go has only one looping construct, the `for` loop.

### Basic for loop

The basic `for` loop has three components separated by semicolons:

* init statement: `i := 0` *exec before 1<sup>st</sup> iteration*
    
* condition expression: `i < n` *eval on every* iteration
    
* post statement: `i++` *exec after each iteration*
    

*The expression is not surrounded by parentheses* `( )` but the braces `{ }` around a set of instructions are required.

```go
for i := 0; i < n; i++ {
    //business logic
    //set of instructions
}
```

Init and post statements are optional.

```go
for ; i < n; {
    //business logic
}
```

### For is also a while() loop

```go
for i < n {
    //business logic
}
```

### Infinite loop

```go
for {
    //business logic
}
```

### Example

```go
package main
import "fmt"

func main() {
    for i := 0; i < 5; i++ {fmt.Printf("Iteration number: %d\n", i)}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/OysJvNK3rNz)

```go
Iteration number: 0
Iteration number: 1
Iteration number: 2
Iteration number: 3
Iteration number: 4
```

***Thank you for reading this blog, and please give your feedback in the comment section below.***