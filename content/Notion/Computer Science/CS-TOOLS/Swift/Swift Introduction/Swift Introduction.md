### Swift Syntax

This document will contain an introduction to Swift:

---

**Variables**:

To work with variables, we do:

```Swift
let someString = "string" // constant
var optionalString: String? = "this is optional" // like Java Optional
```

---

**Functions**:

Working with functions in Swift is similar to functions in other langauges:

```Swift
func add(num1 first: Int, num2 second: Int) -> Int {
    return first + second
}

print(add(num1: 1, num2: 2))

func add2(_ num1: Int, _ num2: Int) -> Int {
    return num1 + num2
}

print(add2(1, 2))
```

The above is an example of 2 ways we can define and call functions. Swift defines `internal` and `external` labels. What this is that `external` label is the name of what is passed as an argument. It is then mapped to an `internal` label, which is in the scope of the function. To avoid this, and use the same label, use the second option above.

Closures are similar to anonymous functions or lambdas in other programming languages.

In addition, we can pass functions as parameters:

```Swift
// we can do something like this
let add: (Int, Int) -> Int =
  {
    (lhs: Int, rhs: Int) -> Int in
      return lhs + rhs
  }

add(30,30)
// in the above, we are assiging a "function" type to a variable, much like javascript
func customAdd {
  _ lhs: Int,
  _ rhs: Int,
  using function: (Int, Int) -> Int) -> Int {
    return function (lhs, rhs)
}
```

Now, say we want to be able to call the `customAdd` function. Here is how we would do so:

```Swift
customAdd(20,30, using: {
    (lhs: Int, rhs: Int) -> Int in
      return lhs + rhs
  }

// the above is if we explcitly define the function when calling. However, Swift likes to shorten
// this whenever we have a closure as the last argument:
customAdd(20,30) {
  (lhs: Int, rhs: Int) -> Int in
    return lhs + rhs
  }
```

It's important to remember that these are essentially lambdas. For instance, we use this to sort:

```Swift
let nums = [1,2,7,5,6,345,33,27]
num
s.sorted(by: <) // this takes 2 numbers at a time, and compares them. Hence, sorting
```

---

**Structures**:  
Just like C++, basically scuffed versions of classes. These data structures do not have the intention of being mutated. Consider:  

```Swift
struct Car {
    var type: String
    mutating func changeType(newType: String) {
        self.type = newType
    }
}
// if type was a let statement, then we can never change, as it is a constant
// but for var, we can. But, as structs are inherently not designed for these kinds of things,
// we will need to mark the method as 'mutating'. Not great though. If need to mutate or change use a class instead
```

---

**Computed Property**:  
In Swift, a computed property is a property that doesn't store a value directly but provides a getter and/or a setter to compute its value dynamically. The value of a computed property is determined based on other properties, variables, or calculations.  

Here's an example of a computed property in Swift:

```Swift
struct Circle {
    var radius: Double

    var area: Double {
        return Double.pi * radius * radius
    }
}


let circle = Circle(radius: 5.0)
print(circle.area) // Output: 78.53981633974483
```

The `area` property is computed each time it is accessed. It doesn't store a separate value but dynamically calculates the result based on the `radius` property.

---

**Enumerations**:

This is `enum`, just like in Java. The general syntax is:

```Swift
enum Animals {
  case cat
  case dog
  case rabbit
}

let cat = Animals.cat
```

The above was a really simple example. Let's consider a harder example:

```Swift
enum Shortcut {
    case fileOrFolder(path: URL, name: String)
    case wwwURL(path: URL)
    case song(artist: String, songName: String)
}

let wwwApple = Shortcut.wwwURL(path: URL(string: "www.apple.com")!)

switch wwwApple {
case .fileOrFolder(path: let path, name: let name):
    path
    name
    break
case .song(artist: let artist, songName: let songName):
    artist
    songName
    break
case .wwwURL(path: let path):
    path
    break
}
```

