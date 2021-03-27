---
title: "Complex Types"
linkTitle: "Complex Types"
weight: 1
description: >
  This paragraph is to explain some complex types that you will use on a daily basis.
---

## What is included in this section?
The complex types shown here are:
- arrays
- slices
- maps
- struct

On the right menu you can find a rapid link to directly jump into a section you need a quick refresh.

## Arrays

Arrays are fixed collections of a declared type.

{{< alert title="Key points" color="success" >}}
- arrays are **fixed** and cannot be resized
- the array elements can be modified
- are pointed started from zero to the length minus one
- by default the elements of a new array are initialized to the ZERO values for that type 
- you can use literals to initialize an array
- it is possible to **compare only two arrays of the same type and the same length**
- two arrays are equals if all the elements have the same values
{{< /alert >}}

{{% alert title="Important" color="warning" %}}
Remember that every time you pass an array to a func, Go is copying the array into another one, it is time and memory consuming.
To avoid this you can use **pointers**.
{{% /alert %}}

This is an example that covers all the topics listed above (printValues is a function that prints a value):

```golang
var a [5]string // declaration of an array of 5 elements (init at ZERO values)
// cycle to get the values
printValues(a)
// set first and last value
a[0] = "first"
a[len(a) - 1] = "last"
printValues(a)

// initialization using literals
var b [5]string = [5]string{"a", "b", "c", "d", "e"}
//	b[5] = "test" // compile error => out of bounds
printValues(b)

// initialization using literals (without var) taking the length from the
// number of elements
c := [...]string{"a", "b", "c", "d", "e"}
printValues(c)

// it is possible to compare only two arrays of the same type and the same length
// two arrays are equals if the elements have the same values
fmt.Println("b == c", b == c)

// init and implicit declare specifying the size
i1 := [3]int{1,2,3}
i2 := [3]int{1,2,4}
i3 := [4]int{1,2,4,5}
fmt.Println("i1 == i2", i1 == i2)
fmt.Println("i3:", i3)
//	fmt.Println("i1 == i3", i1 == i3) // compile time error, same type but different length
```

## Slices
Slices are fixed collections of a declared type.

{{< alert title="Key points" color="success" >}}
- declaration is similar to the array without having to declare the length
- it has three components a pointer, a length and a capacity
- is a lightweight structure that gives access to the underlying object that is an array
- **pointer**: the first element reachable of the array (not necessarily the first)
- the **length** is the number of slice elements (***len*** function gives it)
- the **capacity** is the number of elements between the start of the slice, and the end of the array (***cap*** function gives it)
- multiple slices can refer to the same underlying array
- you cannot use == operator to test if two slices have the same values, you need to do it with your own func
{{< /alert >}}
Follow an awesome image taken from the book ***Go Programming Language*** that illustrates the len, cap and the underlying array:

![](/img/slices.png)

The operator **x[i:z]** returns a new slice that goes from i (int index) to z -1 of a sequence z. What is `x`?
- another slice
- an array
- a pointer to an array
if i is not present it is 0, if z is not present, it assumes len(x)

### Init a slice
Below there are some ways to create a slice:
```golang
var s []int // nil slice
var s1 = []int{} // empty slice with 0 elemes (not nil)
s2 := []int{1,2,3} // slice literal
s3 := make([]int, 10, 20) // slice of 10 len and 20 cap with 10 elems set to int ZERO value
s4 := new([20]int)[:10] // same as s3
fmt.Println(s, s1, s2, s3 ,s4)
```
Behind the scene, make creates and underlying unnamed array variable and return a slice of it. The array is accessible only throughout
the slice.

Below there are other examples to build slices using [:] syntax:
```golang
months := []string{1: "January", 3: "March", "April", "May", "June", "July"}
// an example of the slice operator that creates a new slice
firstMonths := months[0:3] // -> 0,1,2 indexes
printValues(firstMonths)
// other slice examples
allMonths := months[:]
// from beginning to end
allMonthsTwo := months[0:]
// or
allMonthsThree := months[0:9]
printValues(allMonths)
printValues(allMonthsTwo)
printValues(allMonthsThree)
```

