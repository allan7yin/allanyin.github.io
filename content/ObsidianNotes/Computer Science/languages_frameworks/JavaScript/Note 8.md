# JavaScript HTML DOM EventListner

Event Listners are a core concept.

### The `addEventListner()` method

We can create an event listener that fires when a user clicks a button:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript addEventListener()</h2>

<p>This example uses the addEventListener() method to attach a click event to a button.</p>

<button id="myBtn">Try it</button>

<p id="demo"></p>

<script>
document.getElementById("myBtn").addEventListener("click", displayDate);

function displayDate() {
  document.getElementById("demo").innerHTML = Date();
}
</script>

</body>
</html>
```

- Above, the `addEventListner()` method attaches an event handler to the specified element. The `addEventListener()` method attaches an event handler to an element without overwriting existing event handlers.
- So, we can have an event that has many event handlers, which makes sense. We can add many event handlers of the same type to one element, i.e. 2 "click" events. We like to use `addEventListner` since the JavaScript is separated from the HTML markup, ensuring better readability and maintainability
- You can easily remove an event listener by using the `removeEventListener()` method.

The general syntax for this is:

```Plain
element.addEventListener(event, function, useCapture);
```

The first parameter is the type of the event (like "click" or "mousedown" or any other HTML DOM Event.) Here is a link for a complete list of. HTML DOM Events: [https://www.w3schools.com/jsref/dom_obj_event.asp](https://www.w3schools.com/jsref/dom_obj_event.asp)

The second parameter is the handler function we call when the event occurs.

The third parameter is a boolean value specifying whether to use event bubbling or event capturing. This parameter is optional.

Consider the code below:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript addEventListener()</h2>

<p>This example uses the addEventListener() method to attach a click event to a button.</p>

<button id="myBtn">Try it</button>

<script>
document.getElementById("myBtn").addEventListener("click", function() {
  alert("Hello Allan!");
});
</script>

</body>
</html>
```

When the button is clicked by the user, we've added an event listener for it, and it executes an alert.

We could also write the function definition outside of the `addEventListener()`:

```Plain
<script>
document.getElementById("myBtn").addEventListener("click", myFunction);

function myFunction() {
  alert ("Hello World!");
}
</script>
```

In my opinion, the second one is more preferable.

### Add many event handlers to the same element

Something like:

```Plain
element.addEventListener("click", myFunction);
element.addEventListener("click", mySecondFunction);
```

is allowed, as well as this:

```Plain
element.addEventListener("mouseover", myFunction);
element.addEventListener("click", mySecondFunction);
element.addEventListener("mouseout", myThirdFunction);
```

The `addEventListener()` method allows you to add event listeners on any HTML DOM object such as HTML elements, the HTML document, the window object, or other objects that support events, like the xmlHttpRequest object.

Consider this code:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript addEventListener()</h2>

<p>This example uses the addEventListener() method on the window object.</p>

<p>Try resizing this browser window to trigger the "resize" event handler.</p>

<p id="demo"></p>

<script>
window.addEventListener("resize", function(){
  document.getElementById("demo").innerHTML = Math.random();
});
</script>

</body>
</html>
```

This prints a number to the document, and it continually changes into something random every time the window is resized.

### Parsing Parameters

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript addEventListener()</h2>

<p>This example demonstrates how to pass parameter values when using the addEventListener() method.</p>

<p>Click the button to perform a calculation.</p>

<button id="myBtn">Try it</button>

<p id="demo"></p>

<script>
let p1 = 5;
let p2 = 7;
document.getElementById("myBtn").addEventListener("click", function() {
  myFunction(p1, p2);
});

function myFunction(a, b) {
  document.getElementById("demo").innerHTML = a * b;
}
</script>

</body>
</html>
```

Honestly, not much to do for parsing, straightforward.

### Event Bubbling vs Event Capturing

There are two ways of event propagation in the HTML DOM, bubbling and capturing. Event propagation is a way of defining the order of event response. If you have a `<p>` element inside a `<div>` element, and the user clicks on the `<p>` element, which element's "click" event should be handled first?

In bubbling the inner most element's event is handled first and then the outer: the `<p>` element's click event is handled first, then the `<div>` element's click event.

In capturing the outer most element's event is handled first and then the inner: the `<div>` element's click event will be handled first, then the `<p>` element's click event.

