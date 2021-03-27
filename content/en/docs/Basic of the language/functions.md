---
title: "Using Functions"
linkTitle: "Using Functions"
weight: 2
description: >
  What they are and how to use them
---

## Interesting topics

Before starting to explore the functions it is worth to mention some interesting concepts subject to misunderstanding.

### Parameters vs Arguments
- **Parameters** are variables declared into a Function declaration. They are the variables that we can use into the Function itself.
- **Arguments** are the values that are passed to the Function when invoking it.

### Expressions vs Statements
In IT the two concepts are often confused.
- **expression** is whatever you can assign or print. We can see as something that returns a value.
```golang
5 * 5
len(y)
max(x)
print(5)
```
- **statement** is an istruction executed inside a process:
```golang
if .. else
for .. := range xx
while {}
```

## A simple Function
A function declaration has a **name**, a **list of parameters**, an **optional list of results**, and a **body**:
```golang
func deleteElements(m map[string]int) {
	delete(m, "ag")
	fmt.Printf("values of m are %v\n", m)
}
```
In the above example:
- `deleteElements` is the name
- `m` is the parameter of type `map[string]int`
- we have nothing in return
- we have a body with two statements

the structure is:
```
func name(parameters) ret-values {
  body
}
```
- `parameters` can be more than one, separated by comma (name + type). Arguments are passed ***by value***: Go copies the value of the argument. In general the changes to the parameters do not affect the local variable passed as argument (except for reference types or if we use a pointer).
- `ret-values` can be more than one, separated by comma (type). It is possible to have names return types but is not often used.
- `body`: the brain of a function. It could be a function without, it one implemented in a language other than Go

{{< alert color="success" title="Note" >}}
Comparing to Python, **Go has not**:
- *named arguments* (only positional ones)
- way to set *default values* for the parameters
{{< /alert >}}

the ***type*** of a function is called *signature*: it is the type of the parameters and the type of the return values.
For example for these functions:
```golang
func example(par1 int, par2 string) int {
	fmt.Println("the par2 has this value: ", par2)
	return par1 * 2
}

func exampleNoReturn(par1 int, par2 string) {
	fmt.Println("the par2 has this value: ", par2)
}
```
to get the type (or signature) write:
```golang
fmt.Printf("Type of example is %T\n", example)
fmt.Printf("Type of exampleNoReturn is %T\n", exampleNoReturn)
```
and we get:
```
Type of example is func(int, string) int
Type of exampleNoReturn is func(int, string)
```

## Multiple return values
It is possible to have multiple return values and pick them, for example:
```golang
	// call a function that returns multiple values
	ret1, ret2 := multipleReturns(100, "test")

  /*
	This func has two params (int and string) and returns to result to the
	caller.
*/
func multipleReturns(par1 int, par2 string) (int, string) {
	fmt.Println("the par2 has this value: ", par2)
	return par1 * 2, par2 + "-return"
}
```
To ignore any of the parameters use the '_' character:
```golang
	// call a function that returns multiple values
  ret1, _ := multipleReturns(100, "test")
```
It is possible use named variables as return, instead of single types, for example:
```golang
func multipleNamedReturns(par1 int, par2 string) (val int, name string) {
	fmt.Println("the par2 has this value: ", par2)
	val = par1 * 2
	name = par2 + "-test"
	return
}
```
in the example above we use a technique called `bare return` possible when using named params. It will return all the parameters with the current value.

## Recursion technique
This technique is useful when we need to traverse some content that has a tree structure.
At a first glance could be a terrible exercise for a human and easily for a computer but, with a bit of practice, it will be more readable.

Guess to have this string:
```golang
var webSite = `
<html>
  <head>
    <title>your title here</title>
  </head>
  <body bgcolor="ffffff">
    <center>
      <img src="clouds.jpg" align="bottom">
    </center>
    <hr>
    <a href="http://somegreatsite.com">link name</a>
    is a link to another nifty site
    <h1>this is a header</h1>
    <h2>this is a medium header</h2>
    <a href="http://text2.com">link name</a>
    send me mail at <a href="mailto:support@yourcompany.com">support@yourcompany.com</a>
    <p> this is a new paragraph! </p>
    <p>
      <b>this is a new paragraph!</b>
      <br> <b><i>this is a new sentence without a paragraph break, in bold italics.</i></b>
      <hr>
    </p>
  </body>
