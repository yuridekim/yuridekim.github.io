---
title : 'Go: Dependency Injection'
date : 2023-06-27 23:52:00 +0900
categories : [Cloud Computing, Go]
tags : [go, programming, TDD] #소문자만 가능
pinned : 0
---

# Dependency Injection for Go
Dependency Injection and mocking are two crucial concepts at the core of TDD(test-driven development). As its name implies, it operates by injecting dependencies. Commonly, when writing code to create a function, we define a specific type of object to receive as a parameter. However, if you do not want to be influenced by the implementation of this object and want to determine the object yourself and inject it from the outside, you can inject dependencies using interfaces without creating objects.

That is why we say it depenends on behavior, rather than the specific implementation.

## Code Usage
Let's say we want to write a `Greet` function that takes in a name string to greet.

```go
func Greet(name string) {
	fmt.Printf("Hello, %s", name)
}
```

Now take a deeper look at how the `Printf` actually works.
```go
// It returns the number of bytes written and any write error encountered.
func Printf(format string, a ...interface{}) (n int, err error) {
	return Fprintf(os.Stdout, format, a...)
}
```
We notice that Printf is just a mere function that uses `Fprintf`, which uses `os.Stdout` as its location to print. 

```go
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
	p := newPrinter()
	p.doPrintf(format, a)
	n, err = w.Write(p.buf)
	p.free()
	return
}
```

Fprintf requires an `io.Writer` to implement its functionalities. This is where <b>dependency injection</b> comes in. Fprintf doesn't care about where or how the printing takes place, so it rather prefers an interface which is much more flexible than using concrete types.

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

So we see here that `io.Writer` is actually an implementation of an interface.

### Why do we need more degrees of freedom?
In usual cases, while running an application, we often use `stdOut` to print strings out, but for testing `Buffer` is a much more appropriate choice.

```go
func TestGreet(t *testing.T) {
	buffer := bytes.Buffer{}
	Greet(&buffer, "Chris")

	got := buffer.String()
	want := "Hello, Chris"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

If we were to use concrete types, we would need to duplicate the existing `Greet` function just to modify the type the function takes in.

However buffer is also an implementation of the writer interface which makes things much easier. Now we just reuse the `Greet` function.

### Decoupling

```go
package main

import (
	"fmt"
	"io"
	"os"
)

func Greet(writer io.Writer, name string) {
	fmt.Fprintf(writer, "Hello, %s", name)
}

func main() {
	Greet(os.Stdout, "Elodie")
}
```
In the main running application, we choose to use `os.Stdout` while the testing package still uses `buffer`. With dependency injection, the implementation is independent of the objects, but rather can be injected from the outside.

### HTTP connections
```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

func Greet(writer io.Writer, name string) {
	fmt.Fprintf(writer, "Hello, %s", name)
}

func MyGreeterHandler(w http.ResponseWriter, r *http.Request) {
	Greet(w, "world")
}

func main() {
	log.Fatal(http.ListenAndServe(":5001", http.HandlerFunc(MyGreeterHandler)))
}
```

HTTP connections are a good real-life example of dependency injections.
Even with the HTTP implementation as above, http.ResponseWriter also implements Writer interface. With the io.Writer interface, you can also write greetings in http responses and reuse them without additionally implementing the Greet function for http operations.

## How does DI improve efficiency?
1. Representing dependencies in an abstract or general way reduces the amount of knowledge you need to know as you proceed.
2. Isolation from dependencies makes it possible to write test code.
3. Minimize impact when extending or changing code.


## Finale
Dependency injections also aid <b>mocking</b>, to be continued in the next chapter.

This post was influenced by [Learn Go with tests](https://quii.gitbook.io/learn-go-with-tests).