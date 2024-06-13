There are kind of 2 new kinds of loops in JS:

- Loop For In
- Loop For Of

The first one is used for looping through the properties of an object. The second one is used for looping through an iterable container, such as Arrays, Strings, Maps, NodeLists, and more:

```Plain
for (let x of Books) {
  // do something...
}
```

---

### Sets in JS

Sets are just arrays that cannot have duplicate values inside of them. Some basic methods are:

```Plain
new set() // creates a new set, pass array to this
add() // adds element to the set
delete() // removes an element from the set
has() // returns true if a value exists in a set
forEach() // invokes a callback for each elemnt in the set
values() // returns an iterator with all values in a set

size // returns size
```

---

### Maps in JS

Map - array of key value pairs, keys can be any data type, remembers insertion order of the keys

```Plain
new Map() // creates a new set, pass array to this
set() // sets the value for a key in the map
get() // retrieves the vlaue for a key in the map
delete() // removes an element from the set, specified by the key
has() // returns true if a key exists in a set
forEach() // invokes a callback for each elemnt in the set
entries() // returns an iterator wioth the [key,value] pairs in a map

size // returns size
```

---

### Datatypes

```Plain
typeof "John"                 // Returns "string"
typeof 3.14                   // Returns "number"
typeof NaN                    // Returns "number"
typeof false                  // Returns "boolean"
typeof [1,2,3,4]              // Returns "object"
typeof {name:'John', age:34}  // Returns "object"
typeof new Date()             // Returns "object"
typeof function () {}         // Returns "function"
typeof myCar                  // Returns "undefined" *
typeof null                   // Returns "object"
```

---

There is type conversion in JS, and is done either automatically or my casting.

- *Regular expressions in JS: **

The syntax is:

```Plain
/pattern/modifiers;

// for example
/W3schools/i; // W3schools is pattern to be searched, i means case insensitive
```

In JavaScript, regular expressions are often used with the two string methods: search() and replace().

This kind of thing, you can just reference the documentation when you need this: [https://www.w3schools.com/jsref/jsref_obj_regexp.asp](https://www.w3schools.com/jsref/jsref_obj_regexp.asp)

---

### Error catching in JS

The simple error catching syntax is the same as c++. We have:

```Plain
try
catch
throw

// the only new thing is
finally // defines a block of code that runs regardless of result
```

Here is an example:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript Error Handling</h2>

<p>How to use <b>catch</b> to display an error.</p>

<p id="demo"></p>

<script>
try {
  adddlert("Welcome guest!");
}
catch(err) {
  document.getElementById("demo").innerHTML = err.message;
}
</script>

</body>
</html>
```

Here, `adddlert` is purposely not defined, hence, an error is raised. With `throw`, we can throw custom errors (strings, numbers, or objects) and control program flow. e.g.

```Plain
throw "Too big";    // throw a text
throw 500;          // throw a number
```

Here is an example of using error catching:

```Plain
<!DOCTYPE html>
<html>
<body>

<p>Please input a number between 5 and 10:</p>

<input id="demo" type="text">
<button type="button" onclick="myFunction()">Test Input</button>
<p id="p01"></p>

<script>
function myFunction() {
  const message = document.getElementById("p01");
  message.innerHTML = "";
  let x = document.getElementById("demo").value;
  try {
    if(x == "") throw "empty";
    if(isNaN(x)) throw "not a number";
    x = Number(x);
    if(x < 5) throw "too low";
    if(x > 10) throw "too high";
  }
  catch(err) {
    message.innerHTML = "Input is " + err;
  }
}
</script>

</body>
</html>
```

JavaScript has built in error objects. They contain two properties, `name` and `message`. There are 6 different values that can be returned by the error name property:

```Plain
EvalError // an error has occured in the eval() function
RangeError //a number "out of range" has occured
ReferenceError // an illegal reference found

// A ReferenceError is thrown if you use (reference) a variable that has not been declared

SyntaxError // a syntax error has occured
TypeError // a type error
URIError // 	An error in encodeURI() has occurred
```

In JS, we have something called **hoisting**. Essentially, JS has the default behavior of moving all declarations to the top of the current scope (current script or function). This means that we can declare a variable after it has been assigned and used.

So far, we've seen a couple ways to declare variables, such as `var`, `let`, and `const`.

`let` and `const` are different from `var` in that they cannot be declared after they are defined. so, the below code snippets would be invalid:

```Plain
carName = "Volvo";
let carName;
// reference error

carName = "Volvo";
const carName;
// reference error
```

JavaScript only hoists declarations, not initializations. So we cannot have initialization after it is used (the opposite of C++)

---