</html>`
```
This is a simple webpage. Using the html package we will search all the tag of type <a> using recursion. Remember that HTML is a tree, so html is the first leaf, head and body are siblings and so on.
Write the search func:
```golang
func searchHtml() {
	doc, err := html.Parse(strings.NewReader(webSite))
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlinks1: %v\n", err)
		os.Exit(1)
	}
	for _, link := range visit(nil, doc) {
		fmt.Println(link)
	}
}
```
We do it:
- parse HTML
- call the visit that will get a link []string
The `visit` is our recursive function that will call itself when:
- the html node has a child or a sibling
This is the function:
```golang
func visit(links []string, n *html.Node) []string {
	if n.Type == html.ElementNode && n.Data == "a" {
		for _, a := range n.Attr {
			if a.Key == "href" {
				links = append(links, a.Val)
			}
		}
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		links = visit(links, c)
	}

	return links
}
```
By this way there is a recursion, for simplcity add a print to understand on which node is the visit func:
```golang
func visit(links []string, n *html.Node) []string {
	if n.Type == html.ElementNode && len(n.Parent.Data) > 0{
		fmt.Printf("%s.%s -> ",n.Parent.Data, n.Data)
	}
	if n.Type == html.ElementNode && n.Data == "a" {
		for _, a := range n.Attr {
			if a.Key == "href" {
				links = append(links, a.Val)
			}
		}
	}

	for c := n.FirstChild; c != nil; c = c.NextSibling {
		links = visit(links, c)
	}

	return links
}
```
This output will help to better understand:
```
html.head -> head.title -> html.body -> body.center -> center.img -> body.hr -> body.a -> body.h1 -> body.h2 -> body.a -> body.a -> body.p -> body.p -> p.b -> p.br -> p.b -> b.i -> body.hr -> body.p ->
http://somegreatsite.com
http://text2.com
mailto:support@yourcompany.com
```

## Functions as values

Like all the other type, functions have a value, its own value types and can be assigned to a variable:
```golang
func example(par1 int, par2 string) int {
	fmt.Println("the par2 has this value: ", par2)
	return par1 * 2
}

// assign function to a variable
f := example
// then call it
fmt.Println("result of f() is:", f(1, "10"))
```

- the ***type*** of a function are both parameters and return values in an unique type
- ***value*** of a function is the body
- the ***ZERO value*** of a function type is nil
- if assigned to a variable we can assigne a function with the same type (as point 1)

Follow an example of a function that calls other function passed as parameter. We have this func that has the `f` parameter of type `func(par1 int, par2 string) int`:

```golang
func testWithFuncInput(f func(par1 int, par2 string) int){
  for i:=0; i < 10; i++{
    fmt.Printf("%d,",f(i, string(i)))
  }
  fmt.Println()
}
```

here is a function of the same type:

```golang
func example(par1 int, par2 string) int {
  return par1 * 2
}
```

we can call as the func `testWithFuncInput` passing `example` func as argument. This is very flexible because we can define several functions of the same type
each one with its own logic and change the behaviour of `testWithFuncInput` passing a different function to it.

```golang
// Example to pass as argument of a function a parameter that is another
// function itself
testWithFuncInput(example)
```

## Anonymous functions

It is a function without a name that can be assigned within any expression. Using this technique is possible to write a
closure.

A closure is a function nested in another function that has a reference to one or more variable contained into the host function.
By this way the reference to the variable is active also when the host function ends. This is important to show that the **function values**
are not only code but can have a state.

This is an example:

```golang
func counter() func() int{
  var i int
  return func() int {
    i++
    return i
  }
}
```
the above function has a variable i == 0. Then returns a func of type func() int. By this way the i variable is alive because is a value of the
anonymous function return by counter. So, doing an assignment:

```golang
func useCounter(){
  f := counter() // f is func() of type func() int
  fmt.Printf("call anonymous func, value is %d\n", f()) // -> 1
  fmt.Printf("call anonymous func, value is %d\n", f()) // -> 2
  fmt.Printf("call anonymous func, value is %d\n", f()) // -> 3
}
```
`f` contains a value that is the value of the function counter(). The variable `i` is alive in memory because referenced by the anonymous function.
If we invoke counter() again another `i` variable and anonymous func would be created. Invoking more time the anonymous increments the i variable referenced by it.

This shows how the function value can be not only pure code but have a stated value.

{{% alert title="Important" color="warning" %}}
Remember that using anonymous function to be recursive, you need to declare them into a variable before using, example:
{{% /alert %}}

```golang
func topoSort(m map[string][]string) []string {
  var order []string
  seen := make(map[string]bool)
  var visitAll func(items []string)

  visitAll = func(items []string) {
    for _, item := range items {
      if !seen[item] {
        seen[item] = true
        visitAll(m[item])
        order = append(order, item)
      }
    }
  }

  var keys []string
    for key := range m {
    keys = append(keys, key)
  }

  sort.Strings(keys)
  visitAll(keys)
  sort.Strings(order)
  return order
}
```

