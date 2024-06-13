Comparing the previous two programs, you might notice that functions with a pointer argument must take a pointer:

```Go
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK
```

while methods with pointer receivers take either a value or a pointer as the receiver when they are called:

```Go
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```

For the statement `v.Scale(5)`, even though `v` is a value and not a pointer, the method with the pointer receiver is called automatically. That is, as a convenience, Go interprets the statement `v.Scale(5)` as `(&v).Scale(5)` since the `Scale` method has a pointer receiver. Below is full code for reference:

```Go
package main

import "fmt"

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(2)
	ScaleFunc(&v, 10)

	p := &Vertex{4, 3}
	p.Scale(3)
	ScaleFunc(p, 8)

	fmt.Println(v, p)
}

```

and:

```Go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func Scale(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	Scale(&v, 10)
	fmt.Println(Abs(v))
}
```

For struct methods, we cannot have the same one defined for both pointer and value receivers. Consider:

```Plain
// Pointer receiver method
func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// Value receiver method
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

We need to choose one. If the method needs to modify the receiver struct, you would use a pointer receiver. Otherwise, if the method only needs to operate on a copy of the struct, you would use a value receiver.

## Interfaces

An _interface type_ is defined as a set of method signatures. A value of interface type can hold any value that implements those methods. So, I have something like:

```Go
type Abser interface {
	Abs() float64
}

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	var a Abser
	v := Vertex{3, 4}
	a = v // a Vertex implements Abser
	// a = &v works too
}
```

A type implements an interface by implementing its methods. There is no explicit declaration of intent, no "implements" keyword. Implicit interfaces decouple the definition of an interface from its implementation, which could then appear in any package without prearrangement. Here's an example:

One of the most ubiquitous interfaces is [`Stringer`](https://go.dev/pkg/fmt/#Stringer) defined by the [`fmt`](https://go.dev/pkg/fmt/) package.

````Go
type Stringer interface {
    String() string
}```

A `Stringer` is a type that can describe itself as a string. The `fmt` package (and many others) look for this interface to print values.
```Go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	a := Person{"Arthur Dent", 42}
	z := Person{"Zaphod Beeblebrox", 9001}
	fmt.Println(a, z)
}
````

==nice to know: convert int to string==  
The below is for making the   
`IPAddr` type implement `fmt.Stringer` to print the address as a dotted quad. For instance, `IPAddr{1, 2, 3, 4}` should print as `"1.2.3.4"`.

```Go
package main

import "fmt"

type IPAddr [4]byte

// TODO: Add a "String() string" method to IPAddr.
func (ip IPAddr) String() string {
	result := ""
	for i, num := range ip {
		result += fmt.Sprintf("%d", num)                // notice this line
		if i != len(ip) - 1 {
			result += "."
		}
	}
	return result
}

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\\n", name, ip)
	}
}
```