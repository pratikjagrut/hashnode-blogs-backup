---
title: "Concurrency in Go (Part-1): Goroutines, Channels and Select"
datePublished: Wed Mar 15 2023 13:08:39 GMT+0000 (Coordinated Universal Time)
cuid: clf9p7k3i000k09jxfjy8am6o
slug: concurrency-in-go-part-1-goroutines-channels-and-select
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/o0HrQE7bNHM/upload/e2f0abd01045a5e0779996b4ed4c076f.jpeg
tags: programming-blogs, go, concurrency, goroutines, concurrency-in-go

---

### Introduction

Concurrency, asynchronous, parallel and threaded are some keywords we see floating around when we talk about running multiple processes using multiple resources. Although these terms may appear interchangeable in English, they have distinct differences in computer language, particularly at the compiler level and during runtime execution, as well as in their underlying philosophies.

In the software development world, concurrency and parallelism are considered the same and used interchangeably while discussing them. Concurrency and parallelism aren't the same things. They may have some overlapping similarities but still, they're different. ***Concurrency is dealing with a lot of things at once and parallelism is doing a lot of things at once***.

Concurrency is at the code level, it is a way of structuring the code so that it may use parallelism to run but not necessarily. We can not write parallel code we can only write concurrent code hoping it will run parallel.

For example, if you write a chat application then it needs to be able to handle each message from each user independently without affecting other users. So you structure your program in such a way that employs concurrency. Now suppose you've got a single-core CPU, in this case, the second message will have to wait until the first message is processed but if you've 2 core CPUs then both messages will be processed parallelly.

### Go's philosophy of concurrency

It is very important to understand the thought behind Go's pattern of achieving concurrency before we began to write concurrent code.

Go is a concurrent language. Concurrency primitives are built in Go's core, so it implicitly supports concurrency. Go was designed around CSP. CSP stands for ***Communicating Sequential Processes***. It was introduced by ***Charles Antony Richard Hoare*** in 1978 in a paper published by the same name at Association for Computing Machinery(ACM).

### Communicating Sequential Processes

CSP works on three simple rules:

* #### Write sequential code
    
    Writing parallel code is not an easy task. We've to use a significant amount of brain power to understand the problem and write in a way that it'll surely run in parallel without getting into any parallel processes running traps such as deadlock or livelock. Instead writing sequential code is easy. We've been doing it all the time.
    
* #### Processes will have local state and no shared state
    
    One of Go's mottos is, ***Share memory by communicating, don't communicate by sharing memory***. So every process will have its local state and no shared state but if the process has to share data with another process then communicate that data instead of using shared memory. This means a process will send a copy of data to another process.
    
* #### Scale by adding the same processes
    
    So once you have a sequential code that can be run independently, then add more of those processes to scale your software.
    

Go is primarily designed around these CSP primitives but it also supports the traditional way of writing concurrent code using ***Memory Access Synchronization***.

### Concurrency building blocks in Go

To write a concurrent program, Go provides us with a toolset:

* Goroutines
    
* Channels
    
* Select
    
* Sync package
    

### Goroutines

Go follows a concurrency model called `fork-join`***.*** Fork-join means that a program can create a separate branch of execution, or child process, that runs concurrently with its parent process and will eventually rejoin the parent process at a later point in time.

In Go, this forked process or child process can be called a goroutine.

Goroutines are unique to Go. It is one of the basic units of the Go program. It is a lightweight thread of execution that enables concurrent programming.

Goroutines are neither the OS thread nor the green thread, they're abstraction over coroutines. Coroutines are concurrent subroutines that can not be interrupted, they can be suspended or reentered through multiple points in execution. Goroutines are lightweight, cheap and faster at execution than thread.

Every Go program has at least one Goroutine, i.e `main goroutine`***.***

`go` keyword is used to fork a child i.e to create a goroutine in the program.

Here's an example of a goroutine:

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func fetchURL(url string) {
	resp, err := http.Get(url)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("URL hit: ", url)
	defer resp.Body.Close()
}

