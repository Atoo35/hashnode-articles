---
title: "Functional Options in GoLang"
datePublished: Fri Jul 14 2023 15:27:56 GMT+0000 (Coordinated Universal Time)
cuid: clk2qgqr4000809jpa8fy6nuv
slug: functional-options-in-golang
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1689348392641/c92272f2-9219-4944-91e2-3c1e1a00758c.png
tags: design-patterns, go, golang, functional-options

---

What even are functional options? In Golang, they are a design pattern that accommodates frequently changing parameters or configuration variables for a package. Say you are developing a package and to initialize it, some configurations are needed and you want to make the package as configurable as possible for the client to use. At the same time, you want to make sure you don't introduce breaking changes when a new version of the package is published. What do you do?

Making a new constructor for each combination of configurations is an option but the question is what if you have like a toooooooon of configs? It's not possible or ideal to create that many constructors right? Another way is to create a struct type called "Config" and pass it in the constructor and handle it. Sure this is good when you have a small amount of configs and don't change a lot, but what about breaking changes in the newer version?

This is where functional options come into play. Let's understand the design pattern with an example. Take an example of a package you are creating for initializing a server. Below is the simplest code of `server.go` file.

```go
package server

import "fmt"

type Server struct {
	host string
	port int
}

func New(host string, port int) *Server {
	return &Server{host: host, port: port}
}

func Start(s *Server) {
	fmt.Printf("Server started at %s:%d\n", s.host, s.port)
}
```

Here is how your client would initialize the package in `main.go` :

```go
package main

import "github.com/Atoo35/functionaloptions/server"

func main() {
	s := server.New("localhost", 8080)
	server.Start(s)
}
```

But what if the client does not pass in the `host` and the `port`? Yes, you can easily handle the default case in the `New` func, but right now we are not allowing for customizability in the code.

Let's use the Functional Options design pattern to fix that. This is what the `server.go` file would look like:  

```go
package server

import "fmt"

type Server struct {
	host string
	port int
}

type Option func(*Server)

func WithHost(host string) Option {
	return func(s *Server) {
		s.host = host
	}
}

func WithPort(port int) Option {
	return func(s *Server) {
		s.port = port
	}
}

func New(opts ...Option) *Server {
	s := &Server{host: "localhost", port: 8080}
	for _, opt := range opts {
		opt(s)
	}
	return s
}

func Start(s *Server) {
	// start server
	fmt.Printf("Server started at %s:%d\n", s.host, s.port)
}
```

Let's understand what exactly is happening. First off, we change the `New` function to be a [variadic function](https://www.digitalocean.com/community/tutorials/how-to-use-variadic-functions-in-go). In the first line we set up the server with the default values.

> Note: This can be done by using another function that returns the default server to make it better for testing.

Then we loop over the options sent in the function call and apply those options to the server `s` we initialized before. Since we are passing on the reference of `s`, we can be sure that the same `s` has been applied to the options passed in.

Let us understand how is `opt` in the loop a function we are passing the server type to. We created a type `Option` just after imports which is a function and accepts the type `Server` as a parameter.  

```go
type Option func(*Server)
```

This is then used with the other function `WithHost` which accepts `string` value in the input and returns the type `Option`. Remember, the type `Option` is a function accepting `Server` which is why the `WithHost` function needs to return a function accepting `Server` as a type. This is why we have the return function call inside the function and we are only setting the `host` property here.

```go
func WithHost(host string) Option {
	return func(s *Server) {
		s.host = host
	}
}
```

Similarly, we have the `WithPort` function which sets the `port`. Since the `New` function is accepting functions of type `Option` we can use the passed-in parameters like a function because they indeed are one.

Onto the `main.go` file now. The file should look like this:  

```go
package main

import "github.com/Atoo35/functionaloptions/server"

func main() {
	s := server.New(
		server.WithPort(8081),
	)
	server.Start(s)
}
```

This is quite similar to how we initialized the server before with the change being the passed-in parameters. Instead of passing in the port, we passed the function `WithPort` we wrote before with the value `8081`. This will set the port value to 8081 and should produce the output as

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689347830069/c5f6e9cc-8cdb-4f90-a29a-3a1cb140c79c.png align="center")

Since we had configured the server to have default values, `host` was defaulted to `localhost`. To add some other host simply pass in the `WithHost` function in the `New` function call like this:  

```go
s := server.New(
    server.WithPort(8081),
    server.WithHost("mydomain"),
)
```

The output would look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689347988035/57d900f3-7462-4188-8457-f90891ef644f.png align="center")

  

That's it for today folks. This is how you implement the Functional Options Design Pattern in Go.

I hope this was a good read and would love to get some feedback in the comments!!

### Support

If you liked my article, consider supporting me with a coffee ☕️ or some crypto ( ₿, ⟠, etc)

Here is my public address `0x7935468Da117590bA75d8EfD180cC5594aeC1582`

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/default-yellow.png align="left")](https://www.buymeacoffee.com/atoo)

### Let's connect

[**Github**](https://github.com/Atoo35)

[**LinkedIn**](https://www.linkedin.com/in/atoo35)

[**Twitter**](https://twitter.com/atharva_35)

### Feedback

Let me know if I have missed something or provided the wrong info. It helps me keep genuine content and learn.