---
title: "Go routines"
linkTitle: "Go routines"
weight: 1
description: >
    This topic is to explain what are and how to use the go routines
---

## What is a go routine?
> A goroutine is a lightweight thread managed by the Go runtime.

Adding `go` before calling a function or a method causes the function to be executed in a newly created
go routine.
Take a look at this simple example:
```go
func main() {
	go counter(1 * time.Second)
	time.Sleep(3 * time.Second)
	fmt.Println()
}

func counter(delay time.Duration) {
	for {
		for _, r := range `1234567890` {
			fmt.Printf("\r%c/%d seconds", r,len(`1234567890`))
			time.Sleep(delay)
		}
	}
}
```
What's happening? Main function is executed in the _**main**_ go routine. This routine is associated with the application at its
lifetime corresponding to the life of the program.
The `counter` function instead is executed in a separate go routine. The `counter` go routine will terminate if the main
go routine terminates. The `time.Sleep(3 * time.Second)` is a statement to block the main routine so that counter can write
its output. If you remove the statement, the execution is so fast that the main terminates and you could not see any output
from the counter function.
{{% alert title="IMPORTANT" color="warning" %}}
Goroutines run in the same address space, so access to shared memory must be synchronized.
{{% /alert %}}

### How to wait for a go routine to end?
In the previous chapter we have seen a use case where we simply inform the user on the elapsed time. But, in case we need
to wait for a go routine to end to read the result or do something else?
We need to introduce `channels`. They are a reference type (as `slice` and `map`) and represent a way to communicate
between two or more goroutines.

Through the channels it is possible for a go routine to send values to another. The channel is a conduit of a
particular type.
`chan int` means a channel of an int type as element.

Copying a channel or passing one as an argument to a function, the reference is passed: an update will reflect on the caller
and callee as they refer to the same data structure.
* The zero value of a channel is nil.
* comparison between two channel is true if they refer to the same data structure
* to send a value through the channel: `channel <- value operands`
* to receive a value from a channel: `<-channel operand`

Examples on  how to use the channel:
```go
// create a channel of string type
c := make(chan string)
c <- "hello" // to send
x := <-c // to receive
<-c // receive and discarding result
```
{{% alert title="Tip" color="success" %}}
To close a channel use close(channel) but remember that:
- send data on a closed channel **will panic**
- receive on a close channel yields values until no more values are left, then it continues to receive the ZERO values
for the channel type
{{% /alert %}}

There are two types of channel:
- 0 capacity, called `unbuffered`
- with capacity, called `buffered`
  You create the corresponding type passing the capacity arg to the make function.

#### Using an `unbuffered channel`
This is a channel with capacity 0. When you use this type of channel keep in mind that:
- send operation on `unbuffered channel` (with zero capacity) blocks the sending goroutine until another
  goroutine receives the data. From this point both goroutine can continue.
- if the receive operation was attempted first, the receiving goroutine is locked until another doesn't send data
- both goroutines on `unbuffered channel` are synchronous.
- when a value is sent on an unbuffered channel, the receipt of the value happens before the reawakening of the
  sending goroutine
- when a go routine is locked we talk about a goroutine _leak_. Consider that a go routine consumes resources and
share the same address space of the other routines, so it's better to avoid this and ensure that the go routines
  will end their job
  
Follow an example:
```go
func main() {
	done := make(chan struct{})
	// invoke single job as go routine
	go singleJob(done)
	fmt.Println("I am waiting the end of singleJob...")
	// lock the routine until a message is read from the done channel
	<-done
	fmt.Println("I can terminate the application, bye in 3 sec...")
}
```
in the code above we create a `done` channel of type `struct{}` that is simply an empty base struct (the type is not
important, we could use int or bool instead). Then `singleJob` is executed in a separate go routine.
Then the main go routine is locked because it waits to receive data on an `unbuffered channel`.
In case the value is never received the routine will lock forever. Let's take a look at the `singleJob` function:
```go
func singleJob(done chan struct{}) {
	// wait 2 seconds (simulating some jobs to do)
	time.Sleep(2 * time.Second)
	// write into the channel
	done <- struct{}{}
}
```

#### Using a `buffered channel`
It is a channel with capacity > 0. It is a queue where it is possible to write until the queue is full.
When you use this type of channel keep in mind that:
- send operation on `buffered channel` doesn't blocks the sending goroutine until the queue is full
- reading from this channel doesn't lock the reader routine until the queue is empty

Follow an example:
```go
func main() {
	fmt.Println("Main routine start...")
	fmt.Println("Mirrored query = ", mirroredQuery())
	time.Sleep(2 * time.Second)
	fmt.Println("Main routine end")
}
```
The above code contains a simple main function executed on the main go routine that calls the `mirroredQuery` function:
```go
func mirroredQuery() string {
	responses := make(chan string, 2)
	go func() {
		responses <- request("asia.gopl.io")
		fmt.Println("end go routine asia.gopl.io")
	}()
	go func() {
		responses <- request("europe.gopl.io")
		fmt.Println("end go routine europe.gopl.io")
	}()
	return <-responses // return the quickest response
}
```
In the code above we have:
* a string channel with capacity == 2 called `responses`
* two anonymous function executed as two separate goroutines