func main() {
	urls := []string{
		"https://www.google.com",
		"https://www.youtube.com",
		"https://www.amazon.com",
		"https://www.linkedin.com",	
    }

	for _, url := range urls {
		go fetchURL(url)
	}
	time.Sleep(2 * time.Second)
}
```

If you run the above program you'll see the output as follows.

```bash
➜ go run main.go
URL hit:  https://www.youtube.com
URL hit:  https://www.google.com
URL hit:  https://www.linkedin.com
URL hit:  https://www.amazon.com
```

A ***function*** called with a `go` keyword, that's all we need to create a block of code that runs concurrently. This is how the first rule of CSP is satisfied, write a sequential process that is capable of running independently.

In the above example, we're creating a goroutine for each URL. `time.Sleep()` is used to block the `main goroutine` until all the goroutines are finished execution. Go provides better features, such as `channels` and `sync package` to block or to make the parent process wait for goroutines.

### Channels

Channels in Go are synchronization primitives of CSP. Channels are widely used for communicating data between goroutines.

Channels can be thought of as a river. Similarly how river functions, a channel functions, as a means of conveying a continuous flow of information. They're typed conduits through which goroutines can safely and efficiently send and receive data.

Creating a channel is very easy. You've to declare a chan variable with the datatype.

Here's an example:

```go
var chanName chan interface{}
chaneName := make(chan interface{})
```

The above example of creating a bidirectional channel, i.e we can write to or read from the channel but Go also allows us to create a unidirectional channel.

To create a unidirectional channel we need to use the `<-` operator. The position of the `<-` operator decides if the channel is a read channel or a write channel.

Here's an example of a channel that can only read or receive.

```go
var chanName <-chan interface{}
chaneName := make(<-chan interface{})
```

Here's an example of a channel that can send or write data.

```go
var chanName chan<- interface{}
chaneName := make(chan<- interface{})
```

Unidirectional channels are not often instantiated instead, they're used as parameters and return types of functions.

Here's an example:

```go
package main

import (
	"fmt"
	"net/http"
)

func fetchURL(url string, done chan<- bool) {
	resp, err := http.Get(url)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("URL hit: ", url)
	defer resp.Body.Close()
	done <- true
}

func main() {
	urls := []string{
		"https://www.google.com",
		"https://www.youtube.com",
		"https://www.amazon.com",
		"https://www.linkedin.com",
	}
    // Create a boolean channel
	done := make(chan bool)
    // Close channel after use
	defer close(done)
    // Create goroutines
	for _, url := range urls {
		go fetchURL(url, done)
	}
    // Block the main goroutine execution
	for i := 0; i < len(urls); i++ {
		<-done
	}
}
```

This is the same example from the goroutine section. Here we used channels to block the execution of the main goroutine. Channel done is declared as a bidirectional channel in the main but we passed it as a unidirectional channel (only send/write). Go implicitly converted the bidirectional channel to a unidirectional channel.

Method `close()` is used to close the channel after use. When a closed channel is written to, a panic occurs.

```bash
panic: send on closed channel
```

Closing a channel is not strictly necessary, and there are situations where it is not needed. However, it is a good practice to close channels when they are no longer needed, as it allows the receiving goroutines to exit gracefully and can prevent memory leaks.

#### Buffered channel

In Go, a buffered channel is a channel with a fixed capacity that allows multiple values of data to be stored. Buffered channels allow us to make ***n*** writes, where n is capacity defined, regardless of whether it's being read or not. We can create a buffered channel by passing capacity in ***integer*** value to make a function. A buffered channel is an in-memory FIFO queue for concurrent processes to communicate.

Here's an example of a buffered channel:

```go
package main

import (
	"fmt"
	"net/http"
)

func fetchURL(urls []string, done chan<- string) {
	for _, url := range urls {
		resp, err := http.Get(url)
		if err != nil {
			fmt.Println("Error:", err)
			return
		}
		defer resp.Body.Close()
		done <- fmt.Sprintln("URL hit: ", url)
	}
	close(done)
}

