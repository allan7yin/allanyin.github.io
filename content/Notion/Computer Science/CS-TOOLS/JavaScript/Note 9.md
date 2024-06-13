# JavaScript HTML DOM Elements (Nodes)

We can add and remove nodes (HTML elements)

To add a new element to the HTML DOM, you must create the element first, and then append it to an existing element. Here is an example:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript HTML DOM</h2>
<p>Add a new HTML Element.</p>

<div id="div1">
<p id="p1">This is a paragraph.</p>
<p id="p2">This is another paragraph.</p>
</div>

<script>
const para = document.createElement("p");
const node = document.createTextNode("This is new.");
para.appendChild(node);
const element = document.getElementById("div1");
element.appendChild(para);
</script>

</body>
</html>
```

Let's take a moment to see whats going on inside this example code snippet. As seen, we first created a `<p>` node, and since the text inside of it is needed to be in a textNode, we then create a text node with the desired text inside. Then we append the text node as a child node to the paragraph node. Markup languages like HTML, are all trees.

If I wanted to then add, say an attribute, to the paragraph node, I would do something like this:

```Plain
const node = document.getElementById("div1");
const a = document.createAttribute("my_attrib");
a.value = "newVal";
node.setAttributeNode(a);
console.log(node.getAttribute("my_attrib")); // "newVal"
```

---

### Creating new HTML Elements - insertBefore()

Previously, the `appendChild()` method appends the new element as the last child of the parent. If we want the element to be appended as the first child, we can use the `insertBefore()` method.

### Removing Existing HTML Elements

To remove an HTML element, we use the `remove()` method:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript HTML DOM</h2>
<h3>Remove an HTML Element.</h3>

<div>
<p id="p1">This is a paragraph.</p>
<p id="p2">This is another paragraph.</p>
</div>

<button onclick="myFunction()">Remove Element</button>

<script>
function myFunction() {
document.getElementById("p1").remove();
}
</script>

</body>
</html>
```

- *NOTE: ** `remove()` does not work in older browsers, instead, on older versions, we would use `removeChild()`.

### Removing a child node

Example:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript HTML DOM</h2>
<p>Remove Child Element</p>

<div id="div1">
<p id="p1">This is a paragraph.</p>
<p id="p2">This is another paragraph.</p>
</div>

<script>
const parent = document.getElementById("div1");
const child = document.getElementById("p1");
parent.removeChild(child);
</script>

</body>
</html>
```

The `removeChild()` method is from the parent node object. If your unsure what the parent node is, we can simply get the parent node of the child node through a method of the child node:

```Plain
const child = docuemnt.getElementById("p1");
child.parentNode.removeChild(child);
// yes, we pass the entire object into it
```

### Replacing HTML elements

In order to replace an element in the HTML DOM, we use the `replaceChild()` method. Consider the following code:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript HTML DOM</h2>
<h3>Replace an HTML Element.</h3>

<div id="div1">
<p id="p1">This is a paragraph.</p>
<p id="p2">This is a paragraph.</p>
</div>

<script>
const parent = document.getElementById("div1");
const child = document.getElementById("p1");
const para = document.createElement("p");
const node = document.createTextNode("This is new.");
para.appendChild(node);
parent.replaceChild(para,child); // replaces the child node with the para node
</script>

</body>
</html>
```