# Index

- [Index](#index)
- [Overview](#overview)
- [Packages and Modules](#packages-and-modules)
  - [Executable Programs](#executable-programs)
    - [Create a Module](#create-a-module)
    - [Basic Program](#basic-program)
    - [Multiple Folders Program](#multiple-folders-program)
    - [Multiple Programs in a Module](#multiple-programs-in-a-module)
- [Variables](#variables)
  - [Declaration and Initialization](#declaration-and-initialization)
  - [Data Types](#data-types)
    - [Basic Types](#basic-types)
- [Functions](#functions)
  - [Function Syntax](#function-syntax)
  - [Pointers](#pointers)
- [Control Flows](#control-flows)
  - [If/else](#ifelse)
  - [Switch](#switch)
  - [For](#for)
  - [Defer](#defer)
  - [Panic](#panic)

# Overview

Go is a programming language that was developed at Google. It was announced in 2009 as an open-source project by Robert Griesemer, Rob Pike, and Ken Thompson. Since then, Go has been used for developing other well-known technologies like Docker, Kubernetes, and Terraform.

Programs written in Go can run on Unix systems, such as Linux and macOS, and on Windows. Go is notable in part because of its unique concurrency mechanisms, making it easy to write programs that can take advantage of multiple cores at once. It's primarily a strongly and statically typed language, meaning variable types are known at compile-time. It does, however, have some dynamically typed capabilities.

Go has many similarities with C and inherits aspects of C syntax like control-flow statements, basic data types, pointers, and other elements. Both the language's syntax and semantics go beyond C, though. It also draws similarities to Java, C#, Python, and more.

In general, Go tends to borrow and adapt features from other programming languages, while shedding most of the complexity. For example, you can use some object-oriented (OO) programming features and design patterns in Go, but the full OO paradigm isn't fully implemented.

**Principles**

Here are the underlying principle benefits of the Go programming language:

- The Go license is 100% open source.
- Go programs compile to a single self-contained binary, making it easy to share and distribute.
- Go strives to keep the language small and simple, and to do more in fewer lines of code.
- Concurrency is a first-class citizen, and enables any function to be run as a lightweight thread with little programmer effort.
- Go provides automatic memory management including garbage collection.
- Compilation and execution are fast.
- Go requires that all code be used, or else an error is thrown.
- There's official formatting that helps to maintain consistency across projects.
- Go has a large and comprehensive standard library, and many applications can be built without third-party dependencies.
- Go guarantees language backward compatibility with past releases.

**Use Cases**

- Systems level applications
- Web applications
- Cloud-native applications
- Utilities and command line tools
- Distributed systems
- Database implementations

**Hello World**

- https://go.dev/doc/tutorial/getting-started

Create a directory `hello/`.

```bash
$ mkdir hello
$ cd hello
```

Create a file `hello.go` and write the following code.

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

Run your code to see the greeting.

```bash
$ go run hello.go
Hello, World!
```

**Learning Materials**

- Tour of Go https://go.dev/tour/
- The Go Programming Language Specification https://go.dev/ref/spec
- Effective Go https://go.dev/doc/effective_go
- The Little Go Book https://github.com/karlseguin/the-little-go-book/
- https://docs.microsoft.com/en-us/learn/paths/go-first-steps/

# Packages and Modules

- [Organizing a Go module - Go](https://go.dev/doc/modules/layout)
- [Tutorial: Create a Go module - Go](https://go.dev/doc/tutorial/create-module)
- [Naming a module - Go](https://go.dev/doc/modules/managing-dependencies#naming_module)

We can develop a Go project as executable programs with one or more *main* package, or a non-executable package without a *main* package, such as a SDK.

To organize a Go project, we can group the code into *packages*, and group *packages* into *modules*.

> Package - A directory of one or more related `.go` files. In Go, the entire package is compiled in a step.

> Module - A collection of one or more related packages. It also specifies dependencies needed to run our code, including the Go version and the set of other modules it requires.

## Executable Programs

An executable program is created by linking the *main package* with all the packages it imports, transitively. Programs start running in the main package.

- The main package must have package name `main`.
- The main package must declare **one and only one** `main()` function that takes no arguments and returns no value.
- The main package must be unimported.

```go
package main

func main() { … }
```

### Create a Module

Suppose the root directory of the project is `greetings/`. 

First, create a module for this program.

```
go mod init <module-name>

go mod init example/greetings     // We will use this for the following
go mod init example.com/greetings // E.g. you are going to publish your module
```

A `go.mod` file will also be created, and the `module` line should say:

```
module example/greetings
```

**Module naming convention**

- The module name is typically of the following form: `<prefix>/<descriptive-text>`.
- The *prefix* is typically a string that partially describes the module, such as a string that describes its origin.
- The *descriptive text* can be the project name.

### Basic Program

Here is the project structure:

```
// module name: example/greetings
greetings
├── go.mod
├── auth.go
├── auth_test.go
└── main.go
```

We can have our code split across multiple files, all declaring `package main`. 

By convention, we call the file that contains the `main()` function as `main.go`.

To install the program:

```
go install example/greetings
```

### Multiple Folders Program

Here is the project structure:

```
// module name: example/greetings
greetings
├── go.mod
├── main.go
└── auth
    ├── auth.go
    └── auth_test.go
```

The `greetings` package resides in the root directory. However, we only put a `main.go` file in the root directory, so it just declares `package main`.

The `auth` package resides in `greetings/auth` directory, all its files declare `package auth` and can be imported by users with:

```
import "example/greetings/auth"
```

To install the program:

```
go install example/greetings
```

### Multiple Programs in a Module

Here is the project structure:

```
// module name: example/greetings
greetings
├── go.mod
├── hello.go
├── auth
│   ├── auth.go
│   └── auth_test.go
└── cmd
    ├── client
    │   └── main.go
    └── worker
        └── main.go
```

The `greetings` package resides in the root directory, all its files declare `package greetings` and can be imported by users with:

```
import "example/greetings"
```

The `auth` package resides in `greetings/auth` directory, all its files declare `package auth` and can be imported by users with:

```
import "example/greetings/auth"
```

To install the programs:

```
go install example/greetings/cmd/client
go install example/greetings/cmd/worker
```

# Variables

## Declaration and Initialization

**Declare variables**

We use the `var` keyword to declare a variable.

```go
var firstName string
```

We can declare more than one variable in a single line if they are the same type.

```go
var firstName, lastName string
```

> When you declare a variable and don't use it, Go throws a compilation error.

**Initialize variables**

A `var` declaration can include initializers, one per variable, and the type can be omitted because Go will infer it.

```go
var firstName, lastName, age = "John", "Doe", 32
```

Inside a function, the `:=` short assignment statement can be used in place of a `var` declaration with implicit type.

```go
firstName, lastName, age := "John", "Doe", 32
```

Outside a function, every statement begins with a keyword (`var`, `const`, and so on) and so the `:=` construct is not available.

**Declare constants**

We use the `const` keyword to declare a constant.

```go
const HTTPStatusOK = 200
```

> Constants cannot be declared using the `:=` syntax.

> You can declare a constant without using it.

**Declare in one block**

We can declare multiple variable or constants in one block.

```go
var (
    firstName = "John"
    lastName  = "Doe"
    age       = 32
)

const (
    StatusOK              = 0
    StatusConnectionReset = 1
    StatusOtherError      = 2
)
```

> In Visual Studio Code, the `=` operators are automatically formatted when you save your code.

**Naming convention**

- https://go.dev/wiki/CodeReviewComments#variable-names
- https://google.github.io/styleguide/go/decisions#variable-names

## Data Types

Go is a strongly typed language. Every variable you declare is bound to a specific data type and will only accept values that match that type.

In Go, you have four categories of data types:

- Basic types: numbers, strings, and booleans
- Aggregate types: arrays and structs
- Reference types: pointers, slices, maps, functions, and channels
- Interface types: interface

### Basic Types

**Integer numbers**

```
int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point
```

The `int`, `uint`, and `uintptr` types are usually 32 bits wide on 32-bit systems and 64 bits wide on 64-bit systems.

When you need an integer value you should use `int` unless you have a specific reason to use a sized or unsigned integer type.

Note that `int` is a distinct type and not an alias for `int32` or `int64`. In other words, you'll need to cast explicitly when a cast is required.

```go
var integer int = 127
var integer32 int32 = 32767
fmt.Println(int32(integer) + integer32)
```

A `rune` is simply an alias for `int32` data type. It's used to represent a Unicode character (or a Unicode code point). For example, suppose you have the following code:

```go
rune := '伙'
fmt.Println(rune) 

// Result:
// 20249
```

**Floating-point numbers**

```
float32 float64
```

**Complex numbers**

```
complex64 complex128
```

**Boolean**

```go
var featureFlag bool = true
```

**String**

```go
var firstName string = "John"
```

To initialize a string variable, you need to define its value within double quotation marks (`"`). Single quotation marks (`'`) are used for single characters and runes.

**Default values**

Variables declared without an explicit initial value are given their *zero value*.

The zero value is:

- `0` for numeric types,
- `false` for the boolean type, and
- `""` (the empty string) for strings.

# Functions

## Function Syntax

Here's the syntax for creating a function:

```go
func name(parameters) (results) {
    body-content
}
```

A function can take zero or more parameters.

```go
func add(x int, y int) int {
	return x + y
}

// The same as above
func add(x, y int) int {
	return x + y
}
```

A function can return any number of values. When a function returns multiple values, you must use multiple variables to store them, otherwise it won't compile.

```go
func swap(x string, y string) (string, string) {
	return y, x
}

a, b := swap("a", "b")
```

If you don't need one of the return values from a function, you can discard it by assigning the returning value to the `_` variable.

```go
a, _ := swap("a", "b")
```

## Pointers

Go is a "pass by value" programming language. Whenever you pass a value to a function, Go takes that value and creates a local copy (a new variable in memory). Changes you make to that variable in the function don't affect the one you sent to the function.

For example, say you create a function to update a person's name. Notice what happens when you run this code:

```go
package main

import "fmt"

func main() {
    firstName := "John"
    updateName(firstName)
    fmt.Println(firstName)
}

func updateName(name string) {
    name = "David"
}

// Result:
// John
```

If you want the change you make in the `updateName` function to affect the `firstName` variable in the `main` function, you need to use a pointer.

A *pointer* is a variable that contains the memory address of another variable. When you send a pointer to a function, you're not passing a value, you're passing a memory address. So every change you make to that variable affects the caller.

In Go, there are two operators for working with pointers:

- The `&` operator takes the address of the object that follows it.
- The `*` operator dereferences a pointer. It gives you access to the object at the address contained in the pointer.

The type `*T` is a pointer to a `T` value. Its zero value is `nil`.

Notice that the output now shows David instead of John:

```go
package main

import "fmt"

func main() {
    firstName := "John"
    updateName(&firstName)
    fmt.Println(firstName)
}

func updateName(name *string) {
    *name = "David"
}

// Result:
// David
```

Unlike C, Go has no pointer arithmetic.

# Control Flows

## If/else

Syntax:

- The condition doesn't require parentheses `( )`.
- The condition may be preceded by a simple statement, which executes before the condition is evaluated. Variables declared by this statement are only in scope in all `if` branches.

```go
func givemeanumber() int {
    return -1
}

func main() {
    if num := givemeanumber(); num < 0 {
        fmt.Println(num, "is negative")
    } else if num < 10 {
        fmt.Println(num, "has only one digit")
    } else {
        fmt.Println(num, "has multiple digits")
    }
}
```

## Switch

Syntax:

- The condition doesn't require parentheses `( )`.
- You can invoke a function after the condition.
- A `case` statement can include more than one case.

```go
func main() {
    // weekday := time.Now().Weekday().String()
	// switch weekday {
	switch time.Now().Weekday().String() {
	case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday":
		fmt.Println("It's time to learn some Go.")
	default:
		fmt.Println("It's the weekend, time to rest!")
	}
}
```

In some programming languages, you write a `break` keyword at the end of every `case` statement. But in Go, when the logic falls into one case, it will not continue to the next case, unless you use the `fallthrough` keyword.

```go
func main() {
    switch num := 15; {
    case num < 50:
        fmt.Printf("%d is less than 50\n", num)
        fallthrough
    case num > 100:
        fmt.Printf("%d is greater than 100\n", num)
        fallthrough
    case num < 200:
        fmt.Printf("%d is less than 200", num)
    }
}

// Result:
// 15 is less than 50
// 15 is greater than 100
// 15 is less than 200
```

## For

Syntax:

- The expression doesn't require parentheses `( )`.
- Semicolons `;` separate the three components of for loops:
  - An initial statement that's executed before the first iteration (optional).
  - A condition expression that's evaluated before every iteration. The loop stops when this condition is `false`.
  - A poststatement that's executed at the end of every iteration (optional).
- You can use `break` keyword to exit the loop.
- You can use `continue` keyword to skip the current iteration of a loop.

```go
func main() {
    sum := 0
    for i := 1; i <= 100; i++ {
        sum += i
    }
    fmt.Println("sum of 1..100 is", sum)
}
```

Go has no `while` keyword, but you can use a `for` loop instead:

```go
func main() {
    var num int64
    rand.Seed(time.Now().Unix())
    for num != 5 {
        num = rand.Int63n(15)
        fmt.Println(num)
    }
}
```

You can also write a infinite loop with `break` keyword:

```go
func main() {
    for {
        ...
        break
    }
}
```

## Defer

A `defer` statement pushes a function call onto a list. The list of saved calls is executed **after the surrounding function returns**.

A deferred function's arguments are evaluated when the `defer` statement is evaluated.

```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}

// Result:
// 0
```

Deferred function calls are executed in *Last In First Out order* after the surrounding function returns.

```go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}

// Result:
// 3210
```

Deferred functions execute after the surrounding function returns:

```go
func main() {
	fmt.Print(c())
}

func c() int {
	i := 0
	defer func() { i = 2 }()
	return i
}

// Result:
// 0
```

However, deferred functions may read and assign to the returning function's named return values.

```go
func main() {
	fmt.Print(c())
}

func c() (i int) {
	i = 1
	defer func() { i = 2 }()
	return i
}

// Result:
// 2
```

A typical use case of `defer` is to perform various clean-up actions.

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

## Panic

Runtime errors make a Go program panic (crash), such as attempting to access an array by using an out-of-bounds index or dereferencing a nil pointer. You can also force a program to panic.