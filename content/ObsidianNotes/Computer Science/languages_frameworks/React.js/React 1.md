These notes will serve as a review of the syntax and functionality of React.js.

⇒ **This first file will review some important JS knowledge**

## ES6 Classes

Very simple, we can define classes in javascript (and typescript) and establish inheritance:

```TypeScript
// this is the vehicle class
class Vehicle {
	constructor(name: string) {
		this.brand = name;
		}

	getVehicleBrand() {
		return 'I have a ' + this.brand;
	}
}

class Car extends Vehicle { // establsih inheritance 
	constructor(name, model) {
		super(name); // super class constructor 
		this.model = model;
	} 

	getCarModel() {
		return 'The model is ' + this.model;
	}
}
```

## ES6 Arrow Functions

These simplify writing functions, especially when we are defining a function as we are passing it to another function. What’s worth noting here is the differences that occur if we use arrow functions for defining class methods:

```TypeScript
class Header {
  constructor() {
    this.color = "Red";
  }

//Regular function:
  changeColor = function() {
    document.getElementById("demo").innerHTML += this;
  }
}

const myheader = new Header();

//The window object calls the function:
window.addEventListener("load", myheader.changeColor);

//A button object calls the function:
document.getElementById("btn").addEventListener("click", myheader.changeColor);
```

  

The above `this` refers to the object which has called on the function, so `window` and `button`, respectively. This makes the output of the two functions different. Now, replacing the `changeColor` function with an arrow function changes things:

```TypeScript
class Header {
  constructor() {
    this.color = "Red";
  }

//Arrow function:
  changeColor = () => {
    document.getElementById("demo").innerHTML += this;
  }
}

const myheader = new Header();


//The window object calls the function:
window.addEventListener("load", myheader.changeColor);

//A button object calls the function:
document.getElementById("btn").addEventListener("click", myheader.changeColor);
```

  

For the `this` above, they both refer the the Header object. So, both functions now have this referring to the same thing.

## ES6 Variables

There are three ways of defining variables: `let`, `var`, and `const`

  

⇒ **Var**

- Outside of function, global scope
- Inside function, local scope
- Inside of a block (perhaps a loop), still available outside of block

  

⇒ **Let**

- Limited to the block scope, local

⇒ **Const**

- Limited to block scope, can’t change the value it references through it, but, that value can still change

## ES6 Array Methods

Many useful array methods out there, here, let’s go over some of the more important ones to know. Others, you can just reference when needed.

  

⇒ **.map()**

This allows us to map a function to each element of an array, so, passing each array element to the function, one at a time. Here is an example:

```TypeScript
import React from 'react';
import ReactDOM from 'react-dom/client';

const myArray = ['apple', 'banana', 'orange'];

const myList = myArray.map((item) => <p>{item}</p>)

ReactDOM.render(myList, document.getElementById('root'));
```

## ES6 Destructuring

A very useful mechanism is the ability to destructure. This applies to many data structures:

  

⇒ **Array**

```TypeScript
const fruits = ['Apple', 'Bannana', 'Orange'];
// old way 
const apple = fruits[0];
const bannana = fruits[1];
const ornage = fruits[2];

// with destructuring 
const [apple, bannana, orange] = fruits;
// for arrays, order matters 
```

⇒ **Objects**

```TypeScript
const vehicleOne = {
  brand: 'Ford',
  model: 'Mustang',
  type: 'car',
  year: 2021, 
  color: 'red'
}

myVehicle(vehicleOne);

// old way
function myVehicle(vehicle) {
  const message = 'My ' + vehicle.type + ' is a ' + vehicle.color + ' ' + vehicle.brand + ' ' + vehicle.model + '.';
}

// with destructuring
function myVehicle({type, color, brand, model}) {
  const message = 'My ' + type + ' is a ' + color + ' ' + brand + ' ' + model + '.';
}

// note that the order of arguments does NOT need to match the object
```

## ES6 Spread Operator

This is the `…` that you frequently see. The JavaScript spread operator (`...`  
) allows us to quickly copy all or part of an existing array or object into another array or object.  

⇒ **Arrays**

```TypeScript
const numbers = [1, 2, 3, 4, 5, 6];

const [one, two, ...rest] = numbers;
```

Above, we have stored `one = 1`, `two = 2`, and `rest = [3,4,5,6]`.

⇒ **Objects**

```TypeScript
const myVehicle = {
  brand: 'Ford',
  model: 'Mustang',
  color: 'red'
}

const updateMyVehicle = {
  type: 'car',
  year: 2021, 
  color: 'yellow'
}

const myUpdatedVehicle = {...myVehicle, ...updateMyVehicle}

// so, the myUpdatedVehicle object will look like:
/*

{
	brand: 'Ford',
	model: 'Mustang',
	type: 'car',
	year: 2021,
	color: 'yellow'
}
*/
// matching properties will take on the last object passed 
```

  

## ES6 Ternary Operator

Faster way to express logic gates.

**Syntax:** `condition ? <expression if true> : <expression if false>`