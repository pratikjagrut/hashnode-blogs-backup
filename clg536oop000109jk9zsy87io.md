---
title: "Concurrency in Go (Part-2): sync package primitives"
datePublished: Thu Apr 06 2023 12:20:45 GMT+0000 (Coordinated Universal Time)
cuid: clg536oop000109jk9zsy87io
slug: concurrency-in-go-part-2-sync-package-primitives
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/G2lgiBBzeEM/upload/0f1a0ee43752f4e38452b54d3124dc2e.jpeg
tags: programming-blogs, go, concurrency, goroutines, concurrency-in-go

---

### Introduction

Go is primarily designed around ***Communicating Sequential Processes*** i.e using channels and select primitives to write concurrent programs, but it also supports the traditional way of writing concurrent code using ***Memory Access Synchronization***.

CSP primitives are sufficient for writing concurrent programs in most cases, but if the need arises to do low-level memory access synchronization, the `sync package` provides primitives for it. The sync package is part of the Go standard library.

sync package's primitives are:

* *WaitGroup*
    
* *Mutex & RWMutex*
    
* *Cond*
    
* *Once*
    
* *Pool*
    

### WaitGroup

WaitGroup provides an alternative to using channels and the select statement to wait for goroutines to finish their execution. WaitGroup is preferred when the results of the concurrent process are not important or we've other ways of collecting results from goroutines.

Here's an example of using WaitGroup to block the main goroutine.

```go
package main

import (
	"fmt"
	"net/http"
	"sync"
)

func fetchURL(url string, wg *sync.WaitGroup) {
	defer wg.Done()
	resp, err := http.Get(url)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()
	fmt.Println("URL hit: ", url)
}

func main() {
	var wg sync.WaitGroup
	urls := []string{
		"https://www.google.com",
		"https://www.youtube.com",
		"https://www.amazon.com",
		"https://www.linkedin.com",
	}
	fmt.Println("Start executing goroutines!")
	for _, url := range urls {
		wg.Add(1)
		go fetchURL(url, &wg)
	}
	fmt.Println("Outside of for loop")
	wg.Wait()
	fmt.Println("All goroutines executed successfully!")
}
```

```bash
➜  go run main.go
Start executing goroutines!
Outside of for loop
URL hit:  https://www.youtube.com
URL hit:  https://www.google.com
URL hit:  https://www.amazon.com
URL hit:  https://www.linkedin.com
All goroutines executed successfully!
```

In the program above, the order of execution of the goroutines is visible in the output. First, the statement before the for loop was executed. Then, inside the for loop, we created four goroutines and before goroutines could finish executing, the for loop exited and the statement after the for loop was executed.

In the above program, we used WaitGroup to synchronise the goroutine execution and block the main goroutine. We incremented the counter by calling `wg.Add(1)` each time we created a new goroutine. Once a goroutine is completed, we decremented the counter by calling `wg.Done()`. We used `wg.Wait()` to pause the main goroutine. `wg.Wait()` stops the program from continuing until the counter reaches 0, indicating that all goroutines had finished executing.

### Mutex & RWMutex

The simultaneous access of shared resources by multiple parallel processes can result in inconsistency in the data or the outcomes of those processes. This inconsistency is referred to as a Race condition.

For example: If a program has multiple goroutines, which are trying to read and write the shared variable. In this case, there is no way to know which goroutine will access the variable first. It may happen that one goroutine is reading that variable and at the same time, another goroutine updates the data on the variable, this may lead to incorrect behaviour of the program. This situation is called a Data Race.

Let's understand it with code:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	count := 0
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			count++
			wg.Done()
		}()
	}
	wg.Wait()
	fmt.Println("Final Count: ", count)
}
```

In the above example, we've got a count variable that is accessed by all the routines. Now when we run the program we see a different final count. But as we were running for loop 1000 times we should get a count equal to 1000, but that's not the case. There is some inconsistency in the data.

```bash
➜  go run main.go       
Final Count:  981
➜  go run main.go
Final Count:  985
➜  go run main.go
Final Count:  979
```

[Run this code in Go Playground](https://goplay.tools/snippet/hpMwsnxvcDi)

In Go, channels are the preferred way to share data by communicating, but if you must use the shared memory then we need to make sure that Memory Access is Synchronised.

#### Mutex

To solve the above issue we use `Mutex`. Mutex stands for mutual exclusion. Mutex is a way to guard or control the Memory Access Synchronization.

In Go, Mutex is implemented as a struct that contains a flag that indicates whether the lock is currently held by a goroutine or not. If the lock is not held, a goroutine can acquire the lock by calling the `Lock()` method on the Mutex. If the lock is already held by another goroutine, the calling goroutine will be blocked until the lock is released.

When a goroutine holds the lock, it has exclusive access to the shared resource and can modify it without any interference from other goroutines. Once the goroutine is done with the shared resource, it can release the lock by calling the `Unlock()` method on Mutex.

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	var lock sync.Mutex

	count := 0
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			lock.Lock()
			count++
			lock.Unlock()
			wg.Done()
		}()
	}
	wg.Wait()
	fmt.Println("Final Count: ", count)
}
```

```bash
➜  go run main.go
Final Count:  1000
➜  go run main.go
Final Count:  1000
➜  go run main.go
Final Count:  1000
```

