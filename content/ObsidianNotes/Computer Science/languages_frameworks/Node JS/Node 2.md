## File System and Command Line Args (Notes App)

  

Just like in bash, we can pass CLI to a node application. NodeJS has `process` , something that allows us to access CLI. Just like how the browser JS supports `document` Node has its own. Consider the following:

```Bash
node app.js Allan
```

Above, we we to run `console.log(process.argv);`, we would get:

```Bash
[
  '/usr/local/bin/node',
  '/Users/allanyin/CS Tools/Node-course/notes-app/app.js',
  'allan'
]
```

  

As seen above, the first two are paths to node and the file we are running, and lastly, the CLI is shown. Of course, to access one of the command line arguments, we simply pass an index to the `argv` vector.

  

### Argument Parsing with `Yargs`

  

Yargs is a module we can install from NPM, and it simply provides a more clean UI experience and introduces some new functionality. For instance, when we do the following:

```Bash
node app.js --help

# the --help will bring up a section that looks something like:

Options:
  --help     Show help                                                 [boolean]
  --version  Show version number                                       [boolean]
```

A nice utility of `Yargs` is that we can add commands:

```JavaScript
yargs.command({
	command: 'add', // the command we are adding
	describe: 'Add a new note', // describes the purpose of the command 
	handler: function() {
		console.log('Adding a new note');
	}
})
```

  

So, we can add the command line argument `add`, and through `Yargs`, it will execute the handler for that command. But, realistically, if we are to add a note through the add CLI, we would need to know more about the title, perhaps something like “title”, etc. So, we introduce the `builder` property, which is an object:

```JavaScript
yargs.command({
	command: 'add', // the command we are adding
	describe: 'Add a new note', // describes the purpose of the command
	builder: {
        title: {
            describe: 'Note title',
            demandOption: true, 
						// means that this tag's information is mandatory.
						type: 'string', // requires that what comes after title is a string 
        }
    },
	handler: function() {
		console.log('Adding a new note');
	}
})
```

  

### Storing Data with JSON

  

Now, prior to this, we’ve done some work that will help us build towards a “notes app”. Before that, we should look at where such data could be stored.

  

```JavaScript
const fs = require('fs');
const book = {
    title: 'Ego is the enemy',
    author: 'Ryan Holiday'
}

const bookJSON = JSON.stringify(book); // contrcts a JSON from an object 
console.log(bookJSON); // bookJSON is not a JS object, it is a JSON. so, something like bookJSON.title will not work 

const parsedData = JSON.parse(bookJSON);
console.log(parsedData.author);

// to save the JSON to a JSON file, we can use the fs module 
fs.writeFileSync('1-json.json', bookJSON);

// now that we've saved the JSON into the file, let's try reading the JSON in from the file 
const dataBuffer = fs.readFileSync('1-json.json');
// console.log(dataBuffer); however, this has one problem. This returns a buffer, something that looks like:
// <Buffer 7b 22 74 69 74 6c 65 22 3a 22 45 67 6f 20 69 73 20 74 68 65 20 65 6e 65 6d 79 22 2c 22 61 75 74 68 6f 72 22 3a 22 52 79 61 6e 20 48 6f 6c 69 64 61 79 ... 2 more bytes>
// we want a string of the object, so, we do:
console.log(dataBuffer.toString());
const data = JSON.parse(dataBuffer.toString()); // this is now a parsed JSON we have 
console.log(data.title);
```

  

The above is the general way you will handle JSON’s, reading them in, parsing them, turning them into strings, etc.

  

### Adding a Note

Here is the logic code for adding a note:

```JavaScript
const fs = require('fs');

const getNotes = function() {
    return 'Your notes...';
}

const addNote = function(title,body) {

    // if we wanted to, we would write some code to prevent duplicate notes from being added, simple loop
    const notes = loadNotes(); // notes are now loaded in, we append new note into the object 
    notes.push({
        title: title,
        body: body
    }) // pushing a new note onto the existing array 
    saveNotes(notes);

}

const saveNotes = function(notes) {
    fs.writeFileSync('notes.json', JSON.stringify(notes));
}

const loadNotes = function() {
    // we need to be able to know if the file notes.json exists or not, and what to do in the case it does not
    try {
        const dataBuffer = fs.readFileSync('notes.json');
        const dataJSON = dataBuffer.toString();
        return JSON.parse(dataJSON);
    } catch (e) {
        return []; // we know no data, we return an empty array 
    }

}

module.exports = {
    getNotes: getNotes,
    addNote: addNote
}
```

  

The rest of the command line argument functions are finished in the folder `notes-app`. Reference that directory for information. This wraps up the main things we need to know about JSON manipulation in Node.