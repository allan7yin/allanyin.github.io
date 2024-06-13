**Type Annotation in TypeScript**

  

- TypeScript uses type annotations to explicitly specify types for identifiers such as variables, functions, and objects.
- TypeScript uses the syntax `: type` after an identifier as the type annotation, where `type` can be any valid type.
- Once an identifier is annotated with a type, it can be used as that type only. If the identifier is used as a different type, the TypeScript compiler will issue an error.

```TypeScript
let variableName: type;
let variableName: type = value;
const constantName: type = value;
```

  

Also consider the following:

```TypeScript
let counter: number;
counter = 1; // valid
counter = 'Hello'; // compilation error 
```

**For Arrays:**

```TypeScript
let arrayName: type[];
let names: string[] = ['John', 'Jane', 'Peter', 'David', 'Mary'];
// specify the array is one of strings 
```

**For objects:**

```TypeScript
let person: {
   name: string;
   age: number
};

person = {
   name: 'John',
   age: 25
}; // valid
```

This specifies the type of each parameter of the object.

  

### Function Arguments and & Return Types

We can declare functions that set up the return and input type beforehand:

```TypeScript
let greeting = (name: string) => string;
greeting = function(name: string) {
	return 'Hi ${name}';
};


// Something like the below would result in compilation error
greeting = function() {
	consolg.log('Hello');
};

// as it both does not match the input parameters, nor the return type
```

## Summary

- Use type annotations with the syntax `: [type]` to explicitly specify a type for a variable, function, function return value, etc.