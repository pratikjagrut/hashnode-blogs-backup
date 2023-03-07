---
title: "Structs in Go (Part-2)"
seoTitle: "Structs in Go"
datePublished: Fri Sep 23 2022 22:16:11 GMT+0000 (Coordinated Universal Time)
cuid: cleddni8j000109lag48gguds
slug: structs-in-go-part-2
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/26MJGnCM0Wc/upload/02071456d6c15892a43837edcde6cdc9.jpeg
tags: programming-blogs, go, tutorials, programming-languages, struct

---

## Nested Structs

Go allows us to use a struct as a field of another struct, this pattern is called nesting. A nested struct can be defined using the following syntax.

```go
type struct1 struct{
    // fields
}

type struct2 struct{
    // fields
    s struct1
}
```

Suppose we need to collect the data of a person, the person's name, age and address. In the address, we need to collect the city and country of that person. So we can do this using nested structs as shown below:

```go
package main

import "fmt"

type Person struct {
    name    string
    age     int
    address Address
}

type Address struct {
    city, country string
}

func main() {
    person := &Person{
        name: "James Bond",
        age:  34,
        address: Address{
            city:    "London",
            country: "UK",
        },
    }

    fmt.Printf("%#v", person)
}
```

```go
&main.Person{name:"James Bond", age:34, address:main.Address{city:"London", country:"UK"}}
```