Knowing the existence of the underlying array we can extend a slice inside the cap of the array, referring to the previous
example:

```golang
// to extend a slice always inside the capacity
fourMonths := firstMonths[:4]
printValues(fourMonths)
```
Slice is a ***reference type*** because it contains a pointer to an array, therefore passing a slice as an argument to a function permits to modify 
the underlying array. Remember that Go always copy the value but, in this case, it copies a pointer not an entire object.
To show this take a look at the following reverse example, as you can notice the argument passed to the function is the original slice
that after execution is reversed:

```golang
// int slice and reverse (Go passes a copy of a pointer of the slice, so the func can change the values)
i := []int{1,2,3,4,5,6}
reverse(i)
fmt.Println(i)

  // Reverse a slice
func reverse(a []int){
	for i,j := 0, len(a) - 1; i < j; i,j = i+1, j-1{
		a[i], a[j] = a[j], a [i]
	}
}
```
in the above example remember the multiple assignment separated by comma, *i,j := 0, len(a) - 1*, where i is 0 and j is 5.

The **ZERO value** of a slice type is nil:
```golang
// nil values
var s []int
fmt.Printf("len(s) is %v, s == nil: %v\n", len(s), s == nil) // len(s) is 0, s == nil: true
s = nil
fmt.Printf("len(s) is %v, s == nil: %v\n", len(s), s == nil) // len(s) is 0, s == nil: true
s = []int(nil)
fmt.Printf("len(s) is %v, s == nil: %v\n", len(s), s == nil) // len(s) is 0, s == nil: true
s = []int{}
fmt.Printf("len(s) is %v, s == nil: %v\n", len(s), s == nil) // len(s) is 0, s == nil: false
```

### Copy a slice
To copy a slice into another slice use the **copy** function:
```
func copy(dst, src []Type) int
```
the number of elements copied is the min len of dst or src, for example:
```golang
var s3 = make([]int, 2)
s2 := []int{1, 2, 3, 5, 6} // slice literal
n := copy(s3, s2)
fmt.Printf("number of elements copied is %d, value of dest is: %v\n", n, s3)
```
the dest must not be nil, else the copy will not be executed.

### Append new element
Use the built-in **append** function:
```golang
// Creation of a slice of 5 elements
i := make([]int, 5, 10)
fmt.Printf("type is %T, len is %v, cap is a %v\n",i,  len(i), cap(i)) // type is []int, len is 5, cap is a 10
fmt.Println(i) // => [0 0 0 0 0]
i = append(i, 100)
fmt.Println(i) // len == 6, cap == 10 => [0 0 0 0 0 100]
```
The second parameter of the append function accepts '...Type'. The ellipsis '...' means that it accepts any number of arguments.
{{< alert color="warning" title="ATTENTION" >}}
Every time a slice changes its ***cap*** means a new allocation and copy, therefore pay attention to the cap and len it's a good
way to be more efficient.
{{< /alert >}}

### Iterate a slice
An example of iteration:
```golang
for i, v := range a {
  fmt.Printf("i: %d, value: %s\n", i, v)
}
```
where *i* is the index and *v* is the value. If you want to omit i or v pass the '_' character.