as you can see from the code above `visitAll(keys)` calls the `visitAll` anonymous function defined above. Thanks to ***https://github.com/adonovan/gopl.io/blob/master/ch5/toposort/main.go*** for this awesome example. To avoid having a long coding space the other section of the code is omitted, you can find the rest on the repository [here](https://github.com/adonovan/gopl.io/blob/master/ch5/toposort/main.go).
{{% alert title="Tip" color="info" %}}
Remember also that the `visitAll` func only sees the variables defined before its declaration, therefore it doesn't see the `keys` slice.
{{% /alert %}}

### A common trap with the for loop

When using the anonymous funcs you could encounter a trap, a sort of pitfall. Using anonymous func you know that you could have a state as
a value, imagine to have this slice declared on top of your package:
```
var numbers = []int{1, 2, 4, 8, 16}
```  
then you write this simple function:
```golang
func counterNew() {
	var funcs []func() int
	for _, i := range numbers {
		funcs = append(funcs, func() int {
			i++
			return i
		})
	}

	for _, f := range funcs {
		fmt.Println(f())
	}
}
```
in the above code we have:
- a slice of funcs of `type () int`
- as a value we add to the slice func that return the i for loop variable
- range on the funcs slice to invoke the funcs

**What do you guess is the output of the print?** Below the result:
```
17
18
19
20
21
```
Did you imagine?? I guess no...
Conceptually it's the name behaviour seen before, anonymous function has the variable `i` internally and can increment it. The problem
is that the variable is inside a for loop. The loop share the variable reference, therefore every anonymous func is pointing to the
same value, the last of the for loop. So 16 + 1, + 2, + 3...

To solve the problem you need a declared variable that store the `i` value:

```golang
func counterNew() {
	var funcs []func() int
	for _, i := range numbers {
		m := i
		funcs = append(funcs, func() int {
			m++
			return m
		})
	}

	for _, f := range funcs {
		fmt.Println(f())
	}
}
```

`m` variable save the value of `i` and the anonymous func value refers to it. Et voilà, the result is what you expect:

```
2
3
5
9
17
```

## What is a variadic function?
It is a function that can be called with any number of arguments of that type.
An example if the `fmt.Println` function defined as:

```
func Println(a ...interface{})
```

to declare a parameter with any number of args use the '...' before the type. In the example above the Println func accepts
any number of any type (interface{}).
Follow an example:
```golang
func variadic(elems ...string) string {
	var res string
	for _, s := range elems {
		res += fmt.Sprintf("%s-", s)
	}
	return res[:len(res)-1]
}

fmt.Println(variadic("Hello", "from", "me!"))
```
calling the above func you get:
```
Hello-from-me!
```
Internal to the function the `elemes` variable is a slice of string. Changing the code as below you'll get the same result:
```golang
func variadic(elems ...string) string {
	return strings.Join(elems, "-")
}
```
You can unpack also a slice with the opposite operation. For instance, if you have a slice of string as argument you can pass directly to the function by this way (avoiding a cycle):
```golang
fmt.Println(variadic([]string{"Hello", "from", "me!"}...))
```

## Defer mechanism
It is a statement reducing in a single call to a function that is **not** executed at the time of the call but deferred at the end of the
function where `defer` is present. It is useful to ensure that a resource will be closed regardless if there is an error, a return statement, a panic.
For example, opening a file:
```golang
func main() {
	f, err := os.OpenFile("test.txt", os.O_RDONLY, 0666)
  testError(err)
  // some code here ...
	defer f.Close()
}
```
in the func above you ensure that the file will be closed when the `main` function finished, regardless the exit state.
{{< alert title="Keypoints" color="success" >}}
- write `defer` immediately after the correct assignment of the resource
- in case of more `defer` present they are called in the inversed order (last defer is called for first and so on)
{{< /alert >}}

Here is another example taken from the ***Go Programming Language Book***:
```golang
func bigSlowOperation() {
	defer trace("bigSlowOperation")() // don't forget the extra parentheses
	// ...lots of work...
	time.Sleep(3 * time.Second) // simulate slow operation by sleeping
}

func trace(msg string) func() {
	start := time.Now()
	log.Printf("enter %s", msg)
	return func() { log.Printf("exit %s (%s)", msg, time.Since(start)) }
}
```
the `defer` above is on the return function call. **Pay attention**: the extra parenthes means that defer is on the return function call.
Go will wait to invoke this second function after the end of `bigSlowOperation()` call. The anonymous function deferred can access to the enclosing variables (`msg` in this case): this is the reason why msg is seen by the anonymous func.