[***Run this code in Go Playground***](https://play.golang.org/p/e4A9YhIirqW)

### Accessing fields of nested struct

To access the fields of nested structs we can do `structObj1.structObj2.structObj3........fieldName`.

Below is an example:

```go
package main

import "fmt"

type Person struct {
    name    string
    age     int
    address Address
}

type Address struct {
    city, country string
}

func main() {
    person := &Person{
        name: "James Bond",
        age:  34,
        address: Address{
            city:    "London",
            country: "UK",
        },
    }

    fmt.Printf("Name: %v\n", person.name)
    fmt.Printf("Age: %v\n", person.age)
    fmt.Printf("City: %v\n", person.address.city)
    fmt.Printf("Country: %v\n", person.address.country)
}
```

```go
Name: James Bond
Age: 34
City: London
Country: UK
```

[***Run this code in Go Playground***](https://play.golang.org/p/OTkVLWQRf3Q)

### Promoted fields

When the struct has another struct as an anonymous field then fields of anonymous structs are called promoted fields. This means we can access the fields of the nested struct without using the object of the nested struct, we can access them just by using the parent struct.

For example:

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
    Address
}

type Address struct {
    city, country string
}

func main() {
    person := &Person{
        name: "James Bond",
        age:  34,
        Address: Address{
            city:    "London",
            country: "UK",
        },
    }

    fmt.Printf("Name: %v\n", person.name)
    fmt.Printf("Age: %v\n", person.age)
    fmt.Printf("City: %v\n", person.city) // Instead of person.address.city we used person.city
    fmt.Printf("Country: %v\n", person.country) // Instead of person.address.country we used person.country
}
```

```go
Name: James Bond
Age: 34
City: London
Country: UK
```

[***Run this code in Go Playground***](https://play.golang.org/p/wkxWR0u6zoT)

***Only unique fields of nested anonymous struct gets promoted.***

If the parent struct and nested anonymous struct have the same name field then that field will not be promoted.

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
    City
}

type City struct {
    name, country string
}

func main() {
    person := &Person{
        name: "James Bond",
        age:  34,
        City: City{
            name:    "London",
            country: "UK",
        },
    }

    fmt.Printf("Name: %v\n", person.name)
    fmt.Printf("Age: %v\n", person.age)
    fmt.Printf("City: %v\n", person.City.name)
    fmt.Printf("Country: %v\n", person.country)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/gklcUfcRJV9)

In this example, we have `City` a nested anonymous struct, whose fields are `name` and `country`. The `name` the field is also available in the `Person` struct. So when we try `person.name` then the compiler will give us access to the Person's name field by default hence to access the name field from the City struct we've to do `person.City.name`. The country field from City struct is unique here so we can just access it by using `person.country`.

## Comparing struct

Structs in Go are value type and so they can be compared.

Two struct values may be equal if:

* They're of the same type.
    
* Their corresponding fields are equal.
    
* Their corresponding fields should be comparable.
    

For example:

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func main() {
    p1 := Person{
        name: "James Bond",
        age:  34,
    }

    p2 := Person{
        name: "James Bond",
        age:  34,
    }

    fmt.Println(p1 == p2) // true
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/KZrnjv-8qC1)

***Comparing pointers to the struct will always be false, we need to compare dereferenced pointer values.***

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func main() {
    p1 := &Person{
        name: "James Bond",
        age:  34,
    }

    p2 := &Person{
        name: "James Bond",
        age:  34,
    }
    fmt.Println(p1 == p2) // false
    fmt.Println(*p1 == *p2) // true
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/qcQYTF_BUcP)

If structs contain incomparable values then struct values can not be compared using `==` operator, it will throw an error.

```go
package main

import "fmt"

type Person struct {
    name     string
    age      int
    measures map[string]float32
}

func main() {
    p1 := Person{
        name: "James Bond",
        age:  34,
        measures: map[string]float32{
            "Height": 183,
            "Weight": 76,
        },
    }

    p2 := Person{
        name: "James Bond",
        age:  34,
        measures: map[string]float32{
            "Height": 183,
            "Weight": 76,
        },
    }
    fmt.Println(p1 == p2)
}
```

```go
./prog.go:30:18: invalid operation: p1 == p2 (struct containing map[string]float32 cannot be compared)
```

In the above code, a map is used as a field in a struct and a map is an incomparable type. To compare such structs we can use `reflect.DeepEqual()` function.

```go
package main

import (
    "fmt"
    "reflect"
)

type Person struct {
    name     string
    age      int
    measures map[string]float32
}

func main() {
    p1 := Person{
        name: "James Bond",
        age:  34,
        measures: map[string]float32{
            "Height": 183,
            "Weight": 76,
        },
    }

    p2 := Person{
        name: "James Bond",
        age:  34,
        measures: map[string]float32{
            "Height": 183,
            "Weight": 76,
        },
    }
    fmt.Println(reflect.DeepEqual(p1, p2)) // true
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/R1nK7gPUcrt)

## Structs and methods/receiver function

Golang supports both function and method. A method is a function that is defined for a particular type or with a receiver. A method in Golang is also called a receiver function. Following is the example.

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func (p Person) GetInfo() string {
    return fmt.Sprintf("\"%v\" is \"%v\" years old.", p.name, p.age)
}

func main() {
    p := Person{
        name: "James Bond",
        age:  34,
    }

    fmt.Println(p.GetInfo())
}
```

```go
"James Bond" is "34" years old.
```

[***Run this code in Go Playground***](https://play.golang.org/p/NnCg1hi-t2d)

We can also use a pointer as the receiver of the method. The difference between value as receiver and pointer as a receiver is, golang passes everything as value and hence in value receiver the function will get a copy of the struct and function will not touch the original struct whereas in pointer as a receiver function will get a copy of a pointer which will be pointing to an original struct.

The below examples illustrate the difference between a pointer and a value as a receiver.

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

// Value as receiver
func (p Person) UpdateAge(age int) {
    p.age = age
}

// Pointer as receiver
func (p *Person) UpdateName(name string) {
    p.name = name
}

func main() {
    p := Person{
        name: "James Bond",
        age:  34,
    }

    // Value as receiver example
    fmt.Println("Age before update: ", p.age)
    p.UpdateAge(24)
    fmt.Println("Age after update: ", p.age)

    fmt.Println()

    // Pointer as receiver
    fmt.Println("Name before update: ", p.name)
    p.UpdateName("Jon Snow")
    fmt.Println("Name after update: ", p.name)
}
```

In above example `UpdateAge()` is using value as the receiver and `UpdateName()` is using pointer as a receiver.

```go
Age before update:  34
Age after update:  34

Name before update:  James Bond
Name after update:  Jon Snow
```

[***Run this code in Go Playground***](https://play.golang.org/p/gvDMobx70PK)

## Empty Struct

In Golang, we can have an empty struct. A struct without any field is called an empty struct. Below is the signature of an empty struct.

```go
type T struct{}
var s struct{}
```

Empty struct does not have any field so it does not occupy any memory i.e it occupies `zero bytes` of storage.

```go
var s struct{}
fmt.Println(unsafe.Sizeof(s)) // prints 0
```

We can create an array of the empty struct and still it occupies zero bytes.

```go
var arr [100000]struct{}
fmt.Println(unsafe.Sizeof(arr)) // prints 0
```

The slice of structs will consume some bytes just to store the pointer, length and capacity of an underlying array but structs still will be of zero bytes.

```go
sls := make([]struct{}, 100000)
fmt.Println(unsafe.Sizeof(sls)) // prints 24
```

```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    var s struct{}
    fmt.Println("Size of an empty struct: ", unsafe.Sizeof(s)) // prints 0

    var arr [100000]struct{}
    fmt.Println("Size of an array of an empty struct: ", unsafe.Sizeof(arr)) // prints 0

    sls := make([]struct{}, 100000)
    fmt.Println("Size of a slice of an empty struct: ", unsafe.Sizeof(sls)) // prints 24
}
```

```go
Size of an empty struct:  0
Size of an array of an empty struct:  0
Size of a slice of an empty struct:  24
```

[***Run this code in Go Playground***](https://play.golang.org/p/SB9x1tSICW1)

Read more about the empty struct and its use cases at [***Dave Cheney's blog.***](https://dave.cheney.net/2014/03/25/the-empty-struct)

## Array/Slice of structs

The ***Array/Slice*** is a collection of the same type of element in a contiguous memory location. We can create an array/slice of structs.

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func newPerson(name string, age int) *Person {
    return &Person{
        name: name,
        age:  age,
    }
}

func main() {
    people := make([]*Person, 0) // slice of a struct Person with initial length 0

    people = append(people, newPerson("James Bond", 34))
    people = append(people, newPerson("Jon Snow", 24))
    people = append(people, newPerson("Joey Tribbiani", 32))

    for _, person := range people {
        fmt.Println(person)
    }

    fmt.Println("Name of friends character: ", people[2].name)
}
```

```go
&{James Bond 34}
&{Jon Snow 24}
&{Joey Tribbiani 32}
Name of friends character:  Joey Tribbiani
```

[***Run this code in Go Playground***](https://play.golang.org/p/g9-754U92un)

***Thank you for reading this blog, and please give your feedback in the comment section below.***