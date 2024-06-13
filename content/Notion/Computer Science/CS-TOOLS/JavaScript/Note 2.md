### Numbers

JS has only numbers, no `long`, no `int`, etc.

Extra large or extra small numbers can be written with scientific (exponent) notation:

```Plain
let x = 123e5;    // 12300000
let y = 123e-5;   // 0.00123
```

This format stores numbers in 64 bits, where the number (the fraction) is stored in bits 0 to 51, the exponent in bits 52 to 62, and the sign in bit 63.

When using `+` in JS, just remember it adds from left to right. So working with the addition of strings and numbers is simple. JS has `NaN` which means not a number. Arises when I do something like 100 / "apple".

If we go out of bounds (smaller than smallest number or larger than biggest number), JS will return `Infinity` or `-Infinity`, dividing by 0 also causes this.

We can make number objects:

```Plain
let x = new Nunber(500);
let y = 500;
```

However, we should not do this - slow + not useful.

```Plain
x == y // evaluates to true
x === y // false, not same type

x === x // false
x == x// false
// comparing an object with another is always false, can't do it
```

If needed, look at the number methods here: [https://www.w3schools.com/jsref/jsref_obj_number.asp](https://www.w3schools.com/jsref/jsref_obj_number.asp)

---

### Arrays

Arrays is JS are the same as any other language. Can static make one or new one, but, never new one as it is pointless. We can access the entire array by passing the array's name:

```Plain
const cars = ["Saab", "Volvo", "BMW"];
document.getElementById("demo").innerHTML = cars;
```

That fills the HTML with: Saab, Volvo, BMW.

In JS, arrays are objects, so `typeof` returns object for an array.

You can have objects in an Array. You can have functions in an Array. You can have arrays in an Array:

```Plain
myArray[0] = Date.now;
myArray[1] = myFunction;
myArray[2] = myCars;
```

The real power of JS arrays are the built-in methods there are.

```Plain
cars.length   // Returns the number of elements
cars.sort()   // Sorts the array

// below is code using foreach
const fruits = ["Banana", "Orange", "Apple", "Mango"];

let text = "<ul>";
fruits.forEach(myFunction);
text += "</ul>";

function myFunction(value) {
  text += "<li>" + value + "</li>";
}

// add to one using push
const fruits = ["Banana", "Orange", "Apple"];
fruits.push("Lemon");  // Adds a new element (Lemon) to fruits, or,
fruits[fruits.length] = "Lemon";  // Adds "Lemon" to fruits

// warning
const fruits = ["Banana", "Orange", "Apple"];
fruits[6] = "Lemon";  // Creates undefined "holes" in fruits
```

A note about objects and arrays. They are very similar:  
The Difference Between Arrays and Objects  

- In JavaScript, arrays use numbered indexes.
- In JavaScript, objects use named indexes.

There are many array methods and sorting methods in the reference above.

---

### JS Map

The `map()` method creates a new array by performing a function on each array element:

```Plain
const numbers1 = [45, 4, 9, 16, 25];
const numbers2 = numbers1.map(myFunction);

function myFunction(value, index, array) {
  return value * 2;
}
```

### JS Filter

The filter() method creates a new array with array elements that passes a test.

```Plain
const numbers = [45, 4, 9, 16, 25];
const over18 = numbers.filter(myFunction);

function myFunction(value, index, array) {
  return value > 18;
}
```

for complete array reference: [https://www.w3schools.com/jsref/jsref_obj_array.asp](https://www.w3schools.com/jsref/jsref_obj_array.asp)

---

### JS Dates

We create date objects with `new Date()` constructor. There are 4 ways:

```Plain
new Date()
new Date(year, month, day, hours, minutes, seconds, milliseconds)
new Date(milliseconds)
new Date(date string)
```

Reference those if needed.

---

### JS Math

JS has some constants:

```Plain
Math.E        // returns Euler's number
Math.PI       // returns PI
Math.SQRT2    // returns the square root of 2
Math.SQRT1_2  // returns the square root of 1/2
Math.LN2      // returns the natural logarithm of 2
Math.LN10     // returns the natural logarithm of 10
Math.LOG2E    // returns base 2 logarithm of E
Math.LOG10E   // returns base 10 logarithm of E
```

The syntax for Math any methods is : `Math.method(number)`. Some examples of rounding mathj methods are:

```Plain
Math.round(x) // Returns x rounded to its nearest integer
Math.ceil(x) // Returns x rounded up to its nearest integer
Math.floor(x) // Returns x rounded down to its nearest integer
Math.trunc(x) // 	Returns the integer part of x, not rounding
```

See here for full reference: [https://www.w3schools.com/jsref/jsref_obj_math.asp](https://www.w3schools.com/jsref/jsref_obj_math.asp)

`Math.random()` returns a random number between 0 and 1. e.g.

```Plain
// Returns a random integer from 0 to 10:
Math.floor(Math.random() * 11);

// Returns a random integer from 0 to 99:
Math.floor(Math.random() * 100);

// Returns a random integer from 1 to 10:
Math.floor(Math.random() * 10) + 1;

// Returns a random integer from 1 to 100:
Math.floor(Math.random() * 100) + 1;
```