### Delete an element
How to delete an element from a slice?
We take the position of the element to delete, move the last value into it and wipe out the last position from the slice:
```golang
func deleteElement(pos int){
	fmt.Println("\n---Delete element")
	s := []int{10,20,30,50,100,90}
	fmt.Println("before deletion s slice is", s)

	// Remove the element at index i from a.
	s[pos] = s[len(s)-1] // last element overrides the pos passed
	s = s[:len(s)-1]   // remove last element
	fmt.Printf("after deletion s slice is %v (len: %d, cap: %d)\n", s, len(s), cap(s))
}
```
On the github repo [go-language](https://github.com/mas2020-golang/go-language/blob/main/slices/slices.go) you can find an example that shows the
slower way to delete an element maintaining the order.

## Maps
It is an **unordered collection** of key/value pairs. Syntax is map[K]V where K is the type of the K, and V is the value. 
A value could be a simple type or a composite type, like other map or an array.

{{< alert title="Keypoints" color="success" >}}
- ***ZERO value*** for a Map is `nil` (Map address is a reference to an HASH table)
- lookup, delete, range, len are ***safe*** on a `nil` Map. **Add element** on a `nil` Map causes the panic instead.
- in Go a map is a reference to an HASH table
- all the keys of a Map are of the same type; all the values are of the same type. Keys and Values do not necessarily be of the same type
- cannot take an address from a map element (e.g. &test["xyz"])
- a map lookup using a key that doesn't exist will not give an error, returns the ZERO value for the value **_type_**
- map ***cannot be compared*** each other (as slices), the only legal comparison is with `nil`
{{< /alert >}}

Elements inside a map are not ordered, suppose you have a map with keys string and numbers as values, to order you can use your own function:
```golang
func sortMap(ages map[string]int)  {
	var names = make([]string, 0, len(ages)) // slice for the names
	// append all the names to the slice in a unordered way
	// we omit the second value in the for loop using only the key of ages and not the value
	for key := range ages{
		names = append(names, key)
	}
	// sort slice of names
	sort.Strings(names)
	// read ordered names from slices and get the values of ages from the key name
	for _, name := range names{
		fmt.Printf("%s\t%d\n", name, ages[name])
	}
}
```

### Init a map
Some examples on how to create a map:
```golang
var testNil map[string]int // ZERO value is nil

// creation with make
names := make(map[string]int)
fmt.Printf("names == nil: %v, content is: %v\n", names == nil, names) // => false

// another way to create an empty map
ages2 := map[string]int{}
fmt.Println("ages2 == nil:", ages2 == nil) // => false

// maps from literals
ages := map[string]int{
  "ag": 43,
  "sm": 29,
  "mg": 0,
}
```

to test if a **key esists** you can type:
```golang
// check if a key exists:
age, ok := names["test"]
fmt.Printf("names test exists: %v and its value is %v", ok, age)
```
this code will save in `age` the value of the key 'test' if a key exists, else it will store the **ZERO** value of the element type. 
`ok` will be true in case a key exists, otherwise false.
Simply testing a key, without using the previous code, in case the key doesn't exist, returns the **ZERO** value of the element type, e.g. in case of int 0.
Sometimes could be useful, sometimes no, in that case use the check form instead.

### Add new element
To add elements you can use a not existing key as shown below:
```golang
func addElements(m map[string]int) {
	m["test"] = 100
	m["zed"] = 90
	m["jhon"] = 53
	fmt.Printf("values of m are %v\n", m)
}
```

### Remove an element
To remove an element you can type:
```golang
func deleteElements(m map[string]int) {
	delete(m, "ag")
	fmt.Printf("values of m are %v\n", m)
}
```
in case we delete an element that doesn't exist, no error will be thrown.

## Struct

It is simply a collection of fields/properties.
In an OOP perspective we can think of a struct as a light class that supports composition but not inheritance.

{{< alert title="Keypoints" color="success" >}}
- ***ZERO value*** for a Map is the composition of the ***ZERO value*** for its of each fields.
- an empty struct is considered a struct with *ZERO* fields and is expressed as `struct{}`. To assign it, for example to the following map: `map[string]struct{}` you have to write (supposed map is m): `m["test"] = struct{}{}`.
- outside the package where the Struct is located, can be accessed only the capitalized fields
- an instance of a struct is a variable. The *fields of the struct are variables too* and can be accessed, assign to them a value, referenced as pointers.
- Fields order is significant to the type identity
- A struct cannot contain a field of the same type of the struct itself, instead it can have a field that is a pointer to the struct type. It could be useful to create a recursive struct.
- when you pass a complex struct to a func, is more efficient if you pass it as a pointer
{{< /alert >}}

### Init a struct

Suppose to have the struct:

```go
// Simple struct
type Vehicle struct {
  brand      string
  model      string
  horsepower int
  year       int
}
```

it can be initialized as:
```go
// 1- Using the literal syntax
Vehicle{brand: "Fiat",
  model:      "Punto",
  horsepower: 90,
  year:       2002}
 
// 2- Using the the dot notation
var v Vehicle
v.brand = "Tesla"
v.model = "Tesla1"
v.horsepower = 120
v.year = 2020
fmt.Println(v)
 
// 3- using literal in a particular order omitting the name of the field
fmt.Println(Vehicle{"Fiat", "Giulia", 140, 2018})
 
// 4- using the new keyword that creates a pointer to the type initialing the fields
// with the zero value of the specific type
fmt.Println(new(Vehicle))
// It is similar to
fmt.Printf("%#v\n",&Vehicle{})
```
For the moment don't take care at the '&' character in the last row.
You can also declare a struct without give it a particular name and instance it, take a look:
```golang
var color = struct {
	rgb int
	name string
}{
	100,
	"red",
}

fmt.Println("color is", color.name)
```
### Compare structs
Remember that if all the fields in a struct are comparable, the structs are comparable.
For example:

```golang
func compare()  {
  v1 := Vehicle{
    brand:      "Tesla",
    model:      "V1",
    horsepower: 125,
    year:       2018,
  }

  v2 := Vehicle{
    brand:      "Tesla",
    model:      "V2",
    horsepower: 135,
    year:       2019,
  }

  fmt.Printf("v1 == v2 => %t\n", v1 == v2) // => false
}
```

### Add a method to a structure

To add a method to a structure you have to use a ***receiver***. A receiver is a special way in which Go passes at the method the object related to the strut itself.
To create a method you have this special syntax:

```go
func (object Type) Method()  {
	...
}
```
the object is the first parameter, follow by the `func name + args + return` values.

for example, if you want to add a method **`Go`** to the vehicle struct, type:

```go
func (v Vehicle) Go() {
	fmt.Println("Started!")
}

// and use it
var v Vehicle
v.brand = "Tesla"
v.model = "Tesla1"
v.horsepower = 120
v.year = 2020
fmt.Println(v)
v.Go()
```
Under the hood, Go will take the v object and passes its instance to the Go() function, so inside the method you can use v object.
**Remember** that ***Go will copy the value*** of the v object, therefore if you change it (for instance changing a value of a field) inside the method, the original object is still unchanged. If you need to do it (or your struct is big and you want to improve the memory usage) you can use a pointer instead:
```go
func (v *Vehicle) Go() {
  fmt.Println("Started!")
  // change velocity for example...
}
```
We can attach a method to any type (except a pointer or an interface) not only on structs. 
When invoking a method of an object checks the receiver type parameter, it could be a pointer. Golang converts for us from normal type
to a pointer in such cases, for example having this struct:
```golang
// Simple struct
type VehicleNew struct {
	brand      string
	model      string
	horsepower int
	year       int
}

func (v *VehicleNew) Go() {
	fmt.Println("Started!")
}

func (v VehicleNew) GoSlow() {
	fmt.Println("Started!")
}
```
above we have two methods, one accepts a pointer as receiver parameter, the second accepts the T VehicleNew. Some cases to avoid mistakes:
- argument is a T and receiver is a *T:
```
vn := VehicleNew{
  "Tesla", "TeslaXY", 125, 2020,
}
vn.Go() // Golang automatic converts to a pointer
(&vn).Go() // explicit call using a pointer as a receiver argument
```
- argument is a *T and receiver is a *T:
```
vn2 := &VehicleNew{
  "Tesla", "TeslaXY", 125, 2020,
}
vn2.Go() // receiver is a pointer, argument vn2 is a pointer
```
- argument is a *T and receiver is a T:
```
vn2 := &VehicleNew{
  "Tesla", "TeslaXY", 125, 2020,
}
vn2.GoSlow() // argument is a pointer, Golang dereference the pointer automatically
(*vn2).GoSlow() // explicit call dereferencing the pointer
```
{{% alert color="warning" title="Important" %}}
- usually when you create a type, if you use a pointer as a parameter receiver, all the methods have to be written in this way
- usually the receiver name is the first letter of the type name (lowercase)
{{% /alert %}}

### Type composition

What is this technique? You can use to compose two or more structures together "similar" as an OOP approach for inheritance.

For example, suppose we have this two type, Basic and Specific:

```go
type BaseType struct {
  Name string
  Surname string
  Age int
}
 
/*
Specific type contains the BaseType struct
*/
type SpecificType struct {
  BaseType
  Type string
  Category string
}

```

As you can see the first property of the SpecificType struct is another Structure, and we not specify the property name.
By this way you can init the SpecificType using the dot notation as:

```go
var st SpecificType
st.Name = "My name" // BasicType
st.Age = 12 // BasicType
st.Category = "Cat of spec type"
st.Type = "Type of spec type"
st.Surname = "Surname" // BasicType
```

instead, using the literal form, we must respect the order of the fields:

```go
st1 := SpecificType{
  BaseType{Name:"My name", Surname: "Surname from Base Type", Age: 12},
  "Type of spec type",
  "Cat of spec type",
}
```

or, we have to specify the fields name:

```go
st1 := SpecificType{
  Type: "Type of spec type",
  Category: "Cat of spec type",
  BaseType: &BaseType{Name:"My name", Surname: "Surname from Base Type", Age: 12},
}
```

{{% alert title="Note" color="info" %}}
If you define a method on a Struct and there is a composition, the method itself is available on the struct that 
includes another struct. **Remember** that this is valid only for the implicit composition as seen above. Instead, if 
you assign the structure type to a field, for instance:

```go
type Job struct {
  Command string
  log *log.Logger
}
```

you cannot invoke the Logger method inside Job object directly, you can only call using:

```go
var j Job
j.log.info()
```

{{% /alert %}}

{{% alert title="Tip" color="success" %}}
It is better to have a pointer in a composition for performance reasons. Go “always” passes the argument by copying values, using a pointer only the reference is passed. For instance:

```go
type SpecificType struct {
*BaseType // using a pointer is better for performance reason
Type string
Category string
}
```

By this way you cannot use the dot notation to init the object because the pointer would be nil, you can only use this way:

```go
st1 := SpecificType{
&BaseType{Name:"My name", Surname: "Surname from Base Type", Age: 12},
"Type of spec type",
"Cat of spec type",
}
```
For major information related to the use of Pointers take a look [here](../pointers/).
{{% /alert %}}



## Go Template
With Golang, we can use template system: it is a way to substitute at a string some value injected by a template `action`.
An `action` is a particular expression included in double brackets: `{{ ... }}`.
With an expression we can:
- print values, 
- select struct fields, 
- call functions and methods,
- use a control flow such as if-else statements and range

`'.'`: the current value inside an expression is given by the 'dot' character.

`'|'`: this character is similar to le Unix PIPE, it permits to pass a value to another command as we are going to see.

{{< alert title="Building a template" color="success" >}}
- to use a template first you need to **`parse`** it: only one time operation
- **`execution`** of a template is the next phase
{{< /alert >}}

### Simple template
An example of a simple template is:
```golang
var t = `{{- range . -}}
-------------------------------------
Brand: {{ .Brand }}
OS: {{ .OS }}
Age: {{ .Age }}
-------------------------------------
{{ end -}}
`
```
the above Golang backtip string shows:
- using of the `range` to cycle the elements. Inside a cycle we show values for the property Brand, OS, Age.
- remember that the 'dot' is the pointer of an execution: range will cycle some cyclable element that we pass to the execution of the template
  
A simple struct that maps the template content coud be:
```golang
type Computer struct {
	Brand     string
	OS        string
	Age       time.Time
	Color     string
	Memory    int
	Processor string
}
```
Suppose we have this method (code is omitted for brevity) that loads some computers:
```golang
computers := fillComputers()
```
then write the code to Parse and Execute the template. Execute will put the output into the stdout stream.
```golang
c, err := template.New("computers").Parse(t)
if err != nil {
  log.Fatal(err)
}
if err := c.Execute(os.Stdout, computers); err != nil {
  log.Fatal(err)
}
```
the output will be:
```
╰─$ go run main.go 
-------------------------------------
Brand: Apple
OS: Catalina
Age: 2016-02-01 00:00:00 +0000 UTC
-------------------------------------
-------------------------------------
Brand: Dell
OS: Windows 10
Age: 2018-02-01 00:00:00 +0000 UTC
-------------------------------------
```
That's it for the basics.
Major information could be find in the official documentation related to the text/template pkg [here](https://golang.org/pkg/text/template/).
A blog post is coming soon!!