With `addEventListner()`, we can specify the propagation type, by using the parameter `useCapture`. The default value of this is set to false, which will use the bubbling propagation. When the value is set to true, the event used the capturing propagation.

### The `removeEventListener()` method

The `removeEventListener()` method removes event handlers that have been attached with the `addEventListener()` method:

```Plain
<!DOCTYPE html>
<html>
<head>
<style>
\#myDIV {
  background-color: coral;
  border: 1px solid;
  padding: 50px;
  color: white;
  font-size: 20px;
}
</style>
</head>
<body>

<h2>JavaScript removeEventListener()</h2>

<div id="myDIV">
  <p>This div element has an onmousemove event handler that displays a random number every time you move your mouse inside this orange field.</p>
  <p>Click the button to remove the div's event handler.</p>
  <button onclick="removeHandler()" id="myBtn">Remove</button>
</div>

<p id="demo"></p>

<script>
document.getElementById("myDIV").addEventListener("mousemove", myFunction);

function myFunction() {
  document.getElementById("demo").innerHTML = Math.random();
}

function removeHandler() {
  document.getElementById("myDIV").removeEventListener("mousemove", myFunction);
}
</script>

</body>
</html>
```

---

### JavaScript HTML DOM Navigation

With the HTML DOM, you can navigate the node tree using node relationships

**DOM NODES**

Again, picture the DOM tree, where each DOM element can be considered a node in a tree. Everything in an HTML document is a node:

- The entire document is a document node
- Every HTML element is an element node
- The text inside HTML elements are text nodes
- Every HTML attribute is an attribute node (deprecated)
- All comments are comment nodes

The relationships between the nodes (parent, child, sibling) are straightforward. Here are some node properties to navigate between nodes in JavaScript:

- `parentNode`
- `childNodes[nodenumber]`
- `firstChild`
- `lastChild`
- `nextSibling`
- `previousSibling`

### Child Nodes and Node Values

A **common** error in DOM processing is to expect an element node to contain text

For example, consider this:

```Plain
<title id="demo">DOM Tutorial</title>
```

The title element/node, does **NOT** contain text. It contains a **text node** with the value "DOM tutorial". The value of the text node can be accessed by the node's `innerHTML` property:

```Plain
myTitle = document.getElementById("demo").innerHTML;
```

Accessing the innerHTML property is the same as accessing the `nodeValue` of the first child.

```Plain
myTitle = document.getElementById("demo").firstChild.nodeValue;
```

Also, as we have an array of child nodes, we can access it this way:

```Plain
myTitle = document.getElementById("demo").childNodes[0].nodeValue;
```

### DOM Root Nodes

There are 2 special properties that allow access to the full document:

- `document.body` - The body of the element
- `document.documentElement` - The full document

---

### The nodeName Property

The `nodeName` property specifies the name of a node:

- `nodeName` is read-only
- `nodeName` of an element node is the same as the tag name
- `nodeName` of an attribute node is the attribute name
- `nodeName` of a text node is always \#text
- `nodeName` of a document node is always \#document

Consider the following example:

```Plain
<!DOCTYPE html>
<html>
<body>

<h1 id="id01">My First Page</h1>
<p id="id02"></p>

<script>
document.getElementById("id02").innerHTML = document.getElementById("id01").nodeName;
</script>

</body>
</html>
```

=> Note: the node name always contains the uppercase tag name of an HTML element, if its something like `div`, then the `nodeName` will be `DIV`, in all capitals.

---

### The `nodeValue` property

This specifies the value of a node:

- `nodeValue` for element nodes is `null`
- `nodeValue` for text nodes is the text itself
- `nodeValue` for attribute nodes is the attribute value (lol)

---

### The `nodeType` property

Like above, the `nodeType` property is read only. It returns the type of a node. Consider the following:

```Plain
<!DOCTYPE html>
<html>
<body>

<h1 id="id01">My First Page</h1>
<p id="id02"></p>

<script>
document.getElementById("id02").innerHTML = document.getElementById("id01").nodeType;
</script>

</body>
</html>
```

While you may expect that the above would have the node type of a the

```Plain
<h1>
```

header to have type "h1", it does not. It has a type of

```Plain
1
```

. Different elements have different integer node types:

These don't seem particularly useful right now, but in the future they will be.