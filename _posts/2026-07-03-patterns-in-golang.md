---
layout: post
title: 'Functional Option and Builder Pattern In Go'
date: 2026-02-02 12:00:00
categories:
  - golang
---

When building an API or creating a library/module that can be consumed, there are two popular patterns in golang that can be used to build an object, the **functional option pattern** and the **builder pattern**. In this article i would explain with some code samples of how to implement it. Let dive into in straightaway


### FUNCTIONAL OPTION PATTERN
Functional option pattern can be used to configure an object via functions. it is constructor styled, great for optional config and very much idiomatic in golang

**Instead of**
```go
NewServer("localhost", 8080, 30, true)
```

**You can have**
```go
NewServer(WithHost("localhost"), WithPort(8080), WithTimeout(30 * time.Second), WithTLS(true))
``` 

####
ADVANTAGES
- Clean Constructors
- Optional Configuration (you can configure the options you want)

####
DISADVANTAGES
- Validation can become scattered. (Validation has be configured in each function)
- Harder to discover available options with autocomplete or documentation


### BUILDER PATTERN






