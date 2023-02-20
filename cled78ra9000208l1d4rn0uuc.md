# Conditional Statements

Conditional Statements are part of every programming language. They help us to decide which instruction is supposed to run when a certain condition is met. *e.g. If I’m hungry then I’ll eat else I’ll wait.* *e.g. If the score is greater than 35%, you passed, else you failed.*

### If `statement` in Go

```go
if condition/expression {
    //instruction to be performed
}
```

The condition needs to be true to perform the given set of instructions.

```go
package main
import "fmt"
func main(){
i := 10
if i % 2 == 0 {
fmt.Printf("%d is a even number", i)
}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/YO_-cELIUii)

### If-Else statement in Go

```go
if condition/expression {
    //instruction to be performed
} else {
    //instruction to be performed
}
```

If the condition needs to be false perform instruction from the else block.

```go
package main
import "fmt"
func main(){
i := 11
if i % 2 == 0 {
fmt.Printf("%d is a even number", i)
} else {
fmt.Printf("%d is a odd number", i)
}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/Fj2fjOaeNZy)

### if-else-if ladder

We can use multiple conditional statements at once.

```go
package main
import "fmt"
func main(){
i := -11
if i == 0 {
fmt.Println("It's zero")
} else if i < 0 {
fmt.Println("Negative number")
} else {
fmt.Println("Positive number")
}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/kL1RyoKUzF-)

### Short statement

```go
package main
import "fmt"
func main() {
if j := 10; j%2 == 0 {
fmt.Println("Even number")
}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/f6SyTAE1Rtz)

***Thank you for reading this blog, and please give your feedback in the comment section below.***