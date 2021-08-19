---
title: "Concurrency"
linkTitle: "Concurrency"
weight: 1 
description: >
  This topic is to explain how to manage share variables between different goroutines
---

## Concurrency in a nutshell

Two events in two or more goroutines happen in a non-specific order: in such cases we are talking about concurrency.
When a particular function continues to work correctly even if it is invoked by two different goroutines, we say that it is concurrency safe.
The gold rule says that we need to access concurrently to a variable only if the corresponding documentation says that this variable is concurrency safe.

### What is a race condition?

A race condition is a situation where the result of concurrency calls are not correct.
For example a **data race** occurs whenever two goroutines access the same variable concurrently and at least one of the accesses is a write.
Suppose you have a `total` variable set to 10. The following program illustrates a sequential routine:

```go
var total int = 10

func main() {
  add(100)
  add(200)
  printTotal()
}

func add (a int){
  total += a
}

func printTotal(){
  fmt.Println("total is: ", total)
}
```

The `total` of this program will always be 310.
If you change as:

```go
var total int = 10

func main() {
  go add(100)
  go add(200)
  time.Sleep(100 * time.Millisecond)
  printTotal()
  time.Sleep(100 * time.Millisecond)
}

func add (a int){
  total += a
}

func printTotal(){
  fmt.Println("total is: ", total)
}
```

You cannot be absolutely sure of that result because the access to the total is not deterministic. If both the sub routines access to the value before it is written from one routine the value will be wrong:
r1 access and get the total = 10
the r1 add 100
the r2 access to total before 100 is written, get 10 and add 200. The total will be 210
the r1 changes total to 110
It is not easy to simulate but it could be possible to happen.

### How to avoid the data race

There are some possibilities to avoid the race condition:

- you can **write your variable only once**: if you create and initialize the variable at a package level, this operation comes before the main function is called. The func that the goroutines can call concurrently, will be a read function.
- you can **write your variable by using only one goroutine**: if you access and modify that variable using only one goroutine you can avoid the problem.
       Since other goroutines cannot access the variable directly, they must use a channel to send the confining goroutine a request to query or update the variable.

To better understand the point take a look at the [chat project](https://github.com/mas2020-golang/examples/blob/main/goroutines/chat-server/chat.go).

Follow an example:

```golang
func manager() {
  var sum int = 0
  for {
    fmt.Println("evaluating select...")
    select {
    case amount := <-add:
      sum += amount
      fmt.Println("received an amount of", amount)
    case total <- sum:
    }
  }
}
```

The `manager` func is the only that can change the sum value. The other routines can add a value writing directly into the `add` channel and get the amount reading the `total` channel.
Thanks to the select the manager func responds when one the two cases arises:

- an amount is added into the `add` channel
- `total` channel is read by any go routine

#### Using the Mutual Exclusion: `sync.Mutex`

It means to protect the variable that is subject to change with a lock channel, it means:
create a semaphore buffered channel of cap 1
acquire the channel before to change the value and read the channel after writing
For example:

```golang
var total int
var sema = make(chan struct{},1)

func main() {
  // simulate some operations
  go deposit(10)
  go deposit(10)
  time.Sleep(200 * time.Millisecond)
  fmt.Println("total is:", getTotal())
}

// deposit adds a number to the total using a channel
func deposit(n int) {
  sema <- struct{}{} // acquire the lock
  total += n
  <-sema // release the lock
}
```

The `sync` package is able to do the jobs for us:

```go
import (
 "fmt"
 "sync"
 "time"
)

var mu sync.Mutex
var total int

func main() {
  // simulate some operations
  go deposit(10)
  go deposit(10)
  time.Sleep(200 * time.Millisecond)
  fmt.Println("total is:", getTotal())
}

// deposit adds a number to the total using a channel
func deposit(n int) {
  mu.Lock() // acquire the lock
  total += n
  mu.Unlock() // release the lock
}

// getTotal reads the total using a channel
func getTotal() int {
  mu.Lock() // acquire the lock
  defer mu.Unlock() // release the lock
  return total
}
```

By convention, the variables guarded by a mutex are declared immediately after the declaration of the mutex itself.
It is also convenient, for no risk on error management, to defer the unlock of a mutex:

```go
// deposit adds a number to the total using a channel
func deposit(n int) {
  mu.Lock() // acquire the lock
  defer mu.Unlock() // release the lock
  total += n
}
```

#### Using a special Mutex for the reading

When you need to read a total without waiting for a mutex of a write operation you can use a special Mutex the `sync.RWMutex`, for example instead of using the Mutex you can change with:

```go
var mu sync.RWMutex
```

and the `getTotal` func becomes:

```go
// getTotal reads the total using a channel
func getTotal() int {
  mu.RLock() // acquire the lock
  defer mu.RUnlock() // release the lock
  return total
}
```

It's only profitable to use an RWMutex when most of the goroutines that acquire the lock are readers (it is slower that the usual Mutex so, if in doubt, use an exclusive lock).
As a golden rule: “isolate variables to a single goroutine; for all other variables, use mutual exclusion”.

Sometimes it could happen that you have a variable to initialize only once and to be sure that a func that will retrieve a value is concurrency safe, you can use the `sync.Once` type. For example:

```go
var loadIconsOnce sync.Once
var icons map[string]image.Image

func loadIcons() {
    icons = make(map[string]image.Image)
    icons["spades.png"] = loadIcon("spades.png")
    icons["hearts.png"] = loadIcon("hearts.png")
    icons["diamonds.png"] = loadIcon("diamonds.png")
    icons["clubs.png"] = loadIcon("clubs.png")
}

// Concurrency-safe.
func Icon(name string) image.Image {
    loadIconsOnce.Do(loadIcons)
    return icons[name]
}
```

This example is taken from the book ‘The Go Programming Language Book by Alan A. A. Donovan and Brian Kernighan’.
The `loadIcons` func is called only once, also if multiple goroutines access it.

#### How to detect the race?

Go will be equipped with an easy-to-use dynamic analysis tool, the race detector.
You can add the flag `-race` to the `go build, test, run` to detect some race condition.
