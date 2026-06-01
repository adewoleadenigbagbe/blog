---
layout: post
title: 'Functional Option and Builder Pattern In Go'
date: 2026-02-02 12:00:00
categories:
  - golang
---

When building an API or creating a library/module that can be consumed, there are two popular patterns in golang that can be used to build an object, the **functional option pattern** and the **builder pattern**. In this article i would explain with some code samples of how to implement it. Let dive into in straightaway


### Funtional Option Pattern
Functional option pattern can be used to configure an object via functions. it is constructor styled, great for optional config and very much idiomatic in golang

**Instead of**
```go
NewServer("localhost", 8080, 30, true)
```

**You can have**
```go
NewServer(WithHost("localhost"), WithPort(8080), 
WithTimeout(30 * time.Second), WithTLS(true))
``` 

#### Advantages
- Clean Constructors
- Optional Configuration (you can configure the options you want)

#### Disadvantages
- Validation can become scattered. (Validation has be configured in each function)
- Harder to discover available options with autocomplete or documentation


### Builder Pattern
The builder pattern can be used to configure object via chained methods. It allows the fluent readablility, useful when the object creation is complex and when you want the validation to be centralized in a method whn build the object.

```go
func (sb *ServerBuilder) Build() (Server, error) {
	if sb.server.Host == "" {
		return Server{}, errors.New("host is required")
	}

	if sb.server.Port == 0 {
		return Server{}, errors.New("port is required")
	}
	return sb.server, nil
}
```

#### Advantages
- Centralized validation
- Step by step for complex object construction

#### Disadvantages
- Boilerplate - methods
- Unidiomatic in Go (feels you are programming in OOP Languages)


### Summary

| Functional Options pattern          | Builder Pattern                   |
| ----------------------------- | --------------------------------- |
| Configuration via functions   | Configuration via chained methods |
| Immutable-style configuration | Step-by-step mutation             |
| Constructor-centered          | Builder-centered                  |
| Lightweight                   | More structured                   |
| Very idiomatic in Go          | Less common in Go                 |
| Great for optional config     | Great for complex workflows       |


You can get code samples for this post here [Functional Option and Builder Pattern In Go](https://github.com/adewoleadenigbagbe/Golang-Blog-Code-Samples/tree/main/patterns)




