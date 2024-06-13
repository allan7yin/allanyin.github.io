```HTML
<!DOCTYPE html>
<html>
<head>
<script>
function myFunction() {
  document.getElementById("demo").innerHTML = "Paragraph changed.";
}
// you can also put this java script in an external file, so you would write
// <script src="file.js">
// or <script src="<https://www.w3schools.com/js/myScript.js>">         a full URL address
// or <script src="/js/myScript.js">          a file path

// the function would then be in the external javascript file
</script>
</head>
<body>
<h2>Demo JavaScript in Head</h2>

<p id="demo">A Paragraph</p>
<button type="button" onclick="myFunction()">Try it</button>

</body>
</html>
```

**Some additional notes**:

- `.innerHTML` property is the content of the HTML element, changing this is a common way to display data in HTML
- `document.write()` writes to an HTML output, if used after an HTML document has loaded, will delete all existing HTML, e.g

```HTML
<!DOCTYPE html>
<html>
<body>

<h2>My First Web Page</h2>
<p>My first paragraph.</p>

<button type="button" onclick="document.write(5 + 6)">Try it</button>

</body>
</html>
```

Here, after the HTML document is loaded, the onclick will replace all existing HTML with 11.

- `window.alert()` writes into an alert box.

```HTML
<!DOCTYPE html>
<html>
<body>

<h2>My First Web Page</h2>
<p>My first paragraph.</p>

<script>
window.alert(5 + 6);
// wrtiing window is optional, also works with just alert
</script>

</body>
</html>
```

When the HTML document is loaded, a pop up window appears with the alert written on it, so for the above, 11.

- last is `console.log`, used for debugging, outputs to browser console
- *NOTE: ** JS does not have any print methods

---

JS types are dynamic:

```JavaScript
let x; // undefiend
x = 5;
x = "allan";
```

JS Objects are written as name value pairs:

```HTML
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript Objects</h2>

<p id="demo"></p>

<script>
const person = {
  firstName: "John",
  lastName : "Doe",
  id       : 5566,
  fullName : function() {
    return this.firstName + " " + this.lastName;
  }
};

// diffeent then c++, this is the object, not a pointer to it

document.getElementById("demo").innerHTML =
person.firstName + " is " + person.age + " years old.";
</script>

</body>
</html>
```

In JS, it is common to declare objects with `const` in front

---

In simple terms, HTML events are "things" that happen to HTML elements. When JS is used in HTML pages, JS can "react" on these events,

Often, when such HTML events are detected, we want to perform some sort of action on it. JS lets us execute code when events are detected.

HTML allows for event handler attributes, with JS code:

```HTML
<element event='some JavaScript'>
<element event="some JavaScript">
```

An example is `onclick` attribute (with code) is added to a `<button>` element:

```HTML
<!DOCTYPE html>
<html>
<body>

<button onclick="document.getElementById('demo').innerHTML=Date()">The time is?</button>

<p id="demo"></p>

</body>
</html>
```

There are countless events in HTML, some of the common ones are:

- `onchange` - HTML element as been changed
- `onclick`
- `onmouseover` - The user moves mouse over an HTML element
- `onmouseout` - The user moves mouse away from an HTML element
- `onkeydown` - The user pushes a keyboard key
- `onload` - The browser has finished loading the page

These are only a couple, and therea re actually many. Here's a link to more: [https://www.w3schools.com/jsref/dom_obj_event.asp](https://www.w3schools.com/jsref/dom_obj_event.asp)

We'll look more into these event handlers when we go over **HTML DOM** chapters

There are 3 methods for extracting a part of a string:

- `slice(start, end)`
- `substring(start, end)`
- `substr(start, length)`

We can use ```replace()`` method to replace a specified value with another value in a string

```JavaScript
let text = "Please visit Microsoft!";
let newText = text.replace("Microsoft", "W3Schools");
```

- The replace() method does not change the string it is called on.
- The replace() method returns a new string.
- The replace() method replaces only the first match
- `let newText = text.replace(/Microsoft/g, "W3Schools");` replaces all instances
- `let newText = text.replace(/Microsoft/i, "W3Schools");` replaces case insensitive

Theres a lot of things you can do with strings, here is the full reference from W3Schools: [https://www.w3schools.com/jsref/jsref_obj_string.asp](https://www.w3schools.com/jsref/jsref_obj_string.asp)

---

The `indexOf()` method returns the index of (the position of) the first occurrence of a specified text in a string:

```JavaScript
let str = "Please locate where 'locate' occurs!";
str.indexOf("locate");
// returns 7
```

The `lastIndexOf()` method returns the index of the last occurrence of a specified text in a string:

```JavaScript
let str = "Please locate where 'locate' occurs!";
str.lastIndexOf("locate");
// retuerns 21
```

Both `indexOf()`, and `lastIndexOf()` return -1 if the text is not found.

Can also pass a second parameter, which is the index to start the search from. `indexOf()` starts from parameter to end, whereas `lastIndexOx()` goes from parameter to the beginning.

To make a string with `` ` `` or `"` in it, we write strins with back-tics, known as template literals:

```JavaScript
let text = `He's often called "Johnny"`;
let text =
`The quick
brown fox
jumps over
the lazy dog`;

// can be mltiline strings like above, here, shares some similarities
// with bash scripting syntax

let firstName = "John";
let lastName = "Doe";

let text = `Welcome ${firstName}, ${lastName}!`;
// allows for variable substitution

let price = 10;
let VAT = 0.25;

let total = `Total: ${(price * (1 + VAT)).toFixed(2)}`;
// allows for expressions in strings, interpolation, can think of like
// a subshell
```

**HTML Templates**:

```HTML
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript Template Literals</h2>

<p>Template literals allows variables in strings:</p>

<p id="demo"></p>

<p>Template literals are not supported in Internet Explorer.</p>

<script>
let header = "Templates Literals";
let tags = ["template literals", "javascript", "es6"];

let html = `<h2>${header}</h2><ul>`;

for (const x of tags) {
  html += `<li>${x}</li>`;
}

html += `</ul>`;
document.getElementById("demo").innerHTML = html;
</script>

</body>
</html>
```

---