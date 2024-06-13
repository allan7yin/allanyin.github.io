  

NodeJs is something you install on your local computer. We can run local files through a command like:

```Bash
node someJavaScriptFile.js
```

  

This will run the file. Moving forward, we will often reference the NodeJS documentation for useful tools and things we may need.

  

### Importing Core NodeJS Modules

  

**Something interesting to note:** When we write `console.log(something)`, this is not JavaScript code. Rather, it is more related to the browser, and in this case, NodeJS.

  

Now, let’s say we want to make a file in our current directory with node, there is a node command:

```JavaScript
fs.writeFileSync(’fileName.txt’, ‘the contents of that file as a string’);
// however, fs is from the NodeJs module, so we need to include it:
// so, the file should look something like:

--------------------------------------------------------------------------------

const fs = require('fs');
fs.writeFileSync('fileName.txt', 'the contents of that file as a string');
fs.appendFileSync('fileName.txt', '. This message was appended afterwards'); 
// the second one appends something to the file
```

  

Above, inside the “require”, is the name of the Node Module.

  

### Importing Your Own Files

  

We can import files into a another file with:

```JavaScript
require('./utils.js'); // we need ./ to mean the path 

const name = 'Allan';

console.log(name);
```

  

In the above, we imported the file `utilis.js` into the current file. What if we want to define a variable in `utils.js`, and to use it in the current file? We can export things from a file, which is then defined as the return value in the current file:

```JavaScript
// in app.js

const name = require('./utils.js'); 
console.log(name);

// in utils.js
const name = 'Allan';
module.exports = name; // the return value when another file acquires this file
```

  

A more realistic export method, however, is to export functions:

```JavaScript
// in app.js

const add = require('./utils.js'); 
console.log(add(1,1));

// in utils.js
const add = function(a,b) {
	return a+b;
}

module.exports = add;
```

  

### Importing NMP Modules

  

How do we load in NPM packages? Installing node also gives us access to everything NPM has to offer, which we can go see on **www.npmjs.com.**

  

To begin, we can run `npm init`. What this does it creates a file containing a JSON which has some details that we wont need right now. NPM is very handy, as it essentailly contains modules/libraries of things that we may want to use. For example, say we wanted to preform some sort of email verification in our code. We would have 2 ways of doing so:

1. We can write the code ourselves, testing edge/boundary cases
2. Import a NPM module that is externally regulated and maintained

  

Of course, the second option is preferable, which is what we’ll work with moving forward. Consider the following code:

```JavaScript
const validator = require('validator');
// we of course, need to load in the npm module
```

Now, there are many different validators available, all of which can be seen in the documentation. For example, some of them include:

- `**isCurrency(str [, options])**`
- `**isCreditCard(str)**`
- `**isEmail(str [, options])**`
- `**isHexadecimal(str)**`

  

As seen, the NPM Module provides many validators that would be essential for many applications to have. For now, let’s focus on the email validation.

  

### **isEmail(str [, options])**

  

Now, the validator that we have imported into the file is an object, and the above validators are all methods of the validator object:

```JavaScript
const validator = require('validator');
const myEmail = 'a7yin@uwaterloo.ca';

if (validator.isEmail(myEmail)) {
	console.log('true');
} else {
	console .log('false');
}
```

  

Now let’s look at another module: Chalk. Chalk is a simple tool that can allow us to change the colour/background colour of text in the terminal. This may be useful if other developers run the application, and see errors in red, etc.

  

```JavaScript
const chalk = require('chalk'); // version 2.4.1, newer one does not work
console.log(chalk.green('Hello World'));

// this logs the message to the console in green 
```

  

### Global NPM Modules and Nodemon

  

So far, we’ve used `require`, which is similar to using `\#include “some c++ module”` in C++. The module files get loaded in the current file. We can also install global NPM modules.

```Bash
npm i Nodemon@1.18.5 -g 
# the -g indicated global
```

  

Installing globally installs the tool on the local computer, on the OS, so you will not see in the directory of the current project.

  

Nodemon is just a little tool that re-runs the script, by itself, calling node, whenever the file is re-saved. This can be used to save time as we don’t need to re-type commands in the terminal to run the script.

```Bash
nodemon app.js
```