func main() {
	urls := []string{
		"https://www.google.com",
		"https://www.youtube.com",
		"https://www.amazon.com",
		"https://www.linkedin.com",
	}
	done := make(chan string, len(urls))
	go fetchURL(urls, done)

	for v := range done {
		fmt.Println(v)
	}
}
```

In the above example, we created a buffered channel of type string and capacity equal to the length of the slice, in this case, 4. We can use `range` to fetch the data from the buffered channel. The important thing to notice is that when we use range to fetch data the buffered channel has to be closed once data is populated otherwise for loop will keep on waiting forever.

### Select statement

In Go, the select statement is like a switch statement but for channels. It's one of the important features of Go's concurrency. It is used to choose from multiple channel operations that are waiting for information to be sent or received on a particular channel.

Here's a code block showing how to use a select statement.

```go
var c1, c2 <-chan int
select {
case <- c1:
    // Do something
case <- c2:
    // Do something
case <-time.After(2 * time.Second):
		fmt.Println("Timed out.")
        return
default:
    // Do someting until all channels are blocked
}
```

Even though the select statement looks like a switch statement but its execution is very different. Cases in the select statement do not execute sequentially, instead, all channels are considered simultaneously to check if any of them is ready. If none of the channels is ready then the default block is executed over and over again.

Here's an example of the select statement:

```go
package main

import (
	"fmt"
	"net/http"
)

func fetchURL(url string, urlChan chan<- string) {
	resp, err := http.Get(url)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()
	urlChan <- fmt.Sprintln("URL hit first: ", url)
}

func main() {
	urls := []string{
		"https://www.google.com",
		"https://www.youtube.com",
	}

	googleChan := make(chan string)
	youtubeChan := make(chan string)

	channels := []chan string{googleChan, youtubeChan}

	for i, url := range urls {
		go fetchURL(url, channels[i])
	}

	select {
	case msg := <-googleChan:
		fmt.Println(msg)
	case msg := <-youtubeChan:
		fmt.Println(msg)
	}
}
```

In the above program, we're trying to hit URLs and then using select to block the execution of the main goroutine. Whichever channel is ready that case will be executed.

```bash
➜  go run main.go
URL hit first:  https://www.youtube.com
```

We can use an infinite for loop to keep on waiting for all channels to be ready or till time out.

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func fetchURL(url string, urlChan chan<- string) {
	resp, err := http.Get(url)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()
	urlChan <- fmt.Sprintln("URL hit: ", url)
}

func main() {
	urls := []string{
		"https://www.google.com",
		"https://www.youtube.com",
		"https://www.amazon.com",
		"https://www.linkedin.com",
	}

	googleChan := make(chan string)
	youtubeChan := make(chan string)
	amazonChan := make(chan string)
	linkedinChan := make(chan string)

	channels := []chan string{
		googleChan,
		youtubeChan,
		amazonChan,
		linkedinChan,
	}

	for i, url := range urls {
		go fetchURL(url, channels[i])
	}

	for {
		select {
		case url := <-googleChan:
			fmt.Println(url)
		case url := <-youtubeChan:
			fmt.Println(url)
		case url := <-amazonChan:
			fmt.Println(url)
		case url := <-linkedinChan:
			fmt.Println(url)
		case <-time.After(2 * time.Second):
			fmt.Println("Timed out.")
			return
		}
	}
}
```

```bash
➜ go run main.go
URL hit:  https://www.youtube.com
URL hit:  https://www.linkedin.com
URL hit:  https://www.google.com
URL hit:  https://www.amazon.com
Timed out.
```

### Sync package

In addition to using channels, Go also allows for the more traditional approach of writing concurrent programs that utilize memory access synchronization. The `sync` package offers various primitives, including `WaitGroup`, `Mutex`, `RWMutex`, `Cond`, `Once`, and `Pool`.

`We will explore the sync package in greater detail in a separate blog post.`

***Thank you for reading this blog, and please give your feedback in the comment section below.***