Now, why did we use a switch? As we have not overloaded/defined a comparison operator for the `enum`, we cannot simply take 2 `enum` selections, and compare them. For now, we can use `switch` statements.

Now, notice how we have used the `switch` statement. It's worth noting that, however, the above format is rarely used. Not many developers like to use `let` over and over again. Instead, the following is how these are often formatted:

```Swift
switch wwwApple {
case let .fileOrFolder(path, name):
    path
    name
    break
case let .song(artist, songName):
    artist
    songName
    break
case let .wwwURL(path):
    path
    break
}
```

Now, what if we want to create a switch statements, that finds something by only one of the associations of the `enum` and does not need to make use of the other? Consider:

```Swift
enum Vehicle {
  case car(manufacturer: String, model: String)
  case bike(manufacturer: String, yearMade: Int)
}

let car = Vehicle.car(manufacturer: "Tesla", model: "X")
let bike = Vehicle.bike(manufacturer: "HD", yearMade: 2023)

// now, we want to switch based on manufacturer only, say:
switch car {
case let .car(manufacturer, _):
  manufacturer
  break
case let .bike(manufacturer, _):
  manufacturer
  break
}

// so, we just add _ to say ignore it
```

But, we can even write a function inside of the `enum` if we want to be able to retrieve certain parts of it:

```Swift
enum Vehicle {
  case car(manufacturer: String, model: String)
  case bike(manufacturer: String, yearMade: Int)

  var manufacturer: String {
    switch car {
      case let .car(manufacturer, _):
        return manufacturer
      case let .bike(manufacturer, _):
        return manufacturer
     }
}
```

In addition, we can have raw values:

```Swift
enum Family: String { // need String here, to tell raw type
  // if we add CaseIterable, we woudld be able to iterate through the cases
  case Father = "Pierre"
  case Mother = "Sophia"
  case Son = "Allan" // "Brian"
  case Daughter = "Brian"
}

// to extract raw type

Family.Father.rawValue
```

---

**Classes**:

Classes in swift are very simple and similar to classes in a lot of other languages. Not much is needed to be said. Here is an example of what they look like:

```Swift
class Tesla {
  let manufacturer = "Tesla"
  let model: String
  let year: Int

  init() { // this is default constructor, use `init` keyword
    self.model = "X"
    self.year = 2023
  }

  init(model: String, year: Int) {
    self.model = model
    self.year = year
  }

  // we add convenience at the front of the constructor to declare that it
  // is using another constructor, something seen in Java
  convenience init (model: String) {
    self.init(model: model, year: 0)
  }
}

// here is how we would create inheritance
class TeslaModelY: Tesla {
  overide init() {
    super.init(model: "Y", year: 2023) {}
  }
}

// now here is how we would use this:

let modelY = TeslaModelY()
```

As usual, we can define these class variables and methods as `private` or `public`, depending on what is needed.

---

**Protocols**:

Protocols are similar to interfaces. They declare methods that must be implemented by the `class` or `struct`. Below is a simple example:

```Swift
protocol MyProtocol {
    var someProperty: Int { get set }
    func someMethod()
}

struct MyStruct: MyProtocol {
    var someProperty: Int

    func someMethod() {
        // Implement the method requirement
    }
}
```

In the above, the `{ get set }` is saying that whatever implements this protocol, must also provide getter and setter implementations for that variable. Protocols are quite flexible, and can inherit from one another. What if protocols want a default implementation for a method? To do, we use `extensions`.

```Swift
protocol MyProtocol {
    func someMethod()
}

extension MyProtocol {
    func someMethod() {
        // Default implementation
    }
}
```

---

**Generics**:  
Like any language, Swift provides support for generics. Say we want some sort of binary function that can support all numbers. In Swift, all numbers implement the  
`Numeric` protocol. So, we could do the following:

```Swift
func perform<N: Numeric> (_ op: (N, N) -> N, on lhs: N, and rhs: N) -> N {
  op(lhs, rhs)
}

// swift will have auto type connection when we use the above function
```

---

**Optionals**:

In Swift, `optionals` are a feature that allows you to represent the absence of a value. They provide a way to indicate when a variable or property may have a value or may be nil, meaning it has no value at all.

```Swift
var optionalValue: Int? = 10

// another instance of this being used
var optionalString: String?  // Optional variable without an initial value
optionalString = "Hello"
optionalString = nil

// another example
func divide(_ a: Int, by b: Int) -> Int? {
    if b != 0 {
        return a / b
    } else {
        return nil
    }
}

let result = divide(10, by: 2) // result = 5
let invalidResult = divide(10, by: 0) // invalidResult = nil

// this is the most example
let stringValue = "sds"
let intValue = Int(stringValue) // this returns an optional
// if the conversion is successful, we will get a value in intValue. However, if the conversion is not successfull
// (string is not a number), intValue will be a nil value instead of throwing an error
```

Now, how do we unwrap an optional? Consider the following:

```Swift
func validateUsername(_ username: String?) {
    guard let unwrappedUsername = username else {
        print("Username is nil")
        return
    }

    guard unwrappedUsername.count > 0 else {
        print("Username is empty")
        return
    }

    // Continue with username validation logic
    print("Validating username: \\(unwrappedUsername)")
}
```

The `username` variable is an optional. Like how in Java, we used `isPresent()` to see if the optional had data, in Swift, we often use `Optional Binding` with `if let` or `guard let`.

---

**Error Handling**

In Swift, error handling is achieved using a combination of `do`, `try`, and `catch` statements. This approach allows you to handle errors that may occur during the execution of a block of code.

Here's an example of how to work with error catching in Swift:

```Swift
enum MyError: Error {
    case errorOne
    case errorTwo
}

func doSomething() throws {
    // Code that may potentially throw an error
    throw MyError.errorOne
}

do {
    try doSomething()
    // Code to be executed if no error is thrown
} catch MyError.errorOne {
    // Code to handle errorOne
} catch MyError.errorTwo {
    // Code to handle errorTwo
} catch {
    // Code to handle other types of errors
}
```

Inside the do block, we use the try keyword to indicate that the code may throw an error. If an error is thrown within the do block, the execution immediately jumps to the corresponding catch block that matches the thrown error.

In the catch blocks, you can handle specific error cases by matching the thrown error with a specific catch clause. If no specific catch block matches the thrown error, the general catch block at the end will catch and handle any other errors.

You can have multiple catch blocks to handle different error cases. The order of the catch blocks is important, as the first matching catch block will be executed.

---

**Collections:**

In Swift, collections are data structures that allow you to store and manipulate multiple values in an organized manner. Swift provides several collection types, including arrays, sets, and dictionaries, each serving different purposes.

- Arrays: An array is an ordered collection of elements of the same type. Elements in an array are accessed using zero-based indices. Here's an example:

```Swift
var fruits = ["Apple", "Banana", "Orange"]
fruits.append("Mango")
fruits.remove(at: 1)
print(fruits) // Output: ["Apple", "Orange", "Mango"]
```

- Sets: A set is an unordered collection of unique elements. Sets are useful when you need to perform operations like intersection, union, and difference on collections. Here's an example:

```Swift
var numbers: Set<Int> = [1, 2, 3, 4, 5]
numbers.insert(6)
numbers.remove(3)
print(numbers) // Output: [1, 2, 4, 5, 6]
```

- Dictionaries: A dictionary is an unordered collection of key-value pairs. Each value in a dictionary is associated with a unique key. Here's an example:

```Swift
var scores = ["Alice": 95, "Bob": 80, "Charlie": 75]
scores["Alice"] = 98
scores["David"] = 85
print(scores) // Output: ["Alice": 98, "Bob": 80, "Charlie": 75, "David": 85]
```

- Ranges: A range represents a sequence of values. It can be used to iterate over a sequence or to represent a subset of values. Here's an example:

```Swift
let range = 1...5
for number in range {
    print(number)
}
// Output:
// 1
// 2
// 3
// 4
// 5
```

Overall, same as always. The syntax of these follows those of Python very closely.