[Run this code in Go Playground](https://goplay.tools/snippet/ECUZm6CC8XC)

In the above example, we're using a mutex, we're locking and unlocking our critical section of code. We apply a lock before the goroutine accesses the count variable. This way only one goroutine can access the count variable and other goroutines will be waiting for the count variable to be unlocked to gain access. This is how mutex synchronises the shared memory access.

Mutex is useful for scenarios where a shared resource needs to be accessed by multiple goroutines in a mutually exclusive manner. However, ***it can be inefficient in scenarios where there are many read operations and fewer write operations, as it allows only one goroutine to hold the lock at a time, regardless of whether it is reading or writing***. In such scenarios, Go provides a more efficient primitive called RWMutex.

#### RWMutex

The Go's sync package also provides RWMutex with traditional Mutex. RWMutex stands for ***Reader/Writer mutual exclusion***.

***The lock can be held by an arbitrary number of readers or a single writer. - sync package doc.***

It is a type of lock that allows multiple readers to access a shared resource concurrently, but only one writer to modify it at a time.

When a reader wants to access the shared resource, it first calls the `RLock()` method on the RWMutex. This method acquires a read lock, which allows multiple readers to access the resource simultaneously. To release the lock it calls the `RUnlock()` method. If a writer holds the lock, the reader must wait until the writer releases it.

When a writer wants to modify the shared resource, it calls the `Lock()` method on the RWMutex. This method acquires a write lock, which prevents other readers and writers from accessing the resource until the writer calls `Unlock()` and releases the lock. If readers are holding the lock, the writer must wait until all readers have released it.

Let's see an example:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	m := &sync.RWMutex{}
	var wg sync.WaitGroup

	for i := 0; i < 5; i++ {
		wg.Add(1)
		go readerFunction(&wg, m)
	}
	wg.Add(1)
	go writerFunction(&wg, m)

	wg.Wait()
}

func writerFunction(wg *sync.WaitGroup, m *sync.RWMutex) {
	defer wg.Done()
	fmt.Println("Writer acquires lock")
	m.Lock()
	m.Unlock()
}

func readerFunction(wg *sync.WaitGroup, m *sync.RWMutex) {
	defer wg.Done()
	fmt.Println("Reader acquires lock")
	m.RLock()
	m.RUnlock()
}
```

```bash
➜  go run main.go
Reader acquires lock
Writer acquires lock
Reader acquires lock
Reader acquires lock
Reader acquires lock
Reader acquires lock
```

[Run this code in Go Playground](https://goplay.tools/snippet/nooyW5TDco1)

#### Mutex VS RWMutex which one to use?

In scenarios where more read operations need to be performed, then RWMutex is an efficient choice because it lets multiple reader goroutines acquire locks simultaneously. This reduces the waiting time for reader goroutines significantly and expedites the overall execution of the program.

Following is an example that shows the difference in execution time for Mutex and RWMutex programs:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	m := &sync.Mutex{}
	var wg sync.WaitGroup

	start := time.Now()
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go readerFunction(&wg, m)
	}
	wg.Add(1)
	go writerFunction(&wg, m)

	wg.Wait()
	fmt.Println("Total execution time: ", time.Since(start))
}

func writerFunction(wg *sync.WaitGroup, m *sync.Mutex) {
	defer wg.Done()
	m.Lock()
	m.Unlock()
}

func readerFunction(wg *sync.WaitGroup, m *sync.Mutex) {
	defer wg.Done()
	m.Lock()
	time.Sleep(1 * time.Second)
	m.Unlock()
}
```

```bash
➜  go run main.go
Total execution time:  3.003424166s
```

[Run this code in Go Playground](https://goplay.tools/snippet/N_SeF4wJtk2)

The above program uses Mutex for synchronisation and it takes `3.003424166s` to execute. Let's try the same program with RWMutex.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	m := &sync.RWMutex{}
	var wg sync.WaitGroup

	start := time.Now()
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go readerFunction(&wg, m)
	}
	wg.Add(1)
	go writerFunction(&wg, m)

	wg.Wait()
	fmt.Println("Total execution time: ", time.Since(start))
}

func writerFunction(wg *sync.WaitGroup, m *sync.RWMutex) {
	defer wg.Done()
	m.Lock()
	m.Unlock()
}

func readerFunction(wg *sync.WaitGroup, m *sync.RWMutex) {
	defer wg.Done()
	m.RLock()
	time.Sleep(1 * time.Second)
	m.RUnlock()
}
```

```bash
➜  go go run main.go
Total execution time:  1.0011145s
```

[Run this code in Go Playground](https://goplay.tools/snippet/jIVoKdkdoyD)

The above program uses RWMutex for synchronisation and it takes `1.0011145s` to execute.

The difference is the execution time is quite significant.

It is important to use RWMutex judiciously because it may not be appropriate for all types of shared resources. For example, if the shared resource is small and is accessed frequently, the overhead of acquiring and releasing locks may outweigh the benefits of using RWMutex. In such cases, other synchronization mechanisms like atomic operations or channels may be more appropriate.

*In the next blog, we'll look into* ***cond, once and pool*** *primitives provided by the sync package.*

***Thank you for reading this blog, and please give your feedback in the comment section below.***