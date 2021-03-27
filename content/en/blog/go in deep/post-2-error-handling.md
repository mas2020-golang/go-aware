---
date: 2020-11-04
title: "The Error handling"
linkTitle: "The Error handling"
description: >
  An article on how Go programs can manage Errors
author: Andrea Genovesi ([@mas2020](https://github.com/mas2020))
---

Errors in Go are non threated as you are used to in other languages. Errors in others languages uses Exception and not ordinary values as in Go. Using Exceptions sometimes lead to an undesiderable outcome: routines in error are reported in a difficult and log stack trace.
By contrast, ***Go uses a simple control flow to threat errors***.

First, we have to understand that: if a command can throw an error, it will be an error value (of type Error) in return of a call. This err value is with the other value that represents a correct execution. For example:
```golang
var x, err := someFunc(someparams)
if err != nil {
  log.Println(err)
}
```
as you can see above err might be `nil` or not. I case is not `nil` we can simply print the error. Using `err` as object to print is equal to use `err.Error()` method that comes back the string error.

Let's see the mechanism on how to manage errors. 

### Return an error to the caller
Usually an internal function returns the error to the caller, for example:
```golang
if err := c.Execute(os.Stdout, computers); err != nil {
  //log.Fatal(err)
  return err
}
```
It is possible to build our error with `fmt.ErrorF()` and return it:
```golang
if err := c.Execute(os.Stdout, computers); err != nil {
  //log.Fatal(err)
  return fmt.Errorf("an error occurred during execution of the template %v", err)
}
```
the tecnique above is useful to enrich the message in a chain.
When a critical error occurred we can exit to the program by two ways:
- with `os.Exit(1)` (usually the result code for an error is 1 or 2)
- with `log.Fatalf` to prefix the error with a date a time information

It is possible to redirect error directly to the standard error with:
```golang
if err := Ping(); err != nil {
    fmt.Fprintf(os.Stderr, "ping failed: %v; networking disabled\n", err)
}
```
