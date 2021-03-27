---
title: "Using Pointers"
linkTitle: "Using Pointers"
weight: 3
description: >
  What they are and how to use them
---

### What is a pointer?

A pointer is an address that references some area in memory. A pointer variable is a variable that stores that address.
So, using a pointer, we can know and use the address to reference a value.

In Go to know the address of a variable we use the **&** character and to get the underlying value of a pointer we use **\*** as a prefix for the variable name.
For example:

```go
// this is an example to change the the value of a string using a pointer
var ori string = "test"
// address is a pointer and the value is the address of ori
var address *string = &ori
fmt.Printf("ori value is %s, ori address is %v, address value is %v\n", ori, address, (*address))
```

in the example above `address` is the variable of type *string that means a pointer of type string. The value of a pointer is an address it self (in this example the address of the variable ori). The value of the `address` variable is the reference value of the address that is the value of the `ori`.
To change the value of the underlying value of `address` we need to write:

```go
(*address) = "test-new"
```

In the code above we are changing the underlying value to the corresponding address. Of course is possible to assign a pointer in an implicit declaration, for instance:

```go
p := &ori
fmt.Printf("p value is %v, p address is %v, p reference value is %v\n", p, &p, (*p))
```

we get something like:

```shell
p value is 0xc000010200, p address is 0xc00000e030, p reference value is test-new
```

the first value is the result of the assignment, the address of `ori`. The second value is the address of `p` itself and, the third value, is the underlying value which the pointer is linked to (now you imagine the answer: the value of the `ori` variable).

***Which is the ZERO value of a pointer?***
It is `nil`, so, if a pointer doesn't point anything it could be `nil`, else the it will evaluate the address of the variable which is pointing to.
Two pointers are equal when they are pointing to the same address, or if they both are nil.
{{% alert title="Tip" %}}
The return of a function can be a pointer to a variable defined in that function. It means that the scope of the variable lives outside the scope of the function where is defined. It is an exception to the rule seen [here.](/docs/basic-of-the-language/#variables-scope)
{{% /alert %}}

### Using a pointer as a function parameter

A gold principle to remember related to Golang is:
{{% alert title="Important" color="warning" %}}
Everything is passed into a function is a copy of the argument itself. For instance:
{{% /alert %}}
```go
func main() {
	t := "test"
	callMe(t)
	fmt.Println(t) // it will report "test"
}

func callMe(p string) {
	p = "test2"
}
```
`t` variable is copied into another variable and this new variable is passed as argument. The corresponding update of `p` string will not be reflected to the `t` string that will remain unchanged.
Pointers come in our help, instead of the previous code we can type:
```go
func main() {
	t := "test"
	callMe(&t)
	fmt.Println(t) // it will report "test2"
}

func callMe(p *string) {
	(*p) = "test2"
}
```
In the example above we have passed as argument of the callMe func the pointer of `t` and we have changed the underlying value. The effect is that we have updated the `t` original value.
The principle that Go passes an argument copying it into another variable is still valid but, when Go copies the &t variable it copies the address value. The result of this is that changing the underlying value brings to change the `t` variable too.
To show this try to change the previous code in:
```go
func main() {
	t := "test"
	fmt.Println("address of t is", &t)
	callMe(&t)
	fmt.Println(t) // it will report "test"
}

func callMe(p *string) {
 	fmt.Println("values of p is", p)
	fmt.Println("address of p is", &p)
	(*p) = "test2"
}
```
you get this result:
```
address of t is 0xc000010200
values of p is 0xc000010200
address of p is 0xc00000e030
```
The conclusion is:
- `&t` is an address to the `t` variable
- `p` is a pointer and its value is the address of `t`
- `&p` has its own address showing that p and t are two different variables pointing to the same address

***Be patiernt and do not worry, with a bit of practice you'll become familiar with this concept...***

### Create variable as a pointer

It is possible, but it is rarely used, create a pointer with the `new` predeclared function. This function accepts as argument the type of pointer to create. The return is a pointer to a unnamed variable with the ***ZERO*** value assigned. For example:
```go
func main() {
	pointer := new(int)
	fmt.Println("address is", pointer)
	fmt.Println("value is ZERO value for that type:", *pointer)

	pointer2 := new(struct{})
	fmt.Println("address is", pointer2)
	fmt.Println("value is ZERO value for that type:", *pointer2)
}
```
the output is something as:
```
address is 0xc00002c008
value is ZERO value for that type: 0
address is &{}
value is ZERO value for that type: {}
```
to be sure that the variable is initialized to the ***ZERO*** value add the following code to the previous function. You will see a **panic error** trying to assign a new element in a map that is `nil`. Remember that for the reference types seen [here](/docs/basic-of-the-language/#long-variable-declaration) the ***ZERO*** value is `nil`:
```go
pointer3 := new(map[string]string)
fmt.Println("address is", pointer3)
fmt.Println("value is ZERO value for that type:", *pointer3)
(*pointer3)["test"] = "test"
```