Each function will execute the `request` function that returns a string (omitted for brevity) and send the value
into the responses channel. The channel has a capacity of 2, so the two goroutines can write into it and terminate.
The return is locked until a value is written on the channel. The first routine that will write on the response will
cause the end of the program.

### Multiplexing with select
The `select` statement is a way to wait for completion from more channels.
Think of the case where we have two channels, and we need to catch the value from each other. The first we listen will
eventually block a value received on the other. The way to accomplish that is using `select`.
`select` waits until a communication for some case is ready to proceed.
It then performs that case executing the associated statements; the other communications do not happen.
Using the `default` clause means to specify what to do if none of the others cases can proceed immediately.
Usually the `select` statement is used for polling.
{{% alert title="IMPORTANT" color="warning" %}}
When multiple cases are valid at the same time the select will pick one of them randomly.
{{% /alert %}}
```go
func main() {
	abort := make(chan struct{})
	// Separate go routine listening that sends an empty structure on the abort channel in
	// case the user will press enter
	go func() {
		os.Stdin.Read(make([]byte, 1)) // read a single byte
		abort <- struct{}{}
	}()
	go count(time.Second * 1)

	// this code is executed on the main go routine and will block until a case is not satisfied: in that case
	// it will execute the corresponding statements and the will go on to the following statements (if any) after
	// the select
	fmt.Println("Press return to abort (you have three seconds)...")
	select {
	case <-time.After(3 * time.Second):
		fmt.Println("\nThree seconds lasted without any abort signal")
	case <-abort:
		fmt.Println("Launch aborted!")
		return
	}
	launch()
}
```
Key points of the above code are:
* we have an `abort` channel where we write in case the user press ENTER (this is accomplished with an anonymous function
  executed as a separate go routine)
* we call in a separate go routine a counter function that simply show the elapsed time on the standard output
* on the main go routine we have a select function that waits for one of the two cases:
    - `time.After` is a function available on time package that returns a chan of Time with the current Time after the
    timeout you specify
    - `<-abort` is executed if data are written into the abort channel
* if no abort is sent into the channel, after 3 seconds the first case is executed and the select statement is done. The
flow proceed towards the execution of the launch statement.

### Close a channel
In some cases it could be useful to notify the other routines that a channel has been closed. Pay attention that:
- send data on a nil channel causes panic
- read data from a closed channel causes, after the channel has been drained, to receive a ZERO value for the channel
type (if the statement is in a loop means to receive forever that value)
  
Take a look at the following code (I add time.Sleep only to better follow the output):
```go
func main() {
	var c = make(chan int)
	go doSomething(c)
	for {
		i := <-c
		fmt.Printf("read %d from channel...\n", i)
		time.Sleep(1 * time.Second)
	}
}
```
in the above `main` function:
- create a c channel
- executes a doSomething function in a separate go routine
- for (infinite) loop that reads from the c channel

Let's take a look at the `doSomething` code:
```go
func doSomething(c chan int) {
	for i := 0; i < 2; i++ {
		if i == 0 {
			fmt.Println("I'm doing something...", i)
			time.Sleep(1 * time.Second)
			c <- i
		}
		if i == 1 {
			fmt.Println("error close the channel...", i)
			time.Sleep(1 * time.Second)
			close(c)
			break
		}
	}
}
```
In case of `i == 0` the code writes `i` into the channel, and the main go routine will read it, then doSomething goes to
another value and closes the channel exiting from the function (and terminating the go routine). But, something terrible
happens, let's take a look to the output:
```
I'm doing something... 0
error close the channel... 1
read 0 from channel...
read 0 from channel...
read 0 from channel...
```
closing the channel determines that the main go routine will read forever the ZERO value from the channel. How can
we stop it?

A way it to check the value read from the channel as:
```go
func main() {
	var c = make(chan int)
	go doSomething(c)
	for {
		if i, ok := <-c; ok {
			fmt.Printf("value read from channel is %d\n", i)
			time.Sleep(1 * time.Second)
		}else{
			break
		}
	}
	fmt.Println("Main go routine end")
}
```
As you can see the check is similar to the type assertion. In case `ok` is true there is a valid value, otherwise
we can terminate the application.
We can rewrite using the `range` statement in a more compact way:
```go
func main() {
	var c = make(chan int)
	go doSomething(c)
	for i := range c {
		fmt.Printf("value read from channel is %d\n", i)
		time.Sleep(1 * time.Second)
	}
	fmt.Println("Main go routine end")
}
```
Some use cases when could be convenient use the `close` statement could be:
- stop reading of a channel in case of any error
- notify the end of the calculation in a go routine in case we use the go routine to notify the current point of the 
execution: think to have a routine that checks file in a directory and return the size of every single file read 
