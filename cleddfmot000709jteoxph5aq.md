# Structs in Go (Part-1)

## What is Struct?

A struct is a sequence of named elements, called fields, each of which has a name and a type. Field names may be specified explicitly (IdentifierList) or implicitly (EmbeddedField). Within a struct, non-blank field names must be unique. [***From golang docs.***](https://golang.org/ref/spec#Struct_types)

In simple words, a struct is a user-defined type that allows the collection of different types of elements. These elements are called fields. Structs can be used to keep certain data together. A struct is a very popular and widely used user-defined type in golang.

## Struct Declaration and Initialization

A struct is defined using keywords `type` and `struct`. The type keyword is used to define user-defined data types and the struct keyword specifies that it's a struct.

```go
type Person struct {
    name    string
    age     int
}
```

To use the struct we create a struct object.

`var p Person` or shorthand `p := Person{}` will create an object of struct Person and will initialize its field with the zero-value of their type.

```go
package main

import "fmt"

type Person struct {
    name    string
    age     int
}

func main() {
    var p Person
    fmt.Printf("%#v", p)

    p1 := Person{}
    fmt.Printf("%#v\n", p1)
}
```

```go
main.Person{name:"", age:0}
main.Person{name:"", age:0}
```

`%#v` a Go-syntax representation of the value.

[***Run this code in Go Playground***](https://play.golang.org/p/APy03otXxrI)

***Struct literals***

Creating a struct object with specifying field names

```go
p := Person{name: "Jon Snow", age: 24}
p1 := Person{
    name: "Jon Snow",
    age:  24,
}
```

The order of the field does not matter when specifying field names

```go
p := Person{age: 24, name: "Jon Snow"}
```

Creating a struct object without specifying field names

```go
p := Person{"Jon Snow", 24}
p1 := Person{
  "Jon Snow",
  24,
}
```

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func main() {
    p := Person{name: "Jon Snow", age: 24}
    fmt.Printf("%#v\n", p)

    p1 := Person{"Jon Snow", 24}
    fmt.Printf("%#v\n", p1)

    p2 := Person{
        name: "Jon Snow",
        age:  24,
    }
    fmt.Printf("%#v\n", p2)

    p3 := Person{
        "Jon Snow",
        24,
    }
    fmt.Printf("%#v\n", p3)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/Ec0BcnT7dwg)

## Accessing struct fields

To access individual fields of the struct, `.` (dot) operator is used. e.g.`struct.fieldname`.

To use the value present in a struct field.

```go
value := struct.fieldname
```

To assign a new value to the struct field

```go
struct.fieldname = value
```

The below example demonstrates how to access the struct fields.

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func main() {
    p := Person{name: "Jon Snow", age: 24}
    fmt.Printf("Name of the person: %v\n", p.name)
    fmt.Printf("Age of the person: %v\n", p.age)

    p.age = 30
    fmt.Printf("Updated age of the person: %v\n", p.age)
}
```

```go
Name of the person: Jon Snow
Age of the person: 24
Age of the person: 30
```

[***Run this code in Go Playground***](https://play.golang.org/p/jL6nC8aoHZs)

## Pointer to the struct

[Pointers](/blog/golang/series/pointers) are special variable that stores the address of another variable. We can store the address of a struct object in a pointer by passing the memory address using `&` operator.

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func main() {
    // First way of creating pointer to struct
    personPtr := &Person{name: "James Bond", age: 32}
    fmt.Println(personPtr)
    fmt.Println(*personPtr) // Dereferencing the struct pointer

    fmt.Printf("Age of \"%v\" is \"%v\" \n", (*personPtr).name, (*personPtr).age)

    fmt.Println()
    // Second way of creating pointer to struct
    person := Person{name: "Jon Snow", age: 24}
    ptr := &person
    fmt.Println(ptr)
    fmt.Println(*ptr) // Dereferencing the struct pointer

    fmt.Printf("Age of \"%v\" is \"%v\" \n", (*ptr).name, (*ptr).age)
}
```

```go
&{James Bond 32}
{James Bond 32}
Age of "James Bond" is "32" 

&{Jon Snow 24}
{Jon Snow 24}
Age of "Jon Snow" is "24"
```

[***Run this code in Go Playground***](https://play.golang.org/p/lJYO7n5moq5)

In the above program, we're accessing the struct field by dereferencing the pointer `(*ptr).name, (*ptr).age` which reduces the readability and makes a program a little unkempt. The Golang also allow us to access fields of the struct without dereferencing the pointer as shown in the below example.

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func main() {
    // First way of creating pointer to struct
    person := &Person{name: "James Bond", age: 32}
    fmt.Println(person)
    fmt.Printf("Age of \"%v\" is \"%v\" \n", person.name, person.age)
}
```

```go
&{James Bond 32}
Age of "James Bond" is "32"
```

[***Run this code in Go Playground***](https://play.golang.org/p/nYCTG9in-ws)

In the above program, we're accessing the field using `person.name, person.age`.

## Anonymous Struct and anonymous fields

### Anonymous Struct

In Golang we're allowed to create a struct without a name such a struct is called an anonymous struct. This kind of struct is useful if we want to use the struct only once in the program. We can create a struct using the following syntax:

```go
variable := struct{
    // field
}{
    // field value
}

// OR

PtrVariable := &struct{
    // field
}{
    // field value
}
```

Below is an example of an anonymous struct.

```go
package main

import "fmt"

func main() {
    person := struct {
        name string
        age  int
    }{
        name: "James Bond",
        age:  32,
    }
    fmt.Println(person)

    ptrPerson := &struct {
        name string
        age  int
    }{
        name: "Jon Snow",
        age:  24,
    }
    fmt.Println(ptrPerson)
}
```

```go
{James Bond 32}
&{Jon Snow 24}
```

[***Run this code in Go Playground***](https://play.golang.org/p/1FQzainnKVs)

### Anonymous fields

In Golang we're allowed to define anonymous fields in a struct. Anonymous fields are fields without a name. We just need to define the type of the field and Go will use that type as the name of that field. Below is the syntax of anonymous fields

```go
type structName struct{
    // value type
    string
    int
    bool
}
```

```go
package main

import "fmt"

type Person struct {
    string
    int
}

func main() {
    person := &Person{
        "James Bond", 
        32,
    }
    fmt.Println(person)

    fmt.Printf("Age of \"%v\" is \"%v\" \n", person.string, person.int)
}
```

```go
&{James Bond 32}
Age of "James Bond" is "32"
```

[***Run this code in Go Playground***](https://play.golang.org/p/r5kNOoc1Xb5)

In the above example, we access the anonymous fields by using value type as the name of the field `person.string, person.int`.

***It is not allowed to define anonymous fields of the same type more than once.***