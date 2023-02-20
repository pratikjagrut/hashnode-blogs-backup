# Switch Statement in Go

A switch statement is another way to write a sequence of if-else statements.

Go’s switch is like the one in C and C++ except that `it only runs the selected case, not all the cases that follow` we don’t need a break statement here.

```go
switch expression {
    case exp1:
        //Executes if expression matches exp1
    case exp2:
        //Executes if expression matches exp2
    default:
        //Executes if expression does not matches with any case
}
```

### Switch evaluation order

Switch cases evaluate cases from top to bottom and stop when a case succeeds.

In the below example, it checks `case "Linux":` if os matches `Linux` then stops at that case else goes to the next case.

```go
package main
import (
"fmt"
"runtime"
)
func main() {
//Prints which OS you're using
switch os := runtime.GOOS; os {
case "linux":
fmt.Println("Linux.")
case "darwin":
fmt.Println("OS X.")
default:
fmt.Printf("%s.\n", os)
}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/pnjVB1dPSkX)

### Switch with NO condition

Switch with no condition is like `switch true`. It is useful for writing a long `if-else-if` ladder.

```go
package main
import (
"fmt"
"time"
)
func main() {
t := time.Now()
switch {
case t.Hour() < 12:
fmt.Println("Good morning!")
case t.Hour() < 17:
fmt.Println("Good afternoon.")
default:
fmt.Println("Good evening.")
}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/NT19bQTpqAi)

***Thank you for reading this blog, and please give your feedback in the comment section below.***