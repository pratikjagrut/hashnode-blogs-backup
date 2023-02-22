# Anonymous Functions in Go

The anonymous function is a feature in Golang which let us define a function without a name. This feature is also called a function literal. This is useful when you want an inline function or to form a closure.

### Declaring the anonymous function

The syntax is pretty straightforward and much similar to normal function.

```go
func(parameter_list)(return_type){
  return
}()
```

`Parameter list` and `return type` are optional.

`()` this will invoke the function as soon as it is defined.

```go
package main

import "fmt"

func main() {
	func() {
		fmt.Println("Golang Rocks!")
	}()
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/Kpnl__MXw7V)

### Assigning an anonymous function to a variable

In Go, you can assign an anonymous function to a variable. The assigned variable will be of function type and it can be called as a regular function.

```go
package main

import "fmt"

func main() {
	v := func() {
		fmt.Println("Golang Rocks!")
	}
	fmt.Printf("Type of variable v: %T\n", v)
	v()
}
```

```go
Type of variable v: func()
Golang Rocks!
```

[***Run this code in Go Playground***](https://play.golang.org/p/Ebz_b0v-AX-)

### Passing an argument to an anonymous function

An anonymous function can take any number of arguments similar to a regular function.

```go
package main

import "fmt"

func main() {
	func(name string) {
		fmt.Println("Hello, ", name)
	}("Jack")
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/oSsoVNXmNwX)

**With any number of trailing arguments similar to Variadic functions**

```go
package main

import "fmt"

func main() {
	func(i ...int) {
		fmt.Println(i)
	}(1, 2, 3, 4)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/vMjTAcWpKec)

**Passing an anonymous function as an argument**

You can pass an anonymous function as an argument to a regular function or an anonymous function.

* *Anonymous functions as an argument to a regular function*
    

```go
package main

import "fmt"

func sayHello(af func(s string) string) {
	fmt.Println(af("Jack"))
}

func main() {

	sayHello(func(s string) string {
		return "Hello, " + s
	})
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/UKJB3fh9DaA)

* *Anonymous function as an argument to an anonymous function*
    

```go
package main

import "fmt"

func main() {
	func(v string) {
		fmt.Println(v)
	}(func(s string) string {
		return "Hello, " + s
	}("Jack"))
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/d8dprfgLXgF)

This above code looks a bit complicated and hard to fathom so another way we can achieve this is the following:

```go
package main

import "fmt"

func main() {
	af := func(s string) string {
		return "Hello, " + s
	}

	func(af func(s string) string) {
		fmt.Println(af("Jack"))
	}(af)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/l7sP7Nz8qpw)

### Return an anonymous function from another function

We can return an anonymous function from another function

* *Returning from regular function*
    

```go
package main

import "fmt"

func sayHello() func(s string) string {
	r := func(s string) string {
		return "Hello, " + s
	}
	return r
}

func main() {
	f := sayHello()
	fmt.Printf("Type of variable f: %T\n", f)
	fmt.Println(f("Jack"))
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/NY3YiPG4N9h)

* *Returning from the anonymous function*
    

```go
package main

import "fmt"

func main() {
	f := func() func(s string) string {
		r := func(s string) string {
			return "Hello, " + s
		}
		return r
	}

	fmt.Printf("Type of variable f: %T\n", f)
	c := f()

	fmt.Printf("Type of variable c: %T\n", c)
	fmt.Println(c("Jack"))
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/jwfmOnhyOGh)

***Thank you for reading this blog, and please give your feedback in the comment section below.***