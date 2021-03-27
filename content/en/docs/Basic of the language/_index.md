---
title: "Basic of the language"
linkTitle: "Basic of the language"
weight: 2
description: >
  What do you need to know to try with Golang?
---

This chapter explains the basic knowledge to start with **Golang**.

## Structure of a Go application

In Go any program is composed by one, or many files that end with '*.go'. Every file has a `package` declaration on top it.
The elements are:
- **package** declaration
- import declaration
- declaration of types, variables, constant and functions (order doesn't matter)

An example of a Go application:
```go
package main

import "fmt"

type BaseType struct {
	Name    string
	Surname string
	Age     int
}

func (b *BaseType) Greetings() {
	fmt.Println("Hello from BaseType!")
	(*b).Name = "test"
}

/*
Specific type contains the BaseType struct
*/
type SpecificType struct {
	*BaseType // using a pointer is better for performance reason
	Type      string
}

func createSpecificType() {
	// new SpecificType using literal
	st1 := SpecificType{
		Type:     "Type of spec type",
	}
	st1.Greetings()
}
```
In this example we have in order: package declaration, import declaration, types and functions.

***A Go application might have one or more packages***. A single package could be one file or more file that at the beginning start
all with the same package declaration. See how to use your own package later on in the documentation.

### Packages, what they are and how to use

{{< alert title="Keypoints" color="success" >}}
- a package is declared using the corresponding keyword
- the source code of a package resides in one o more .go files located in the same folder
- usually the name of the package is the name of the folder which is located
- all the files of the same package can use functions and variables without qualifying them
- outside the package to use variables or functions we need to point to the package
- only the variables or functions with the first capital letter are exposed outside the package they belong
{{< /alert >}}

Ok, try to create a new simple project with this structure:
```
├── main.go
└── pkgtest
    └── test.go
```
The content of the file `main.go` is:
```golang
package main

import (
	"fmt"
	"pkgtest"
)

func main(){
  fmt.Println("Start...")
  fmt.Println(pkgtest.Version())
}
```
and the content of the file `test.go` is:
```golang
package pkgtest

func Version() string {
	return "1.0.0"
}
```

we have a project executable due to the presence of the main function, and we have a nested package to use named *pkgtest*.
We know that to export something the first letter has to be capitalized, so we try:
```
go run main.go
```
but, unfortunately it doesn't work...let's take a look:
```
go run main.go 
main.go:5:2: cannot find package "pkgtest" in any of:
        /usr/local/Cellar/go/1.14.2_1/libexec/src/pkgtest (from $GOROOT)
        /Users/andrea/go/src/pkgtest (from $GOPATH)
```
Ops...Go is trying to fetch the source of pkgtest in two locations: GOROOT (where go is installed) and GOPATH. In both cases the searched folder is **`src/pkgtest`**.

The Go versions >1.11 solve this issue introducing the **module** feature, but we come back later on this, so the only way to get work the project is to
move it from the current location (wherever it is) to the GOPATH/src folder.
The folder to move is only pkgtest, the rest of the project can be wherever you want. Therefore, copy the pkgtest into the GOPATH/src directory.
In this example we move the code here:

```shell
/Users/andrea/go/src/pkgtest
```

Let's try `go run main.go`  and it will work.

#### Packages initialization

If you have more than one file that refer to the same package, they are initialized in the order where are fetched from the compiler (alphabetical order).
If you need to group some init steps you can use the ***init()*** func that is automatically invoked from Go. The dependencies are initialized first to be sure the corresponding package is ready before using it.

**Main package** is the last to be initialized.

## Variables

In Go a variable can be declared in various way. It can be declared and then assigned or both at the same time.
Take a look at some examples.

### Long variable declaration

Using the *var* keyword with this syntax:

**var** name ***type*** = ***value*** ==> type and value can be omitted but not both, it means that:
- if you omit type the type of the variable is set by the value
- if you omit the value the variable is initialized with the ***ZERO*** value that is:
  - **0** for numbers, **false** for booleans, **""** for string, and **nil** for interfaces and reference types (slice, pointer, map, channel, function). Therefore, in Go there is no such thing as uninitialized variables.

Here some basic usage:
```go
// explicit assignment
var (
  name1     string
  age1      int
)
name1, age1 = "test", 1
fmt.Println("variables are:", name1, age1)

// declaration of multiple variables
var a,b,c int
println("variables a,b,c, are:", a,b,c)

// Variable can also be declared one by one
var one string
var second int
var third bool = false
one, second = "hello", 100
fmt.Println("variables are:", one, second, third)
```
### Short variable declaration

In this example name and location are implicit infered as string by Go:

```go
name, location := "Prince Oberyn", "Dorne" // implicit string
test := 100 // implicit int
```

{{% alert title="Important" color="warning" %}}
Implicit declaration cannot be use at the package level. It means that you can use this syntax only into a function.
{{% /alert %}}


### Variables scope

Inside a Go program the variables or constants are visible at:

- `package level`: each variable or constant defined into a package is visible inside the files belong to that package and from external package
  if the name is with a capital letter.   
- `function level`: variables are visible inside the scope of the function. Although there are some exceptions,
  for now let's focus on this principle.
- `block level`: for blocks we refer to statements enclosed by brackets '{}'. For instance: for loop, if, switch etc... 
  The variable declared into a block lives into it.

### Assignment

A simple assignment in Go comes with the form `var = value`. You can also have:

```go
// declaration and assignment
i := 1

// increment by one
i++
// decrement by one
i--
// short form to increment by x
i += 5
// it is the same as
i = i + 5
```

Another particular form of assignment is the ***Tuple assignment***. This assignment is useful to swap the values of two variables:
```
a, b = b, a
```

a common use of this assignment is when we invoke a function that returns back more than one value:

```
err, val := someFunc(x)
```

if we do not want to evaluate some variables we can use the *blank identifier* instead:
```
_, val := someFunc(x) // ignore error
```

{{< alert title="Tip" color="success" >}}
To assign a value to a variable, the value must be of the same type, or better: an assignment is valid only if a
specific value is ***assignable*** to a specific type.
{{< /alert >}}


## Types

This is the list of the supported types in Golang.

**boolean** and **string**:

- `bool`
- `string`

**numeric** types:

- `uint`        either 32 or 64 bits
- `int`         same size as uint
- `uintptr`     an unsigned integer large enough to store the uninterpreted bits of
                a pointer value
- `uint8`       the set of all unsigned  8-bit integers (0 to 255)
- `uint16`      the set of all unsigned 16-bit integers (0 to 65535)
- `uint32`      the set of all unsigned 32-bit integers (0 to 4294967295)
- `uint64`      the set of all unsigned 64-bit integers (0 to 18446744073709551615)
- `int8`        the set of all signed  8-bit integers (-128 to 127)
- `int16`       the set of all signed 16-bit integers (-32768 to 32767)
- `int32`       the set of all signed 32-bit integers (-2147483648 to 2147483647)
- `int64`       the set of all signed 64-bit integers
                (-9223372036854775808 to 9223372036854775807)
- `float32`     the set of all IEEE-754 32-bit floating-point numbers
- `float64`     the set of all IEEE-754 64-bit floating-point numbers
- `complex64`   the set of all complex numbers with float32 real and imaginary parts
- `complex128`  the set of all complex numbers with float64 real and imaginary parts
- `byte`        alias for uint8
- `rune`        alias for int32 (represents Unicode code point)

### Assertion and conversion of a type

> ***How can you convert a type into another?***

To **_convert_** types you can use the type as you invoked a function, for example:
```golang
var i int = 100
var f float64 = float64(i) // convert int => float64
var u uint = uint(f)       // convert float64 to an unsigned int
```

to **_assert_** a type:
```golang
z, ok := y.(map[string]interface{})
if ok {
  z["updated_at"] = time.Now()
  fmt.Println(z)
}else{
  fmt.Println("y is not of the type map[string]interface{}")
}
```
ok is a boolean, and it is true if the type corresponds to the type passed. If _true_ z will contain the value of the type, 
else the ***ZERO*** value for that type.

You can also **_assert_** via the **`switch`** operator:
```golang
// check a type with switch..case
switch str := y.(type) {
case int:
  fmt.Println("int type passed as argument:", str)
case map[string]interface{}:
  fmt.Println("y is map[string]interface{}:", str)
}
```

An awesome example taken from _golangbootcamp.com_ is:
```go
if err != nil {
  if msqlerr, ok := err.(*mysql.MySQLError); ok && msqlerr.Number == 1062 {
    log.Println("We got a MySQL duplicate :(")
  } else {
    return err
  }
}
```
**mysqlerr** variable can get the value in case of type mysql.MySQLError. In such case ok will be `true` and, if `true`, the code will evaluate
its value equals to 1062 printing the corresponding message "We got a MySQL duplicate :(". In this example there is a check, assignment and if into the same row.

### Create your own type

It means to create a specific type that has the same type of the underlying one. This is useful to better represent a variable when
the Go type could be not so specific. For example `int` type can be used for meters, miles, age etc...

The way to define a specific type is: `type <name> <underlying-type>`

Let's see an example:

```go
package main

import (
  "fmt"
)

type Meters float64
type Miles float64

func main() {
	var a Meters = 1000
	var b Miles = Miles(a) * Miles(0.621371)
	fmt.Println("a, b are:", a, b)
}
```

{{< alert title="Tip" color="success" >}}
A type conversion is possible if the two types have the same underlying value or are both pointers to the same object. 
In case you convert between different types Go could make a truncation of the value, or a copy of them (string => byte slice). For instance:
```go
var b Miles = Miles(a) * Miles(0.621371)
var c int = int(b) // value will be 621
```
Remember that in any case will be an error at runtime.
{{< /alert >}}
Two different variables can be compared if belong to the same type or are compared with an underlying type. A comparison 
between two different type will fail at compilation time. For instance:
```go
fmt.Println("a == 0", a == 0)
fmt.Println("a > 0", a > 0)
fmt.Println("a == b (Meter == Miles)", a == b) // will not build
```

## Integers and Float [TODO]

## Complex numbers [TODO]

## Booleans [TODO]

## String

What are the strings in Golang? Are ***immutable*** sequence of bytes. These are interpreted as UTF-8 encoded sequences
of Unicode.
*You can consider the strings as a slice of bytes*, where len is the number of bytes that compose a string and the string[i] is the
single byte.

{{< alert color="warning" title="Warning" >}}The i-th bytes of a string is not necessarily a character due to the UTF-8
encoded interpretation that requires two or more bytes.{{< /alert >}}

The slice or *substring* of a string consists of a new string of this length. Remember that the end of a slice is always 
length - 1. If **start** and **end** are omitted means the entire string. For example:
```golang
s := "Hello Golang Aware!"
fmt.Println(s[:]) // -> Hello Golang Aware!
fmt.Println(s[:5]) // -> Hello
fmt.Println(s[2:5]) // -> llo
```
{{< alert title="Keypoints" color="success" >}}
- string are ***immutable**, this is wrong: `s[0] = 'a'`
- string ***can be compared*** with == and <>; the comparison is done byte by byte
- ***ZERO value***  is an empty string
- it is possible that more strings share the same underlying array (as seen for the slices)
- to create a string use the double quotes ""
- building up strings incrementally can be not efficient, use bytes.Buffer in such cases
{{< /alert >}}

As a special sequence of characters you can use escape chars also:

- \a  “alert” or bell
- \b backspace
- \f form feed
- \n newline
- \r carriage return
- \t tab
- \v vertical tab
- \' single quote (only in the rune literal '\'')
You can include arbitrary bytes (only 1 byte) using also:
- hexadecimal escape char \xYY
- octal escape char \ooo (3 octal digits)

