---
title: "Error Handling in Go (Part-2)"
seoTitle: "Error Handling in Go"
datePublished: Sat Nov 19 2022 22:45:18 GMT+0000 (Coordinated Universal Time)
cuid: cledeoyn2000309md1y3k37kz
slug: error-handling-in-go-part-2
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/zlhxvIcicwk/upload/ab7e382baa91bd9db561fd6fd2bf7bcc.jpeg
tags: programming-blogs, go, error-handling, programming-languages, error-handling-in-go

---

## Creating errors with errors.New() function

If we don't need a custom type for an error, and we can work with a single error type then we can use [***errors.New()***](https://pkg.go.dev/errors#New) function to create an error on the fly.

Creating a new error is simple. We simply need to call the New function as shown in the below example:

```go
package main

import (
  "errors"
  "fmt"
)

func division(i, j float32) (interface{}, error) {
  if j == 0 {
    return "", errors.New("ERROR: Division by Zero")
  }
  return i / j, nil
}

func main() {
  if _, err := division(157, 0); err != nil {
    fmt.Println(err)
  }
}
```

```go
ERROR: Division by Zero
```

[***Run the code in Go Playground***](https://play.golang.org/p/BYKrriQfXUh)

***NOTE: Each call to the New() function returns a distinct error value even if the text is identical.***

***The New() function makes it easy to create an error, but how New() function is doing this?***

Take a look at the source code of the [***errors package.***](https://github.com/golang/go/blob/master/src/errors/errors.go)

```go
package errors

// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
  return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
  s string
}

func (e *errorString) Error() string {
  return e.s
}
```

The New () function takes a string as input and returns the pointer to the type `errorString` struct that only has one field `s string`.

The return type of the New() function is `error`, and the actual returning value is a pointer to the `errorString` struct. This means type `errorString` implements `error interface`.

The implementation of the errors package is pretty straightforward and easy to fathom.

## Creating errors with `fmt.Errorf()` function

Use of errors.New() to create new errors is just fine, but if we want to add more information or context to the error, then New() function is futile because it does not have a built-in string formatting mechanism. We will have to use something like `fmt.Spritnf` to format our error string and then pass it to the New() function which is just overwork.

The better way to use is `fmt.Errorf` function.

```go
package main

import (
  "fmt"
)

func division(i, j float32) (interface{}, error) {
  if j == 0 {
    return "", fmt.Errorf("ERROR: Division by Zero\nDividend: %v\nDivisor: %v", i, j)
  }
  return i / j, nil
}

func main() {
  if _, err := division(157, 0); err != nil {
    fmt.Println(err)
  }
}
```

```go
ERROR: Division by Zero
Dividend: 157.000000
Divisor: 0.000000
```

[***Run the code in Go Playground***](https://play.golang.org/p/xmjbqIshXCi)

In the above program, we have replaced errors.New() function with fmt.Errorf(). Now, we can format our error and add new information to it using the Errorf function.

If you look at the implementation of [fmt.Errorf()](https://github.com/golang/go/blob/master/src/fmt/errors.go) is a bit more complicated than the errors package due to the string formatting feature but at some level fmt.Errorf() function calls the errors.New() function.

***So, the common question is when to use fmt.Errorf() and errors.New()? It depends on the behaviour of the error. If the error needs to have any runtime information, like the address stored in a pointer or the time at which an error occurred, then it is a good idea to use fmt.Errorf() which gives the flexibility to format your error as per your need. On the other side, your error is more of a static behaviour or it is a sentinel error and does not need to have any runtime information, then errors.New() is good enough.***

## Adding context to the error

Sometimes error does not make much sense unless we provide context to it. This context or additional information could be anything like the function where the error occurred or any runtime value.

The general way of adding context to the error is by using `fmt.Errorf and %v verb`.

```go
package main

import (
  "errors"
  "fmt"
)

var ErrUnauthorizedAccess = errors.New("Unauthorized access")

func isUserAuthorized(uname string) error {
  // some logic to verify user authorization
  authorize := false
  if authorize {
    return nil
  }
  return ErrUnauthorizedAccess
}

func searchFile(fileId int) (interface{}, error) {
  uname := "NotAnAdmin"
  err := isUserAuthorized(uname)
  if err != nil {
    return nil, fmt.Errorf("ERROR: searchFile: %v", err)
  }
  return "Return file", nil
}

func main() {
  _, err := searchFile(10001)
  if err != nil {
    fmt.Println(err)
  }
}
```

```go
ERROR: searchFile: Unauthorized access
```

[***Run the code in Go Playground***](https://play.golang.org/p/TkD5FW9UdXF)

In the above example, the original error is `ErrUnauthorizedAccess` which is then annotated with extra information using `fmt.Errorf("ERROR: searchFile: %v", err)`.

While creating a new error using `fmt.Errorf`, everything from the original error is discarded except the text. Hence we lose the original error `ErrUnauthorizedAccess` and the only remnant of the original error is its text `Unauthorized access`. The below example illustrates this problem:

```go
func main() {
  _, err := searchFile(10001)
  if err == ErrUnauthorizedAccess {
    fmt.Println("Please use the valid authorization token")
  }
}
```

[***Run the code in Go Playground***](https://play.golang.org/p/zxoZn0eLwU2)

In the above snippet, the error returned from fmt.Errorf is compared to the original error which should be completely valid. But if you run this code, you will see the blank output. This is because the new error has lost the original error and capability to compare it.

To avoid such situations, we can create a custom error type that stores the original error.

```go
package main

import (
  "errors"
  "fmt"
)

var ErrUnauthorizedAccess = errors.New("Unauthorized access")

type serviceError struct {
  fn  string
  err error
}

func (e *serviceError) Error() string {
  return fmt.Sprintf("Error: %v: %v", e.fn, e.err)
}

func isUserAuthorized(uname string) error {
  // some logic to verify user authorization
  authorize := false
  if authorize {
    return nil
  }
  return ErrUnauthorizedAccess
}

func searchFile(fileId int) (interface{}, error) {
  uname := "NotAnAdmin"
  err := isUserAuthorized(uname)
  if err != nil {
    return nil, &serviceError{fn: "searchFile", err: err}
  }
  return "Return file", nil
}

func main() {
  _, err := searchFile(10001)
  if err != nil {
    if errVal, ok := err.(*serviceError); ok && errVal.err == ErrUnauthorizedAccess{
        fmt.Println("Please use the valid authorization token")
    }
  }
}
```

```go
Please use a valid authorization token
```

[***Run the code in Go Playground***](https://play.golang.org/p/Wsv_szFqW6H)

In the above program, we've struct `serviceError`, a custom error type that stores the original error in the `err` field, and `fn` field that stores the function name where the error will occur(i.e context of an error). In the main function, we're using type assertion to extract the concrete value of `err` and using it to compare with the original error.

***This block of code is also referred to as the unwrapping of error. Because we're extracting the underlying error.***

```go
if errVal, ok := err.(*serviceError); ok && errVal.err == ErrUnauthorizedAccess{
  fmt.Println("Please use the valid authorization token")
}
```

## Wrapping and Unwrapping error

Wrapping of errors means creating a hierarchy of errors by adding context or more information to the error. Consider an onion, and how each upper layer of an onion wraps around the inner layer of the onion. Similarly, one error can wrap up another error, and that error can get wrapped by another error, and so on. We can create a hierarchy of errors which is helpful to form stack trace.

In the above section, we've seen how we added the context to an error using fmt.Errorf is also a kind of error wrapping because it creates a new error over an original error.

In Go1.13, the errors package introduced some new functions to manage errors.

```go
To support wrapping, fmt.Errorf now has a %w verb for creating wrapped errors, 
and three new functions in the errors package ( errors.Unwrap, errors.Is and errors.As) 
simplify unwrapping and inspecting wrapped errors.
```

[***Read more in the release note.***](https://golang.org/doc/go1.13#error_wrapping)

#### Wrapping error

When we used fmt.Errorf and %v to add context to the error we lost the original error. Since Go1.13 `fmt.Errorf` supports `%w` a verb whose argument must be an error. When %w is present fmt.Errorf wraps the error and the error is returned by fmt.Errorf will have Unwrap method that will return the argument of %w.

```go
return fmt.Errorf("adding more context: %w", err)
```

This wrapped error will have access to `errors.Is, errors.As and errors.Unwrap` methods which will help handle them.

#### errors.Is

The `errors.Is` function compares an error to a value. `errors.Is(err, error)`

This statement

```go
if err == ErrUnauthorizedAccess { ... }
```

can be replaced by

```go
if errors.Is(err, ErrUnauthorizedAccess) { ... }
```

***In simple terms, the*** `error.Is` function behaves like a comparison to a sentinel error.

#### errors.As

The `errors.As` function tests whether an error is a specific type. `errors.As(err, &e)`

This statement

```go
if errVal, ok := err.(*Error); ok { ... }
```

can be replaced by

```go
var e *serviceError
if errors.As(err, &e) { ... }
```

err is a `*serviceError`, and e is set to the error's value

***In simple term,*** `errors.As` function behaves like type assertion.

#### errors.Unwrap

Unwrap function is used to extract the underlying or wrapped error.

```go
e2 := fmt.Errorf("adding more context: %w", e1)
errors.Unwrap(e2) // returns e1
```

***NOTE: An error containing another error may implement Unwrap method returning the underlying error. Doing so, will give access to the Is and As methods, without needing to use fmt.Errorf with %w.***

```go
package main

import (
  "errors"
  "fmt"
)

var ErrUnauthorizedAccess = errors.New("Unauthorized access")

type serviceError struct {
  fn  string
  err error
}

func (e *serviceError) Error() string {
  return fmt.Sprintf("Error: %v: %v", e.fn, e.err)
}

func (e *serviceError) Unwrap() error { return e.err }

func isUserAuthorized(uname string) error {
  // some logic to verify user authorization
  authorize := false
  if authorize {
    return nil
  }
  return ErrUnauthorizedAccess
}

func searchFile(fileId int) (interface{}, error) {
  uname := "NotAnAdmin"
  err := isUserAuthorized(uname)
  if err != nil {
    return nil, &serviceError{fn: "searchFile", err: err}
  }
  return "Return file", nil
}

func main() {
  _, err := searchFile(10001)
  if err != nil {
    fmt.Println(err)

    if errors.Is(err, ErrUnauthorizedAccess) {
      fmt.Println("Please use valid authorization token")
    }

    var e *serviceError
    if errors.As(err, &e) {
      fmt.Printf("Function %v returned %v error\n", e.fn, e.err)
    }

    fmt.Println(errors.Unwrap(err))
  }
}
```

```go
Error: searchFile: Unauthorized access
Please use a valid authorization token
Function searchFile returned Unauthorized access error
Unauthorized access
```

[***Run the code in Go Playground***](https://play.golang.org/p/OJcik2q5FQk)

In the above program, struct `serviceError` implements the Error method, which makes it a custom error type. It has an error field and implements the `Unwrap` method which returns the original error, this lets us call `errors.Is` and `errors.As` method on it.

If you remember the previous example, we used this block of code to unwrap the error.

```go
if errVal, ok := err.(*serviceError); ok && errVal.err == ErrUnauthorizedAccess{
  fmt.Println("Please use the valid authorization token")
}
```

But now, we replaced it by errors.Is

```go
if errors.Is(err, ErrUnauthorizedAccess) {
  fmt.Println("Please use valid authorization token")
}
```

##### The same example using fmt.Errorf and %w verb

```go
package main

import (
  "errors"
  "fmt"
)

var ErrUnauthorizedAccess = errors.New("Unauthorized access")

func isUserAuthorized(uname string) error {
  // some logic to verify user authorization
  authorize := false
  if authorize {
    return nil
  }
  return ErrUnauthorizedAccess
}

func searchFile(fileId int) (interface{}, error) {
  uname := "NotAnAdmin"
  err := isUserAuthorized(uname)
  if err != nil {
    return nil, fmt.Errorf("ERROR: searchFile: %w", err)
  }
  return "Return file", nil
}

func main() {
  _, err := searchFile(10001)
  if err != nil {
    fmt.Println(err)

    if errors.Is(err, ErrUnauthorizedAccess) {
      fmt.Println("Please use valid authorization token")
    }

    fmt.Println(errors.Unwrap(err))
  }
}
```

```go
ERROR: searchFile: Unauthorized access
Please use a valid authorization token
Unauthorized access
```

[***Run the code in Go Playground***](https://play.golang.org/p/0io6Fr3Sgy1)

*In software, errors are an inevitable part. We can not ignore them, thinking we'll save the hustle of handling them. But ignoring error means weakening the software instead handle it.*

Below are some good reads about error handling.

* [***Don’t just check errors, handle them gracefully***](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)
    
* [***Working with Errors in Go 1.13***](https://blog.golang.org/go1.13-errors)
    
* [***Why Go's Error Handling is Awesome***](https://rauljordan.com/2020/07/06/why-go-error-handling-is-awesome.html)
    
* [***Return and handle an error***](https://golang.org/doc/tutorial/handle-errors)
    

***Thank you for reading this blog, and please give your feedback in the comment section below.***