Now, let's introduce something that is extremely important to the language of Javascript: JavaScript Object Notation (JSON).

### JSON

- JSON is a format for storing and transporting data.
- JSON is often used when data is sent from a server to a web page.

Here is an example of a JSON:

```Plain
"employees":[
  {"firstName":"John", "lastName":"Doe"},
  {"firstName":"Anna", "lastName":"Smith"},
  {"firstName":"Peter", "lastName":"Jones"}
]
```

It is important to recognize that a JSON is not part of JS, rather, it is a format of data that we be read and intrepretedb by any language. It is text. So, to use the data of a JSON, we need to convert it into an object. JS has build in methods for doing so.

Say we have a JSON that is like this:

```Plain
// we'll use a string to demonstrate
let text = '{ "employees" : [' +
'{ "firstName":"John" , "lastName":"Doe" },' +
'{ "firstName":"Anna" , "lastName":"Smith" },' +
'{ "firstName":"Peter" , "lastName":"Jones" } ]}';

// to make this into a javascript object
const obj = JSON.parse(text);
// now, we have the JSON in the form of a javascript object and can use it
```

Now, lets take a step back from JSONS, and look at objects in java script.

- Booleans can be objects (if defined with the new keyword)
- Numbers can be objects (if defined with the new keyword)
- Strings can be objects (if defined with the new keyword)
- Dates are always objects
- Maths are always objects
- Regular expressions are always objects
- Arrays are always objects
- Functions are always objects
- Objects are always objects

In JS, everything that is not a primitive is an object. These primitive data types are:

- string
- number
- boolean
- null
- undefined
- symbol
- bigint