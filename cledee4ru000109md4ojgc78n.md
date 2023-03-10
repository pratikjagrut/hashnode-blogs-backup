# Error Handling in Go (Part-1)

`Error` is a result of unexpected circumstances or abnormal behaviour of the program. For example, a network connection error, a user entering incorrect data, a function invoked with inaccurate values or a file not found, and many other scenarios could cause an error. `Error Handling` is a process of knowing where an error could befall, and having a mechanism to handle it.

In Golang, `an error is a value type whose zero value is nil` i.e, the error is a built-in type. This coerces developers to treat the error as a [***first-class citizen***](https://en.wikipedia.org/wiki/First-class_citizen). Errors are treated similarly as int, string, bool, etc. An error may return from any function, it may pass as a parameter, and it may store into a variable.

Golang has errors but no exceptions. Its way to handle the exception is not to have an exception in the first place. Please read Dave Cheney's blog about [***Why Go gets exceptions right***](https://dave.cheney.net/2012/01/18/why-go-gets-exceptions-right).

Golang has support for multiple return values from the function. This feature is primarily used for returning a result and an error. Below is a signature of a Read method from [***bufio package.***](https://golang.org/src/bufio/bufio.go?s=4875:4925#L188)

```go
func (b *Reader) Read(p []byte) (n int, err error)
```

`Read` the method returns the number of bytes reads into p and returns a variable `err` of type `error`. So, for some reason, if this function fails, then the caller will be apprised by returning a custom-defined error. The caller should first inspect the `error` to determine if an error has occurred or not and only after that returned result should be used. Below is an idiomatic way of handling errors in Golang.

```go
if err != nil {
  // handle error
}
```

The above block of code is pervasive in Golang based projects. If the error value is not equal to nil that means the function has returned an error. This idiomatic way of handling errors gives us more control over error handling. We can decorate the error and make it more informative. Below is an example of error handling.

```go
package main

import (
  "fmt"
  "os"
)

func main() {
  f, err := os.Open("README.md")
  if err != nil {
    fmt.Println(err)
    os.Exit(1)
  }
  fmt.Println(f.Name(), "opened successfully")
}
```

```go
open README.md: no such file or directory
Program exited: status 1.
```

[***Run the code in Go Playground***](https://play.golang.org/p/m6f5_P8Brqz)

In the above program, we are trying to open the `README.md` file that is not present, so `os.Open` the function returns an error that we are inspecting and handling.

## Representation of error type

In Golang, an error is a built-in type, but actually, the error is an `interface type made available globally`. [***It is a single-method interface.***](https://golang.org/ref/spec#Errors)

```go
type error interface {
  Error() string
}
```

Any type that implements the error interface is a valid error. To create a custom error, a type needs to define the `Error` method. In the above example of the opening file, the error returned from `os.Open` function was of type [***\*pathError***](https://github.com/golang/go/blob/master/src/os/file.go#L316). `pathError` is a custom error which [***implements the error interface***](https://golang.org/pkg/io/fs/#PathError).

## Create an error type

From the above section, we know that an error is an interface, and any type which implements that interface is an error.

```go
package main

import "fmt"

type httpError struct {
  code   int
  method string
  fn     string
}

func (h *httpError) Error() string {
  return fmt.Sprintf("ERROR:%d:%s:%s", h.code, h.method, h.fn)
}

//Mock function to demo some operation
func getUser(uid int) (interface{}, error) {
  // some business logic
  authorized := false
  if authorized {
    //find user logic
    return "User found", nil
  }
  return "", &httpError{code: 401, method: "GET", fn: "getUser"}
}

func main() {
  _, err := getUser(1)
  if err != nil {
    fmt.Println(err)
  }

  // Inline variant
  // if _, err = getUser(2); err != nil {
  //  fmt.Println(err)
  // }
}
```

```go
ERROR:401:GET:getUser
```

[***Run the code in Go Playground***](https://play.golang.org/p/dFSDoh2fBjz)

In the above example, the `httpError` struct implements the `error` interface by defining the `Error` method. `getUser` is a mock function that mocks the getUser operation.

Here, the main function is the caller for the methods. So the main function is supposed to handle the returned errors. In the main function, this statement `_, err := getUser(1)` executes the getUser function, stores the error in the `err` variable and ignores the result using `_`.

After the function call, the caller is checking if the function is returning an error or not using an if-else statement.

If you see the return type of an error in `getUser` function is `error interface type` but the return statement is returning a struct or say the address of the struct variable `&httpError{code: 401, method: "GET", fn: "getUser"}`.

***If the return type of a function and the type of actual return value is different, then how can this function be returning anything?***

This is because the type `httpError` implements the `error interface`. Under the hood, the interface variable will hold the value and type that implements it.

Another thing to notice is the output.

```go
ERROR:401:GET:getUser
```

We are passing variable `err` of type `httpError struct` to the `fmt.Println` function, and it is printing nicely documented output, instead of messy struct variables.

***How?***

This is happening because we have defined the `Error` method that returns a custom error message string. So under the hood, the `fmt.Println` function is called the `Error` method and prints the returned string. So we never have to explicitly call the Error method to get a custom error message.

## Handling errors using the type assertion

In the above example, the variable `err` is of type interface. err variable will not have access to fields of httpError. But err is a variable of error interface and that interface is implemented by type httpError we can use type assertion to extract the underlying concrete value of the interface variable.

In the ***interfaces*** blog, we've seen how a type assertion is performed. By extracting this underlying value, we can gain more control over the error value, and we can decide how to return this information to the caller. Below is an illustrative example:

```go
package main

import "fmt"

type httpError struct {
  code   int
  method string
  fn     string
}

func (h *httpError) Error() string {
  return fmt.Sprintf("ERROR: %d:%s:%s\n", h.code, h.method, h.fn)
}

//Mock function to demo some operation
func getUser(uid int) (interface{}, error) {
  // some business logic
  authorized := false
  if authorized {
  //find user logic
  return "User found", nil
  }
  return "", &httpError{code: 401, method: "GET", fn: "getUser"}
}

func main() {
  _, err := getUser(1)
  if err != nil {
    if errVal, ok := err.(*httpError); ok {
      fmt.Printf("Status Code: %d\n", errVal.code)
      fmt.Printf("Method: %s\n", errVal.method)
      fmt.Printf("function returned error: %s\n", errVal.fn)
    }
  }
}
```

```go
Status Code: 401
Method: GET
function returned error: getUser
```

[***Run the code in Go Playground***](https://play.golang.org/p/cVIHQEKs1a5)

This one is the same example as the previous one with a small tweak. Instead of printing the `err` variable, we are performing type assertion on it. Here `errVal, ok := err.(*httpError)` will return a pointer to the instance of the `httpError`. Now we can access the fields of the `httpError` struct using `errVal`.

In case the underlying error is not of type `*httpError`, then the `ok` the variable will be set to false returning control outside the if block.

## Handling errors using type switch

Using a type switch we can check the type of error and handle it accordingly. Read more about type switch in the ***interfaces blog.*** Below is an illustrative example:

```go
package main

import "fmt"

type fooError struct {
  msg string
}

func (f *fooError) Error() string {
  return fmt.Sprintf("Error: %s", f.msg)

}

type barError struct {
  msg string
}

func (b *barError) Error() string {
  return fmt.Sprintf("Error: %s", b.msg)
}

func doSomething() error {
  return &fooError{"Something is wrong with foo!"}
}

func main() {
  err := doSomething()

  switch err.(type) {
  case nil:
    fmt.Println("Everything is fine.")
  case *fooError:
    fmt.Println("Foo: ", err)
  case *barError:
    fmt.Println("Bar: ", err)
  }
}
```

```go
Foo:  Error: Something is wrong with foo!
```

[***Run the code in Go Playground***](https://play.golang.org/p/7tbNi8TmkBg)

In the above example, `switch err.(type)` will discern the underlying type of the `err` variable and after comparing it with all the cases, the error will be handled by matched case.

***In this blog, we have seen:***

* *What is an error in Golang?*
    
* *How error is represented in Golang?*
    
* *Implementing an error interface to create error*
    
* *Using type assertion and type switch to extract more information about the error*
    

*In the next blog, we'll see more about creating custom errors and handling them gracefully.*

***Thank you for reading this blog, and please give your feedback in the comment section below.***