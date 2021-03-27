---
title: "Using Interfaces"
linkTitle: "Using Interfaces"
weight: 4
description: >
  What they are and how to use them
---

## What is an interface?
> An interface is a contract to ensure that a type has implemented the method defined in the reference interface. 
> Remember that in Golang the interfaces are implemented implicitly, so you do not declare or change the type, 
> simply you satisfied or not.

All the types we have seen so far are concrete types. Interfaces are **abstract type** instead: there is not an internal implementation only a definition of methods.

A common interface in Golang is the `io.Writer` let's see how it is implemented:
```golang
type Writer interface {
    Write(p []byte) (n int, err error)
}
```
as you notice the interface is composed by a type named `interface` and one or more methods that define the contract of this type. If another type has an implementation
for all the methods contained in an interface type, it implicitly adopts (or simply implements it). Let's see a complete example: below you can see a type named `MyType` that implements the
interface `io.Writer`.
```golang
import (
	"bytes"
	"fmt"
)

type MyType struct {
	buffer bytes.Buffer
}

// "Constructor" for our type
func NewMyType() MyType{
	m := MyType{
		buffer: bytes.Buffer{},
	}
	m.buffer.Grow(1000)
	return m
}

// Method to save into our Type
func (t *MyType) Write(p []byte) (n int, err error){
	return t.buffer.Write(p)
}
```
the method `Write` implement the interface. To simplify the code, we use behind the scene the `bytes.Buffer` type that implements itself a `io.Writer` interface.
Now, the **fun part**: if a type implements an interface, you can use that type as an argument of a func or a method or simply a variable of an object that has this interface as a type.

We have our `NewMyType` that implements `io.Writer` interface. We know that `fmt` has the `Fprintf` function that, as first parameter, has
a `io.Writer`. Therefore, you can use the object as:
```golang
// instance of new MyType
m := NewMyType()
m.Write([]byte{'a', 'b', 'c'})
m.ShowContent()

// use our type with Fprintf
fmt.Fprintf(&m, " - Hello from %s", "MyNewType")
m.ShowContent()
```
in the first three lines above we assign the `NewMyType` to the `m` variable, then write "abc" using our `Write` method and show the content. After
we call the `fmt.Fprintf` to write the string "- Hello from MyNewType" into the `Writer`. The func converts the string into a slice
of bytes passing it to the `Writer`, calling the `Write` method. By this way, our method is called.
**Remember** that we have defined a pointer as the receiver to the `Write` method, so we have to call using the address `&m`.

Go ahead with our implementation we can implement (or satisfy) another interface, the `Stringer`. It is defined as:
```
type Stringer interface {
    String() string
}
```
basically, to implement it, we need to add a method as:
```golang
func (t MyType) String() string {
	return t.buffer.String()
}
```
to see if it works, we call a func that try to convert its parameters to a string before print it and, to do that, uses the `String` method
of the type passed as argument. A func like that is `fmt.Println`. So, after the last `m.ShowContent()` add:
```
fmt.Println(m)
```
You can find the entire code of ***interfaces*** examples in the [go-language](https://github.com/mas2020-golang/go-language/tree/main/interfaces) repo.

### Satisfy an interface
It means that an interface is satisfied by a type: all the methods of the interface are implemented for that type.
Pay attention during the assignments:
- you can assign at an interface an object of a type that implements that interface:
```golang
var r io.Reader
r = os.OpenFile("test.txt", os.O_RDONLY, 0666)
```
it is OK because the `os.File` implements the `io.Reader` interface.
- the right side of an expression might be an interface also:
```golang
var r io.Reader
var rc io.ReadWriteCloser // interface that implements Reader,Writer and Closer io interfaces
r = rc // it is ok, rc implements the io.Reader
rc = r // it is not ok because r doesn't implement Writer and Closer interfaces
```
{{% alert title="Tip" color="success" %}}
Remember that if a type has a method that implements an interface and the receiver is a pointer, only the pointer
receiver will satisfy the interface.
{{% /alert %}}
Following the `MyType` type (more explanations in the next chapter) this method implements the `io.Reader` interface:
```golang
func (t *MyType) Read(p []byte) (n int, err error){
...
}
```
if you have an instance of `MyType` and try the following code you get an error:
```golang
m := NewMyType()
var _ io.Reader = m

ERROR:
Cannot use 'm' (type MyType) as type io.Reader Type does not implement 'io.Reader' as 'Read' method has a pointer receiver
```
instead, the following it's ok:
```golang
var _ io.Reader = &m
```

### Implement the io.Reader interface
Let's see an example how to implement the `io.Reader` interface into MyType.
```golang
// Method that implements io.Reader interface
func (t *MyType) Read(p []byte) (n int, err error){
	// Check if we are at the end
	if t.i >= int64(t.buffer.Len()) {
		return 0, io.EOF
	}
	n = copy(p, t.buffer.Bytes()[t.i:])
	t.i += int64(n)
	return
}
```
we have a buffer and an internal counter i. When i is > than the len of the buffer we are at the end and return a `io.EOF`.
Otherwise we copy from the buffer starting from i and stopping (potentially) at the end of the buffer (copy will take only min beetween
the remaining length in the buffer, from i to the end, and the size of p).
Then, i is incremented by the copied bytes.
Key points are:
- we return the number of copied bytes or ZERO in case we are at the `io.EOF`
- we overwrite the values of the byte slice for every interaction

An example of using the `Read` method could be:
```golang
buffer := bytes.Buffer{}
buf := make([]byte, 10)
for true {
  n, err := m.Read(buf)
  if err == io.EOF {
    break
  }
  buffer.Write(buf[:n])
}
```
Key points are:
- we read 10 bytes for each iteration
- we write the bytes read into the local buffer
- at the end of the Reader we break
We can then read the value of the buffer.

### Embedded interface
It is possible combine more interfaces into one: a type will implement this particular interface having all the methods as the corresponding interfaces.
For example:
```golang
type Family interface{
  Father
  Mother
}
```
Writing code that implements all the methods related to the `Father` and `Mother` interface, means to satisfy the `Family` interface as
well.

### What is the `interface{}` type?
This is the ***empty interface*** type. You can assign whatever to an empty interface, for example:
```golang
var empty interface{}
empty = 1
empty = ""
empty = true
empty = map[int]int{1: 100}
empty = MyType{}
```