What are the ***raw string literal***? They are string wrapped by \`...\`: content is taken literally and no escape
are interpreted, obviously it can be multiline.
Are Usually used for:
- write regular expressions
- HTML templates
- JSON literals
- other long texts

### Unicode (UTF-32)
Until **_Unicode_** there was **_ASCII_** as a standard coding system. ASCII uses 7 bits to represent 128 chars. Then, with a variety of the languages
to represent, was born Unicode which collects all the characters in all the world’s writing systems.
{{< alert color="success" >}}
- each character is assigned to standard number called a *Unicode code point* (**rune** in Go)
- actually Unicode 8 defines over 120.000 code points (characters)
- the data type for the *rune* is `int32` (UTF-32 representation). Each unicode point has the size of 32 bits
- **IMPORTANT**: most characters uses still number fewer than 65.536 which fit in 16 bits
{{< /alert >}}

### Unicode (UTF-8)

It's an encoding standard invented by the author of Go Ken Thompson and Robe Pike and consist into a schema that count
from 1 byte to 4 bytes.
The purpose is to avoid representing each character with an int32 in such cases where fewer bytes could be fine.
Some bits of the first byte indicates how many bytes will be used to encode the single character. Therefore, you need
to take a look at the left foremost byte with these rules:

- if begins with 0 is 1 byte long (codebase <= 127 because you have only 7 bits to represent the code)
- if begins with 110, the second byte begins with 10 is 2 bytes long (codebase > 127 and <= 2047)
- if begins with 1110, the second and third byte begin with 10 is 3 bytes long (codebase > 2047 and <= 65535)
- if begins with 1110, the second, third, fourth byte begin with 10 is 4 bytes long (codebase > 65535 <= 0x10ffff)

This is not so simple to understand at a first glance: the wrong thing you could do is to represent the number
above as binary ones, for example you know that a character has a code point of 19990 that is `01001110 00010110`.
You could think it is two bytes long in UTF-8, it's wrong, it is three bytes long because some bits positions need to establish 
the length itself. If you ask Go to give you the hexadecimal representation of the character behind 19990 you get:
```
s := "世"
fmt.Printf("% x\n", s) // -> e4 b8 96
```
there are three hex digits, so 3 bytes. If you convert each byte in binary representation you get:
```
11100100 10111000 10010110
```
you can see that we match with the third case. If you separate the bits of the representation you get:
```
`1110` 0100 `10` 111000 `10` 010110
```
if you cut you get: `0100 111000 010110`
and if you join together into a better binary representation you obtain `01001110 00010110` that corresponds to the integer
19990, the codebase of the UTF mapping table.

The **_conclusion_** is that:
- the code point of the above character is 19990
- the hex code corresponding to the code point is 4e16
- the rune has the uint32 value of 19990
- the hex digit of the character is three bytes long and corresponds the way the code point is stored with UTF-8 encoding

Not all characters are simple to write. Unicode escaped in Go allow us to write them: `\uhhhh` for 16-bits and `\Uhhhhhh`
for 32-bits (h are in a hexadecimal format). For example:

```
"\xe4\xb8\x96\xe7\x95\x8c"
"\u4e16\u754c"
"\U00004e16\U0000754c"
```
represents the same two characters. Or in rune literals:
```
'\u4e16'  '\U00004e16'
```
A rune whose value is less than 256 may be written with a single hexadecimal escape, such as '\x41'  for 'A', but for higher values, a \u or \U escape must be used.

```golang
fmt.Printf("%c\t%d\n", '\uFFFD', '\uFFFD') // -> �, unicode escape into a rune, code point 65533
fmt.Printf("%c\t%d\n", '\u4e16', '\u4e16') // -> 世, unicode escape into a rune, code point 19990
fmt.Printf("%s\tlen: %d bytes\n", "\xe4\xb8\x96", len("\xe4\xb8\x96")) // -> 世 bytes: 3, hexadecimal escape, each \xx represents 1 byte
fmt.Printf("%s\tlen: %d bytes\n", "\u4e16", len("\u4e16") )// -> 世 bytes: 3, unicode escape into a string
```

Take a look at this example to read a rune:
```golang
func decodeRunes()  {
	s := "◊Hello, ◊Ç∞"
	fmt.Println("bytes length:",len(s))                    // "18"
	fmt.Println("rune in string:",utf8.RuneCountInString(s)) // "11"

	// Decoding rune: scan of a string reading the character based on the position due to the size
	// of the previous char
	for i := 0; i < len(s); {
		r, size := utf8.DecodeRuneInString(s[i:])
		fmt.Printf("%d\t%c\n", i, r)
		i += size
	}
}
```
DecodeRuneInString method will give the char and the size in bytes so that we can move to the next basing on the size of the previous one.
Fortunately, exists the range that decodes UTF-8 implicitly:
```golang
// decodes implicitly
for i, r := range s{
  fmt.Printf("%d\t%q\t%d\n", i, r, r)
}
```

### String conversions
This paragraph shows some ways of conversions:
```golang
s := "This is my string世"
// convert into a slice of bytes (opposite to string cab be changed)
b := []byte(s)
fmt.Printf("bytes len %d\t%v\n", len(b), b) // --> 20, the last char occupies 3 bytes
fmt.Printf("hex digits % x\n", s)           // --> 20, the last char occupies 3 bytes
// convert from bytes to string
s2 := string(b)
fmt.Printf("original string:\t%s\nconverted string:\t%s\n", s, s2)

// Convert from int to string
n := 123
fmt.Printf("str with Sprintf() is %s\n", fmt.Sprintf("%d", n))        // -> 123, string rep of an int
fmt.Printf("str binary with Sprintf() is %s\n", fmt.Sprintf("%b", n)) // -> 1111011, string rep of an int converted to binary
fmt.Printf("str with string() is %s\n", string(n))                    // -> {, because convert from int to UTF-8 representation
fmt.Printf("str with strconv.Itoa() is %s\n", strconv.Itoa(n))        // // -> "123"

// Convert from string to int
n, _ = strconv.Atoi("123")              // -> conversion using strconv
m, _ := strconv.ParseInt("123", 10, 16) // -> conversion using ParseInt, return i64 into n
fmt.Println("n,m", n, m)
```

## Constants [TODO]