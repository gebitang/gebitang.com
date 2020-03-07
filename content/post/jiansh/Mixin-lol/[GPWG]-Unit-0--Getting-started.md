+++
title = "[GPWG]-Unit-0--Getting-started"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Mixin-lol"
]
toc = true
+++

[link to JianShu](https://www.jianshu.com/p/73f5047d9890)

[get programming with go](https://livebook.manning.com/book/get-programming-with-go)

[https://play.golang.org/](https://play.golang.org/)

[https://golang.org/ref/spec](https://golang.org/ref/spec)


## Unit 0 Getting started
### lesson 1 get ready, get set, go

* With the Go Playground you can start using Go without installing anything.
* Every Go program is made up of functions contained in packages.
* To print text on the screen, use the fmt package provided by the standard library.
* Punctuation is just as important in programming languages as it is in natural languages.
* You used 3 of the 25 Go keywords: `package`, `import`, and `func`

[total 25 key words](https://golang.org/ref/spec#Keywords) 

```go
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

## Unit 1 Imperative programming
Travel to Mars.
### lesson 2 a glorified calculator

```
package main

import "fmt"

func main() {
	fmt.Print("My weight on the surface of Mars is ")
	fmt.Print(149.0 * 0.3783) // always 37.83%
	fmt.Print(" lbs, and I would be ")
	fmt.Print(41 * 365 / 687)
	fmt.Print(" years old.")

	fmt.Println()
	fmt.Println("My weight on the surface of Mars is", 70*0.3783, " KG, and I would be ", 36*365/687, " years old.")

	//  %v for the value of the expression. https://golang.org/pkg/fmt/
	fmt.Printf("My weight on the surface of Mars is %v KG, and I would be %v  years old.\n", 70*0.3783, 36*365/687)

	//align text.
	// positive value pads with spaces to the left,
	// negtive number pads with spaces to the right.
	fmt.Printf("%-15v $%4v\n", "SpaceX", 94)
	fmt.Printf("%-15v $%4v\n", "Virgin Galactic", 100)

	const lightSpeed = 299792 // km/s
	var distance = 56000000   //km
	fmt.Println(distance/lightSpeed, " seconds")
	distance = 401000000
	fmt.Println(distance/lightSpeed, " seconds")
}


```
