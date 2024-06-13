### this

The

```Plain
this
```

keyword is used as a reference to an object.

---

There is a shorter way to write functions in JS, and its called arrow function notation.

```Plain
let myFunction = (a, b) => a * b;
```

This arrow notation allows for shorter code when writing functions. However, there is a very critical difference between the two ways of writing a function, and that is what `this` refers to.

In a normal function definition, `this` is the object that has called the function. However, in arrow functions, `this` is the object that defined the arrow function. To better see this, look at the bottom two code snippets here: [https://www.w3schools.com/js/js_arrow_function.asp](https://www.w3schools.com/js/js_arrow_function.asp)

---

### Classes

Now, JS also supports classes, which like c++, are templates for objects. Here is the general syntax for class definitions:

```Plain
class ClassName {
  constructor() { ... }
  method_1() { ... }
  method_2() { ... }
  method_3() { ... }
}
```

Before now, we've always written the js logic inside of the html file, which is not the best for encapsulation and modularization. Now, we make make js files and place our js code there.

To do so, we make js files and fill them with things we want to **export. ** There are two types of export: Names and Default

- * Names Exports:**

```Plain
export const name = "Jesse";
export const age = 40;

// or

const name = "Jesse";
const age = 40;

export {name, age};
```

- * Default Exports: **

```Plain
const message = () => {
const name = "Jesse";
const age = 40;
return name + ' is ' + age + 'years old.';
};

export default message;
```

- *NOTE: YOU CAN ONLY HAVE ONE DEFAULT EXPORT PER FILE **

Now, in the html file, we want to be able to access these files, thus we need to import them.

- * Importing Named Imports:**

```Plain
import { name, age } from "./person.js";
```

- * Importing Default Exports**

```Plain
import message from "./message.js";
```

---