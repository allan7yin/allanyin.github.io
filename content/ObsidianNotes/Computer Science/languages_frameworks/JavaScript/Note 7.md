### HTML DOM - Changing CSS

The HTML DOM allows Javascript to change the style of HTML elements. We use the syntax:

```Plain
document.getElementById(id).style.property = new style
// note the use of new, hints that we are making an object
```

An example code of this is:

```Plain

<!DOCTYPE html>
<html>
<body>

<h2>JavaScript HTML DOM</h2>
<p>Changing the HTML style:</p>


<p id="p1">Hello World!</p>
<p id="p2">Hello World!</p>

<script>
document.getElementById("p2").style.color = "blue";
document.getElementById("p2").style.fontFamily = "Arial";
document.getElementById("p2").style.fontSize = "larger";
</script>


</body>
</html>
```

### Using Events

Here is an example code snippet of when the style of an HTML element changes when the user clicks a button:

```Plain
<!DOCTYPE html>
<html>
<body>

<h1 id="id1">My Heading 1</h1>

<button type="button"
onclick="document.getElementById('id1').style.color = 'red'">
Click Me!</button>

</body>
</html>

```

For a more complete list of all HTML DOM style properties, look at this link:  
  
[https://www.w3schools.com/jsref/dom_obj_style.asp](https://www.w3schools.com/jsref/dom_obj_style.asp)

---

### Javascript HTML DOM Animation

Now, lets learn how to make HTML animations with javascript. Consider the following code:

```Plain
<!DOCTYPE html>
<html>
<body>

<h1>My First JavaScript Animation</h1>

<div id="animation">My animation will go here</div>

</body>
</html>
```

We will build on top of this simple web page. First off, all animations should be relative to a container element. So, we can place a containing element around what we want to animate. For example, we can:

```Plain
<div id ="container">
  <div id ="animate">My animation will go here</div>
</div>
```

Next, we need to style both the container and the element inside. The container element should be created with style = "`position: relative`". The animation element should be created with style = "`position: absolute`". Like so:

```Plain
<!Doctype html>
<html>
<style>
\#container {
  width: 400px;
  height: 400px;
  position: relative;
  background: yellow;
}
\#animate {
  width: 50px;
  height: 50px;
  position: absolute;
  background: red;
}
</style>
<body>

<h2>My First JavaScript Animation</h2>

<div id="container">
<div id="animate"></div>
</div>

</body>
</html>
```

### Animation Code

We can make Javascript animations by making gradual changes in an element's style. These changes are called by a timer, when the time interval is small, the animation looks continuous. For instance:

```Plain
id = setInterval(frame, 5);

function frame() {
  if (/* test for finished */) {
    clearInterval(id);
  } else {
    /* code to change the element style */
  }
}
```

`setInterval(function, time)` calls a function (`frame` in the example above) every `time` milliseconds, so above, every 5 milliseconds. `setInterval` continues calling the function until `clearInterval` is read in. Now, let's write the entire code to animate:

```Plain
<!DOCTYPE html>
<html>
<style>
\#container {
  width: 400px;
  height: 400px;
  position: relative;
  background: yellow;
}
\#animate {
  width: 50px;
  height: 50px;
  position: absolute;
  background-color: red;
}
</style>
<body>

<p><button onclick="myMove()">Click Me</button></p>

<div id ="container">
  <div id ="animate"></div>
</div>

<script>
function myMove() {
  let id = null;
  const elem = document.getElementById("animate");
  let pos = 0;
  clearInterval(id);
  id = setInterval(frame, 5);
  function frame() {
    if (pos == 350) {
      clearInterval(id);
    } else {
      pos++;
      elem.style.top = pos + "px";
      elem.style.left = pos + "px";
    }
  }
}
</script>

</body>
</html>

```

---

### JavaScript HTML DOM Events

HTML DOM allows JavaScript to react to HTML events.

### Reacting to Events

A javaScript can be executed to react to events, like when a user clicks on an HTML element. To execute code when a user clicks on an element, add JavaScript code to an HTML event attribute:

```Plain
onclick=JavaScript
```

Examples of HTML events:

- When a user clicks the mouse
- When a web page has loaded
- When an image has been loaded
- When the mouse moves over an element
- When an input field is changed
- When an HTML form is submitted
- When a user strokes a key

Below are examples:

**Content changed when user clicks on it:**

```Plain
<!DOCTYPE html>
<html>
<body>

<h1 onclick="this.innerHTML = 'Ooops!'">Click on this text!</h1>

</body>
</html>
```

**Function called as event handler**

```Plain
<!DOCTYPE html>
<html>
<body>

<h1 onclick="changeText(this)">Click on this text!</h1>

<script>
function changeText(id) {
  id.innerHTML = "Ooops!";
}
</script>

</body>
</html>
```

---

### HTML Event Attributes

In order to assign events to HTML elements, we can use event attributes, such as `onclick`:

```Plain
<button onclick="displayDate()">Try it</button>
```

We can also assign these events to HTML elements via JavaScript:

```Plain
<script>
document.getElementById("myBtn").onclick = displayDate;
</script>
```

---

### The onload and onunload Events

The `onload` and `onunload` events are triggered when then user enters or leave the page. The `onload` event can be used to check the visitor's browser type and browser version, and load the proper version of the web page based on the information. We can use the `onload` and `onunload` events to load cookies.

Below is an example:

```Plain
<!DOCTYPE html>
<html>
<body onload="checkCookies()">

<h2>JavaScript HTML Events</h2>

<p id="demo"></p>

<script>
function checkCookies() {
  var text = "";
  if (navigator.cookieEnabled == true) { // navigator provides information on the web browser
    text = "Cookies are enabled.";
  } else {
    text = "Cookies are not enabled.";
  }
  document.getElementById("demo").innerHTML = text;
}
</script>

</body>
</html>
```

### The onchange event

The `onchange` event is used in combination with the validation of input fields. Once you leave something, say an in input field, a function is called. For instance, the below code capitalizes user input, once the user leaves the input field:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript HTML Events</h2>
Enter your name: <input type="text" id="fname" onchange="upperCase()">
<p>When you leave the input field, a function is triggered which transforms the input text to upper case.</p>

<script>
function upperCase() {
  const x = document.getElementById("fname");
  x.value = x.value.toUpperCase();
}
</script>

</body>
</html>

```

### onmouseover and onmouseout

The `onmouseover` and `onmouseout` events can be used to trigger a function when the user mouses over, or out of, an HTML element. Here is an example:

```Plain
<!DOCTYPE html>
<html>
<body>

<div onmouseover="mOver(this)" onmouseout="mOut(this)" onclick="mOn(this)" id="rect"
style="background-color:\#D94A38;width:120px;height:20px;padding:40px;">
Mouse Over Me</div>

<!-- recall that every DOM element is an object, so we can use this for scripting -->

<script>
function mOver(obj) {
  obj.innerHTML = "Thank You";
}

function mOut(obj) {
  obj.innerHTML = "Bye";
}

function mOn(obj) {
	obj.innerHTML = "Colour Change";
  // add some things below by myself to see how they would work
    if (document.getElementById("rect").style.color == "blue") {
    	document.getElementById("rect").style.color = "green";
    } else {
    	document.getElementById("rect").style.color = "blue";
    }
}

</script>

</body>
</html>

```

### The onmousedown, onmouseup, and onclick events

The above three events are all mouse click events. When you initially click something, `onmousedown` is registered, then, when mouse is release, `onmouseup` is registered. Finally, when the mouse click is complete, `onclick` is registered. Example:

```Plain
<!DOCTYPE html>
<html>
<body>

<div onmousedown="mDown(this)" onmouseup="mUp(this)"
style="background-color:\#D94A38;width:90px;height:20px;padding:40px;">
Click Me</div>

<script>
function mDown(obj) {
  obj.style.backgroundColor = "\#1ec5e5";
  obj.innerHTML = "Release Me";
}

function mUp(obj) {
  obj.style.backgroundColor="\#D94A38";
  obj.innerHTML="Thank You";
}
</script>

</body>
</html>
```