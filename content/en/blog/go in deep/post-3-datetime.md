---
date: 2020-12-20
title: "Date and time"
linkTitle: "Date and time"
author: Andrea Genovesi ([@mas2020](https://github.com/mas2020))
description: >
  An introduction to Date and Time in Golang
---

Time in Golang is managed by the ***"time"*** package. A Time represents an instant in time with nanosecond precision. The main object is the `time.time` struct:

```golang
type Time struct {
    wall uint64
    ext  int64
    loc  *Location
}
```

{{< alert color="success" title="Note" >}}
As mentioned by the official documentation we have to pass time variables as values, not pointers.
{{< /alert >}}

The main reference to write this article has been the official documentation package that you find [here](https://golang.org/pkg/time/).

{{< alert color="success" title="Tip">}}
The Golang official documentation is not super simple to understad at a first glance, my suggestion is to take a look at the documentation structure first, and read slowly taking a try at the example code. You can learn a lot only copying the code in you environment
and debug it to fix line by line.
{{< /alert >}}
The more important types in the `time` package are:

- **Time**: for managing the structure that contains date and time
- **Duration**: to calculate duration between two different time
- **Location**: to play with the timezone the time is referring to

***What is the Unix Time or Epoch Time?***
Sometimes you could hear that the time is expressed as Unix Time. This time is simply the number of seconds elapsed since 00:00 UTC on
1 January 1970.

## Create a time instance

How to create a time instance? There are several ways to do that, let's see some ways.

- starting from the **actual date and time**:
  
```golang
// get the local timestamp
t1 := time.Now()
```

- passing the argument to the Date function of the `time` package:

```golang
// get a time passing all the arguments (year, month, day, hour, min, sec, nanosecond)
t2 := time.Date(2020, 12, 22, 8,0, 0, 0, time.UTC)
```

- parsing a string into a time object:

```golang
// convert a string representation of time into a time instance
sTime := "2020-11-12 10:50"
t3, err := time.Parse("2006-01-02 15:04", sTime)
```

## Format convention

Unlike other languages Golang do not use the standard format conventions as "YYYY-MM-DD" but a specific date time
representing the format itself.
This is expressed as:

```shell
Mon Jan 2 15:04:05 -0700 MST 2006
```

The single part of a date in Golang can be represented as:
| What      | Placeholder         |
|-----------|---------------------|
| *Day*       | Mon, Monday or 2, 02            |
| *Month*       | Jan, January or 1, 01            |
| *Year*       | 2006 or 06            |
| *Hour*       | 15, 3, 03 (for pm/am)        |
| *Minute*       | 04, 4           |
| *Second*       | 05, 5            |
| *Nanoseconds*       | .000000000 (trailing zero included), .999999999 (trailing zero omitted)            |
| *Offset*       | -0700            |
| *Timezone*       | MST            |

Some examples of format, according to the previous table are:

```shell
2006-01-02 15:04:05
2006-1-02
2006-January-02
2006-01-02 3:4:5 pm
```

A fractional second is represented by adding either a period and zeros to the end of the seconds section, or a period and 9s:

```shell
"2006-01-02 15:04:05.000000"
"2006-01-02 15:04:05.999999"
```

the difference is that with the 0s if the nano secs doesn’t have that cipher a zero is shown instead, with 9s only the present nano secs ciphers are shown, for example:

```golang
t1 := time.Date(2020, 03, 29, 16,18, 22, 918000000, time.UTC)
fmt.Println(t1.Format("2006-01-02 15:04:05.00000"))
fmt.Println(t1.Format("2006-01-02 15:04:05.99999"))
```

the output is:

```shell
2020-03-29 16:18:22.91800
2020-03-29 16:18:22.918
```

Some code examples to clarify better. We start creating a single time and then formatting its output in various string:

```golang
func formatTime(){
  fmt.Println("\n---Format time")
  // date time
  t1 := time.Date(2020, 03, 29, 16,18, 22, 918000000, time.UTC)

  // anonymous func to check a format with an expected result
  do := func(name, layout, expected string) {
    got := t1.Format(layout)
    if expected != got {
      fmt.Printf("error: for %q got %q; expected %q\n", layout, got, expected)
      return
    }
    fmt.Printf("%-27s %q gives %q\n", name, layout, got)
  }

  do("MM-DD-YYYY", "01-02-2006", "03-29-2020")
  do("YYYY-MM-DD", "2006-01-02", "2020-03-29")
  do("YYYY-MM-DD HH:MM:SS", "2006-01-02 15:04:05", "2020-03-29 16:18:22")
  do("YYYY-MM-DD H:MM:SS (am/pm)", "2006-01-02 3:04:05", "2020-03-29 4:18:22")
  do("YYYY-MM-DD HH:MM:SS.milli", "2006-01-02 15:04:05.000", "2020-03-29 16:18:22.918")
  do("YYYY-MM-DD HH:MM:SS.micro", "2006-01-02 15:04:05.000000", "2020-03-29 16:18:22.918000")
  do("YYYY-MM-DD HH:MM:SS.nano", "2006-01-02 15:04:05.000000000", "2020-03-29 16:18:22.918000000")
  do("Short numeric month", "2006-1-02", "2020-3-29")
  do("Short textual month", "2006-Jan-02", "2020-Mar-29")
  do("Long textual month", "2006-January-02", "2020-March-29")
  do("Day of week", "2006 January Monday", "2020 March Sunday")
  do("Day of week", "Mon of January 2006", "Sun of March 2020")
}
```

The `do` internal func passes the layout according to the Golang format and
an expected result. As you can see we have several ways to represent a single date/time as a string.

Output is:

```shell
MM-DD-YYYY                  "01-02-2006" gives "03-29-2020"
YYYY-MM-DD                  "2006-01-02" gives "2020-03-29"
YYYY-MM-DD HH:MM:SS         "2006-01-02 15:04:05" gives "2020-03-29 16:18:22"
YYYY-MM-DD H:MM:SS (am/pm)  "2006-01-02 3:04:05" gives "2020-03-29 4:18:22"
YYYY-MM-DD HH:MM:SS.milli   "2006-01-02 15:04:05.000" gives "2020-03-29 16:18:22.918"
YYYY-MM-DD HH:MM:SS.micro   "2006-01-02 15:04:05.000000" gives "2020-03-29 16:18:22.918000"
YYYY-MM-DD HH:MM:SS.nano    "2006-01-02 15:04:05.000000000" gives "2020-03-29 16:18:22.918000000"
Short numeric month         "2006-1-02" gives "2020-3-29"
Short textual month         "2006-Jan-02" gives "2020-Mar-29"
Long textual month          "2006-January-02" gives "2020-March-29"
Day of week                 "2006 January Monday" gives "2020 March Sunday"
Day of week                 "Mon of January 2006" gives "Sun of March 2020"
```

Golang has a set of predefined formats as constants, here the list:

```golang
const (
    ANSIC       = "Mon Jan _2 15:04:05 2006"
    UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
    RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
    RFC822      = "02 Jan 06 15:04 MST"
    RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
    RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
    RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
    RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
    RFC3339     = "2006-01-02T15:04:05Z07:00"
    RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
    Kitchen     = "3:04PM"
    // Handy timestamps
    Stamp      = "Jan _2 15:04:05"
    StampMilli = "Jan _2 15:04:05.000"
    StampMicro = "Jan _2 15:04:05.000000"
    StampNano  = "Jan _2 15:04:05.000000000"
)
```

Updated information can be found [here](https://golang.org/pkg/time/#pkg-constants]).

## Time Zone and Location

The Location is the offset referred to a particular Time Zone applied to the UTC.
UTC is the universal time (without any offset applied to it).
There two constants that represent the location:
time.Local: referred to the local time zone
time.UTC: referred to the Universal Time
To build a local or UTC create the time passing the constant:

```golang
t2 := time.Date(2020, 12, 22, 13,0, 0, 0, time.UTC)
t2Local := time.Date(2020, 12, 22, 13,0, 0, 0, time.Local)
```

if you print them, you get this:

```shell
t2: 2020-12-22 13:00:00 +0000 UTC
t2Local: 2020-12-22 13:00:00 +0100 CET
```

the second row shows that we have 1 hour more than UTC time.

When we refer to a **Time Zone**, we mean a particular place and an offset from the UTC. A list of database time zones can be found [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). To express a Time into a Time Zone we use the method `In` of the `Time` type. Follow an example of time expressed in several Time Zones:

```golang
t := time.Now()
newYork, _ := time.LoadLocation("America/New_York")
chicago, _ := time.LoadLocation("America/Chicago")
panama, _ := time.LoadLocation("America/Panama")
rome, _ := time.LoadLocation("Europe/Rome")
casablanca, _ := time.LoadLocation("Africa/Casablanca")
 
fmt.Printf("%-27s %q\n", newYork, t.In(newYork))
fmt.Printf("%-27s %q\n", rome, t.In(rome))
fmt.Printf("%-27s %q\n", casablanca, t.In(casablanca))
fmt.Printf("%-27s %q\n", chicago, t.In(chicago))
fmt.Printf("%-27s %q\n", panama, t.In(panama))
```

the output of the previous code is:

```shell
America/New_York            "2020-12-26 03:13:19.134101 -0500 EST"
Europe/Rome                 "2020-12-26 09:13:19.134101 +0100 CET"
Africa/Casablanca           "2020-12-26 09:13:19.134101 +0100 +01"
America/Chicago             "2020-12-26 02:13:19.134101 -0600 CST"
America/Panama              "2020-12-26 03:13:19.134101 -0500 EST"
```

{{< alert color="success" title="Note" >}}
To work with Location, Golang needs the time zone databases that may not be present on all systems. LoadLocation takes a look in order:
directory or uncompressed zip file named by the ***ZONEINFO*** environment variable
some known installation locations on Unix systems
looks in $GOROOT/lib/time/zoneinfo.zip
{{< /alert >}}

## Duration

The duration is the interval of time in **nanoseconds (int64)** between two instants or time instances. The largest duration can be represented is approximately 290 years.

### Duration constants

There are several constants in the time package that represent the Duration:

```golang
const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```

in the above code Nanosecond is of type Duration (an aliasing for int64) and is equal to 1. Microsecond is equal to 1000 nanosecond
and so on.
For example, to represent a Duration of 10 minutes type:

```golang
d := time.Duration(10 * time.Minute)
// or simply
d := 10 * time.Minute
```

Once you have a Duration instance you can get the single time as:

```golang
d := time.Duration(10 * time.Minute)
fmt.Printf("Duration as string is %v\n", d)
fmt.Printf("%f hours \n", d.Hours())
fmt.Printf("%f minutes \n", d.Minutes())
fmt.Printf("%f seconds \n", d.Seconds())
fmt.Printf("%d millisec \n", d.Milliseconds())
fmt.Printf("%d microsec \n", d.Microseconds())
fmt.Printf("%d nanosec \n", d.Nanoseconds())
```

### Parse from a string

When Golang shows a Duration as string a formatting func is applied, for example to know the Duration from tOne and tTwo:

```golang
tOne := time.Date(2020, 03, 29, 16,18, 22, 0, time.UTC)
tTwo := time.Date(2020, 03, 29, 17,18, 22, 0, time.UTC)
fmt.Printf("Interval of time from tOne and tTwo is %v \n", tTwo.Sub(tOne))
```

we subtract tOne from tTwo and Golang gives the following output:

```shell
Interval of time from tOne and tTwo is 1h0m0s
```

Before to parse a Duration from a string we need to know that in Golang a duration string is (as mentioned by official guide) a possibly signed sequence of decimal numbers, each with optional fraction and a unit suffix, such as "300ms", "-1.5h" or "2h45m".
Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".
Follow an example of parsing some string into valid Duration instances (error check is omitted for keep the code clean):

```golang
func parseDuration(){
  d1, err := time.ParseDuration("100s")
  d2, err := time.ParseDuration("1h20s")
  d3, err := time.ParseDuration("1000ms")
  fmt.Printf("Duration as string is %v with %f seconds \n", d1, d1.Seconds())
  fmt.Printf("Duration as string is %v with %f seconds \n", d2, d2.Seconds())
  fmt.Printf("Duration as string is %v with %f seconds \n", d3, d3.Seconds())
}
```

### Round a Duration

It is possible to round a duration (as it was a number) using the `Round()` method of the Duration type. Round is applied
on the duration passed as argument. The rounding behavior for halfway values is to round away from zero.
Remember that the round is applied based on the Duration scale passed as argument (only if it is possible to apply it).
For example:

```golang
d2, err := time.ParseDuration("1h32s")
fmt.Printf("d.Round(%6s) = %s\n",d2,  d2.Round(time.Second).String())
fmt.Printf("d.Round(%6s) = %s\n",d2,  d2.Round(time.Minute).String())
```

In the first example rounding by Second remains unchanged as could not be applied. The other example instead rounds at 1 minute.

## Add and subtract time

Using the `Time` type is possible to add and subtract time from a specific time instance. The result is a new instance of `Time`.
Some examples:

```golang
t := time.Date(2019, 12, 22, 13, 0, 0, 0, time.UTC)
// add examples
t1 := t.Add(time.Second * 10)
t2 := t.Add(time.Minute * 10 + time.Second * 10)
t3 := t.Add(time.Hour * 2)

fmt.Printf("start = %v\n", t)
fmt.Printf("after 10 secs = %v\n", t1)
fmt.Printf("after 1 min and ten secs = %v\n", t2)
fmt.Printf("after 2 hours = %v\n", t3)

// subtract example
t1 = t1.Add(time.Second * -10)
t2 = t2.Add(time.Minute * -10 + time.Second * -10)
t3 = t3.Add(time.Hour * -2)
fmt.Printf("before 10 secs = %v\n", t1)
fmt.Printf("before 1 min and ten secs = %v\n", t2)
fmt.Printf("before 2 hours = %v\n", t3)
```

In the previous example at `Time` t are added some durations. Then the opposite operation to get the original Time.
To subtract a duration to get a new `Time` object use a negative Duration.
The `Sub()` method instead calculates a duration subtracting a time to the other. For example:

```golang
// subtract two time
begin := time.Now()
end := time.Now().Add(1 * time.Second)
fmt.Printf("elapsed time = %v\n", end.Sub(begin))
```

If *begin* is greater than *end* the result will be negative.

## Compare time

To compare times use the Before, After, and Equal methods:

```golang
t := time.Date(2020, 12, 10, 10, 0, 0, 0, time.UTC)
t2 := time.Date(2020, 9, 1, 12, 0, 0, 0, time.UTC)
fmt.Printf("time %v is before %v: %v\n", t, t2, t.Before(t2))
fmt.Printf("time %v is after %v: %v\n", t, t2, t.After(t2))
fmt.Printf("time %v is equal %v: %v\n", t, t2, t.Equal(t2))
```

Output of the previous code is:

```shell
time 2020-12-10 10:00:00 +0000 UTC is before 2020-09-01 12:00:00 +0000 UTC: false
time 2020-12-10 10:00:00 +0000 UTC is after 2020-09-01 12:00:00 +0000 UTC: true
time 2020-12-10 10:00:00 +0000 UTC is equal 2020-09-01 12:00:00 +0000 UTC: false
```

The equal method compares two instants and is better than using `==`. Two instants are different also if the code calculates time one
following the other, for example:

```golang
t = time.Now()
t2 = time.Now()
fmt.Printf("time %v is equal %v: %v\n", t, t2, t.Equal(t2))
```

Output is:

```shell
time 2020-12-28 07:55:57.620924 +0100 CET m=+0.000215560 is equal 2020-12-28 07:55:57.620924 +0100 CET m=+0.000215686: false
```

## Conversion from and to Unix time

Sometimes the time is expressed in Unix time. In Golang we can manage this. Starting from a Unix time we can get
the corresponding `Time` as:

```golang
unixT := time.Unix(1020002034,0)
fmt.Printf("Time from Unix time is %v\n", unixT)
```

Output is:

```shell
Time from Unix time is 2002-04-28 15:53:54 +0200 CEST
```

`Unix` method on a `Time` instance, returns the Unix time, sec seconds and nsec nanoseconds since January 1, 1970 UTC.

```golang
t := time.Now()
fmt.Printf("Unix time from time.Now() is %d\n", t.Unix())
fmt.Printf("UnixNano time from time.Now() is %d\n", t.UnixNano())
```

Output is:

```shell
Unix time from time.Now() is 1609143092
UnixNano time from time.Now() is 1609143092251696000
```

## Get the elapsed time of a function

We can decorate a function *"simply"* passing the function to execute as a ***parameter*** of another function.
In our case it is useful to measure how long a specific function takes. Below the code for the decoration:

```golang
func timing(f func(p ...interface{}), args ...interface{})  (d time.Duration) {
  start := time.Now()
  f(args...) // our function to execute
  end := time.Now()
  return end.Sub(start)
}
```

the `timing` function accept two parameters:

- the ***func*** to execute that accepts any number of input parameters
- the ***args*** to pass at the function to execute

`p` and `args` are slices of the interface type (anything). The function:

- gets the start time
- executes the function passing interface args (... means unpack the []interface{} slice into single elements)
- gets the end time
- returns the ***Duration*** of the function to execute

We can pass any func of the same type as an input of `timing`. Let's see an example, create a simple testing function:

```golang
func testTiming(i ...interface{}){
  n, ok := i[0].(int)
  if !ok {
    panic("the first parameter must be an int...")
  }
  fmt.Println("executing testFunc...")
  time.Sleep(time.Duration(n) * time.Second)
}

d := timing(testTiming, 1)
fmt.Printf("elapsed time of testFunc was %v", d)
```

An example of output:

```shell
executing testFunc...
elapsed time of testFunc was 1.002484924s
```

- `testTiming` has the same type -> `func (i ...interface{})` of the first parameter of the `timing` func
- 1 is the argument that is passed to the `testTiming` func by the `timing`

To have a generic decorator we needed to write our function with the `(i ...interface{})` as a parameter. By this way, we can use it with a generic bunch of functions to decorate.
