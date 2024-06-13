There is type inference in TypeScript. Consider the following:

```TypeScript
let counter: number; // declaring the type of counter is a number 

// suppose we did this
let counter = 0;
// this would be the same as doing:
let counter: number = 0; // the above has infered typing, just like auto in C++
```

Just as TypeScript can infer types of variables, it can do so for arguments and return values:

```TypeScript
function doSomething (max=100) {
	// default value for max is 100, so type of max is number 
}

function increment(counter: number) {
	return number++;
}

// the above is the same as writing:
function increment(counter: number): number {
	return number++;
}

// compiler able to auto-recognize the return type of a function 
```

  

### Arrays of Types

Consider:

```TypeScript
let items = [0,1,2,3,null];
```

TypeScript has a “best type” algorithm that is used when it encounters an array of multiple types (or single types). For instance, the array above would be of type `numbers[]`. Consider the following:

```TypeScript
let iems = [0,1,null,'Hi']
```