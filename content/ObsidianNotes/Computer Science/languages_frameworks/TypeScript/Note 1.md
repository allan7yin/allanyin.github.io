So, why should we even use `TypeScript` in the first place? There are **2** main reasons we prefer it over JavaScript:

1. `TypeScript` adds a type system to help you avoid many problems with dynamic types in JavaScript
2. `TypeScript` implements future features of JavaScript (ES Next… no idea what this is) so that we can use them today

  

Recall that JavaScript is dynamically types. This makes the following legal:

```JavaScript
console.log(typeof(box)); // undefined

box = "Hello";
console.log(typeof(box)); // string

box = 100;
console.log(typeof(box)); // number
```

The variable box is capable of changing types, which makes mistakes prone:

```JavaScript
let x = 1;
let y = 2;

function add(x,y) {
	return x + y;
}

add(x.value, y.value);
// although x and y are integers, .value method returns a string. So, 
// add(x.value, y.value) returns the string '12' instead of the integer 3
```

Dynamic types can have many benefits. It makes the language very loose and flexible, allowing for some things to be done quicker and in less code. However, it introduces the risk of more error. In something like `C++`, doing something like `20 + ‘allan'` would produce an error, and the code would terminate. However, in JavaScript, no error is produced, and so, the code seemingly “works”.

  

`TypeScript` solves this issue of dynamic types. An issue that often arrises in JavaScript is calling methods that don’t exist, or calling object types, that have not yet been defined. Consider the following JavaScript code:

```JavaScript
 function getProduct(id){
  return {
    id: id,
    name: `Awesome Gadget ${id}`,
    price: 99.5
  }
}

const showProduct = (name, price)  => {
  console.log(`The product ${name} costs ${price}$.`);
};
```

Two common errors are:

```JavaScript
const product = getProduct(1);
console.log(`The product ${product.Name} costs $${product.price}`);
// .Name doesn't exist, undefined, name is
```

and

```JavaScript
const product = getProduct(1);
showProduct(product.price, product.name);
// passed parameters are in the wrong order
```

TypeScript is able to solve these issues. Essentially, instead of returning an object like that, we first sort of create a “class” or a “shape” of the object we wish to return. So, something like:

```TypeScript
interface Product {
	id: number,
	name: string,
	price: number,
}

function getProduct(id) : Product{
// here, we explicitly say that we are returning a Product type 
  return {
    id: id,
    name: `Awesome Gadget ${id}`,
    price: 99.5
  }
}

const product = getProduct(1);
console.log(`The product ${product.Name} costs $${product.price}`);
// this line will show an error, as we know Name is a not a method of a product 
```

To solve the passing in arguments in the wrong order above, we can assign types:

```TypeScript
const showProduct = (name: string, price:number)  => {
  console.log(`The product ${name} costs ${price}$.`);
};
```

This will show an error if we pass a number to the name argument.

## Summary

- JavaScript is dynamically typed. It offers flexibility but also creates many problems.
- TypeScript adds an optional type system to JavaScript to solve these problems.