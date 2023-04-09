---
title: "Concurrency in Go (Part-3): sync package primitives"
datePublished: Sun Apr 09 2023 16:12:41 GMT+0000 (Coordinated Universal Time)
cuid: clg9lsib7000009kxej4ydejt
slug: concurrency-in-go-part-3-sync-package-primitives
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/NbgQfUvKFE0/upload/a8bd526c69cf1b624401cfe91c76482c.jpeg
tags: programming-blogs, go, concurrency, goroutines, concurrency-in-go

---

### Cond

Sometimes, goroutines need to wait for some data or some kind of signal to be available before executing further. Cond primitive provides the most efficient way to just do that.

***Cond implements a condition variable, a rendezvous point for goroutines waiting for or announcing the occurrence of an event. - sync package doc.***

Cond is implemented as a struct with `Locker` field, and implements 3 methods.

The `Broadcast()` method of `Cond` wakes up all goroutines that are waiting on the condition `c`. The caller may or may not hold the associated mutex `c.L` while calling this method.

Similarly, the `Signal()` method of `Cond` wakes up one goroutine that is waiting on a condition `c`, if any.

The `Wait()` method of `Cond` atomically releases the associated mutex `c.L` and suspends the execution of the calling goroutine. After resuming the execution, the method re-acquires the mutex before returning. Unlike in other systems, the `Wait()` the method cannot return unless it is awoken by a call to `Broadcast()` or `Signal()`.

Let's see an example:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var done bool

func reader(i int, c *sync.Cond, wg *sync.WaitGroup) {
	fmt.Printf("goroutine number %d started\n", i+1)
	defer wg.Done()
	c.L.Lock()
	defer c.L.Unlock()
	for !done {
		fmt.Printf("goroutine number %d waiting\n", i+1)
		c.Wait()
	}
	fmt.Printf("goroutine number %d finished\n", i+1)
}

func writer(c *sync.Cond, wg *sync.WaitGroup) {
	fmt.Println("Writer goroutine started")
	defer wg.Done()
	time.Sleep(5 * time.Second)
	c.L.Lock()
	defer c.L.Unlock()
	done = true
	c.Broadcast()
	fmt.Println("Writer goroutine is done")
}

func main() {
	var wg sync.WaitGroup
	m := &sync.Mutex{}
	c := sync.NewCond(m)
	for i := 0; i < 2; i++ {
		wg.Add(1)
		go reader(i, c, &wg)
	}
	wg.Add(1)
	go writer(c, &wg)
	wg.Wait()
}
```

```bash
➜ go run main.go
Writer goroutine started
goroutine number 2 started
goroutine number 2 waiting
goroutine number 1 started
goroutine number 1 waiting
Writer goroutine is done
goroutine number 1 finished
goroutine number 2 finished
```

[**Run this code in Go Playground**](https://goplay.tools/snippet/HKfDuXv3vy0)

In the above example, we wait on a condition `for !done{}`. All the reader goroutines will be suspended until the writer goroutines update the variable `done=true` and send the waking-up signal to all suspended goroutines.

### Once

The `sync.Once` type in Go is used to perform a certain operation exactly once in a concurrent setting, regardless of how many times it is called from different goroutines.

The `sync.Once` type guarantees that the operation is executed only once, even if multiple goroutines attempt to call it concurrently. This is useful when you need to initialize a shared resource that is expensive to create or when you want to ensure that a particular task is executed only once.

The `sync.Once` the type has a single method, `Do(f func())`, that takes a function `f` as an argument. The first call to `Do()` will execute the function `f`, and subsequent calls will do nothing. The `sync.Once` type internally maintains a flag that indicates whether `f` has been executed or not.

Let's see an example:

```go
package main

import (
	"fmt"
	"sync"
)

func DBinit() {
	fmt.Println("Initializing database...")
}

func main() {
	var wg sync.WaitGroup
	var once sync.Once

	for i := 1; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			once.Do(DBinit)
			fmt.Println("Database initialized.")
		}()
	}
	wg.Wait()
}
```

```bash
➜  go run main.go
Initializing database...
Database initialized.
Database initialized.
Database initialized.
Database initialized.
```

[**Run this code in Go Playground**](https://goplay.tools/snippet/3mG4X8f51GT)

### Pool

The `sync.Pool` type in Go enables caching of temporary objects that can be utilized again across multiple goroutines. Essentially, the `sync.Pool` maintains a finite set of objects that are assigned to the goroutine, and once a goroutine finishes utilizing an object, it is returned to the pool for reuse by another goroutine.

The `sync.Pool` type works by maintaining a pool of objects that can be accessed and reused by multiple goroutines. We can request new resource objects using `Get()` the method of `sync.Pool`. It first checks if there are any objects in the pool. If there are, it returns one of the objects. If not, it creates a new object and returns it. When an object is no longer needed, it can be put back into the pool for future use using `Put(x interface{})` the method.

Using `sync.Pool` can significantly reduce the number of objects that need to be garbage collected, which can improve overall performance in Go programs. However, it is important to use `sync.Pool` judiciously, as it is not always beneficial and can sometimes even hurt performance if not used correctly.

Here's an example:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	var bufCount int

	bufPool := &sync.Pool{
		New: func() any {
			bufCount++
			buf := make([]byte, 1024)
			return &buf
		},
	}

	for i := 0; i < 1000000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			buf := bufPool.Get().(*[]byte)
			bufPool.Put(buf)
		}()
	}

	wg.Wait()
	fmt.Println(bufCount)
}
```

```bash
➜  go run main.go
9
```

[**Run this code in Go Playground**](https://goplay.tools/snippet/iXTjC2CWv-Y)

Next, a `sync.Pool` is created using the `New` field, which is a function that creates a new object when the pool is empty. In this case, the New function creates a new `byte slice with a length of 1024` and returns a pointer to the slice. The function also increments the `bufCount` variable to keep track of the number of objects created.

Then, a loop is started that creates `1 million goroutines`. Each goroutine calls the sync.Pool's `Get` method to retrieve a byte slice from the pool. In the end, the goroutine calls the sync.Pool's `Put` method to return the byte slice to the pool.

***If this program were written without using sync.Pool***, each goroutine would create its own byte buffer whenever it needed one. This would result in a lot of unnecessary memory allocation and garbage collection, which can be costly in terms of performance.

***Thank you for reading this blog, and please give your feedback in the comment section below.***