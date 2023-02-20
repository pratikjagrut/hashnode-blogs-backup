# Map in Go

The map is a collection of key-value pairs. It is an implementation of a Hash Table, which provides Create/Add, Read, Update and Delete operations over the data. The collection of key-value pairs is unordered and each key is unique.

## Declaration and initialization

In general GO map looks like

```go
map[KeyType]ValueType
```

The KeyType could be anything that is comparable such as string, int etc. and ValueType could be anything, even if it can be another map.

```go
var m map[int]string
```

The above m is a map of int keys to string values.

Maps are reference types such as slices. So uninitialized maps value is nil i.e the zero value of the map is `nil`. The nil map has no keys and we can not add any key to it because it does not point to any initialized map. Trying to add a key to a nil map will throw `panic: assignment to entry in nil map` an error.

```go
package main

import "fmt"

func main() {
	var m map[int]string
	fmt.Printf("%T\n", m)
	fmt.Println(m)
  	m[1] = "one"
}
```

```go
map[int]string
map[]
panic: assignment to entry in nil map

goroutine 1 [running]:
main.main()
	/tmp/sandbox272715404/prog.go:9 +0x107
```

[***Run this code in Go Playground***](https://play.golang.org/p/V_CiNrkv9gN)

So to get an initialized and ready-to-use map uses the `make` function.

```go
package main

import "fmt"

func main() {
	m := make(map[int]string)
	fmt.Printf("%T\n", m)
	fmt.Println(m)
	m[1] = "one"
	fmt.Println(m)
}
```

```go
map[int]string
map[]
map[1:one]
```

[***Run this code in Go Playground***](https://play.golang.org/p/wBiLI4qulzY)

In the background make function assigns and initializes the hash map data structure and returns the map value which points to the hash map.

**Map literal**

```go
package main

import "fmt"

func main() {
	m := map[string]int{"foo": 0, "bar": 1}
	fmt.Printf("%T\n", m)
	fmt.Println(m)
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/PgpzaAp2X7q)

## Working with the maps

**Create a map using the make function**

We created a map m of string keys to int values.

```go
m := make(map[string]int)
```

**Add key-value pair**

The syntax is fairly similar and easy to follow. Below key `one` is set to the value `1`.

```go
m["one"] = 1
```

Update operation has the same syntax

```go
m["one"] = 2
```

**Read a value**

```go
i := m["one"]
```

If the key is not present then we get the value of the type's zero value.

```go
j := m["two"] // Key not present
// j == 0
```

We can check if the key is present in the map with a two-value assignment statement.

```go
k, ok := m["three"]
_, ok := m["three"] // Without retrieving the value.
```

If the key is present `ok == true` if the key is not found `ok == false`.

**Delete value**

The built-in delete function will delete the values from the map. It takes `map variable` and `key` as an argument and it does not return anything. If the given key is not found it'll not do anything.

```go
delete(m, "one")
```

**Iterating over the values in maps**

We can use `range` it to iterate.

```go
for key, value := range m {
    fmt.Println("Key:", key, "Value:", value)
}
```

While iterating over the map the iteration order is not specific, you may get different iteration orders next time you iterate.

```go
package main

import "fmt"

func main() {
	// **Create a map using make function**
	m := make(map[string]int)

	// **Add key-value pair**
	m["one"] = 1
	fmt.Println("Value of m: ", m)

	// Update
	m["one"] = 2
	fmt.Println("Updated value of m: ", m)
	fmt.Println()

	// **Read a value**
	i := m["one"]
	fmt.Println("Value of i", i)

	// Key not present
	j := m["two"]
	fmt.Println("Value of j: ", j)
	fmt.Println()

	// check if the key is present
	key, ok := m["three"]
	fmt.Println(key, ok)

	// Without retrieving the value.
	_, k := m["three"]
	fmt.Println(k)
	fmt.Println()

	// **Delete value**
	delete(m, "one")
	fmt.Println("Value of m after deletion: ", m)
	fmt.Println()

	// **Iterating over the values in maps**
	m["one"] = 1
	m["two"] = 2
	m["three"] = 3
	fmt.Println("Iterate over map")
	for key, value := range m {
		fmt.Println("Key:", key, "Value:", value)
	}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/rT3ajhAyenr)

***Thank you for reading this blog, and please give your feedback in the comment section below.***