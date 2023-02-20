# Ranges

The range keyword is used to iterate over various data types. It is used for loops and its return values are dependent on the data types over which we're using the range keyword.

### Range over slice/array

The range on **Array and Slice** returns the first value as an index and the second value as an element located at that index.

```go
package main

import "fmt"

func main() {
	nums := []int{1, 2, 3, 4, 5}
	for i, v := range nums {
		fmt.Printf("Index :%d, value: %d\n", i, v)
	}
}
```

Output:

```go
Index :0, value: 1
Index :1, value: 2
Index :2, value: 3
Index :3, value: 4
Index :4, value: 5
```

[***Run this code in Go Playground***](https://play.golang.org/p/WwdJkWeMfpA)

### Range over maps

The range on **Map** returns the first value as a key and the second value is a value associated with that key.

```go
package main

import "fmt"

func main() {
	m := map[string]int{"foo": 0, "bar": 1}
	for k, v := range m {
		fmt.Printf("Key :%s, value: %d\n", k, v)
	}
}
```

Output:

```go
Key :foo, value: 0
Key :bar, value: 1
```

[***Run this code in Go Playground***](https://play.golang.org/p/DA_QyJzPFB7)

### Range over a string

The range on **String** returns the first value as an index and the second value is rune int.

```go
package main

import "fmt"

func main() {
	str := "golang"
	for i, s := range str {
		fmt.Printf("Index :%d, Rune value: %d\n", i, s)
	}
}
```

Output:

```go
Index :0, Rune value: 103
Index :1, Rune value: 111
Index :2, Rune value: 108
Index :3, Rune value: 97
Index :4, Rune value: 110
Index :5, Rune value: 103
```

[***Run this code in Go Playground***](https://play.golang.org/p/lRFYbsOObFc)

*Rune int value can be type cast string using string() method. e.g. string(103) == g*

### Range over a channel

The range on **Channel** returns only one value which is the value received from the channel.

```go
package main

import "fmt"

func main() {
	channel := make(chan string, 2)
	channel <- "Hello"
	channel <- "World"
	close(channel)

	for v := range channel {
		fmt.Println(v)
	}
}
```

[***Run this code in Go Playground***](https://play.golang.org/p/YPHa5RvPGbP)

***\*KEEP IN MIND\****

You can not update the value in the range loop.

```go
package main

import "fmt"

func main() {
	nums := []int{1, 2, 3, 4, 5}
	fmt.Println("Slice before range: ", nums)

	for _, v := range nums {
		v += 1
	}

	fmt.Println("Slice after range: ", nums)
}
```

Output:

```go
Slice before range:  [1 2 3 4 5]
Slice after range:  [1 2 3 4 5]
```

[***Run this code in Go Playground***](https://play.golang.org/p/iGimIFCgcL_J)

In the above example, I try to increment all the values of the slice by 1 but the slice is unaffected. This is because the range loop copies the value from the slice to the local variable. So to update the slice we'll need to use the traditional way `nums[i] += 1`.

***Thank you for reading this blog, and please give your feedback in the comment section below.***