# JS HTML DOM

Now, this is the most important part of javascript, its use in DOM and uses in web development.

- *DOM ** stands for document object model, and is a structure that html takes on. It is a node and branch structure, the diagram is posted below:

Consider this html code:

```Plain
<html>
    <head>
        <title>JavaScript DOM</title>
    </head>
    <body>
        <p>Hello DOM!</p>
    </body>
</html>
```

This tree represents the code snippet above:

In other words, the HTML DOM model is a tree of objects. With this model, javascript has the power to create dynamic html.

---

### HTML DOM Methods

- The HTML DOM Methods are **actions** you can perform on HTML elements
- The HTML DOM properties are values (of HTML elements) that you can set or change
- In the DOM, all HTML elements are defined as objects.
- A property is a value that you can get or set (like changing the content of an HTML element).
- A method is an action you can do (like add or deleting an HTML element).

For example, consider the following code snippet:

```Plain
<html>
<body>

<p id="demo"></p>

<script>
document.getElementById("demo").innerHTML = "Hello World!";
</script>

</body>
</html>
```

Above, `getElementById() is the` is a method and `innerHTML` is a property.

### Finding HTML Elements

```Plain
// method
document.getElementById() // finds an element by ID
document.getElementsByTagName(name) // find elements by tag name
document.getElementsByClassName(name) // finds elements by class name
```

### Changing HTML Elements

```Plain
// property
element.innerHTML =  new html content	// Change the inner HTML of an element
element.attribute = new value	// Change the attribute value of an HTML element
element.style.property = new style	// Change the style of an HTML element

// method
element.setAttribute(attribute,value) // change the attribute value of an HTML element
```

### Adding and deleting elements

```Plain
document.createElement(element) // create an HTML element
document.removeChild(element) // remove an HTML element
document.appendChild(element) // add an HTML element
document.replaceChidld(new,old) // replace an HTML element
docuemnt.write(text) // write into the HTML output stream
```

### Adding event handlers

```Plain
doucment.getElementById(id).onclick = function() {code;}
// adding event handler code to an onclick event
```

### Finding HTML Objects

There are many different ways we can find select HTML objects. Read over them, below is a link of the list on W3Schools:  
  
[https://www.w3schools.com/js/js_htmldom_document.asp](https://www.w3schools.com/js/js_htmldom_document.asp)

---

# DOM Elements

Now, lets look at how to find and access HTML elements. In JS, I will often want to manipulate HTML elements. So, to do so, I need to be able to find the elements I want to work with. There are several ways to do this:

- Finding HTML elements by id
- Finding HTML elements by tag name
- Finding HTML elements by class name
- Finding HTML elements by CSS selectors
- Finding HTML elements by HTML object collections

Consider the code below:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript HTML DOM</h2>

<p id="intro">Finding HTML Elements by Id</p>
<p>This example demonstrates the <b>getElementsById</b> method.</p>

<p id="demo"></p>

<script>
const element = document.getElementById("intro");

document.getElementById("demo").innerHTML =
"The text from the intro paragraph is: " + element.innerHTML;

</script>

</body>
</html>
```

Here is an example of finding an HTML element by ID. If an element is not found, element will contain `null`.

---

### DOM HTML

=> **Changing HTML Content**

The HTML DOM allows Javascript to change the content of HTML elements. The easiest way to modify the content of an HTML element is by using the innerHTML property. To change the content of an HTML element, use this syntax.

```Plain
document.getElementById(id).innerHTML = new HTML
```

=> **Changing the value of an attribute**

To do this, we use the syntax:

```Plain
document.getElementById(id).attribute = new value
```

For example, consider the code below:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript HTML DOM</h2>
<img id="image" src="smiley.gif" width="160" height="120">

<script>
document.getElementById("image").src = "landscape.jpg";
</script>

<p>The original image was smiley.gif, but the script changed it to landscape.jpg</p>

</body>
</html>
```

In the above code, the original photo was `smiley.gif`, but the script changed it to `landscape.jpg`. So, now the document shows the second picture instead of the first one.

**NOTE:** The above code as `.src` since we are changing the `src` attribute. If we're changing a different attribute, we would need to make sure to use that attribute.

=> **document.write()**

This `document.wrirte()` can be used to write directly to the HTML output stream.

Look at the code below:

```Plain
<!DOCTYPE html>
<html>
<body>

<p>Bla, bla, bla</p>

<script>
document.write(Date());
</script>

<p>Bla, bla, bla</p>

</body>
</html>
```

The above code, when it reaches `document.write(Date());` calls the buil-in function `Date()` and prints the the return of that function the the document. Again, don't use this after a document has loaded, for example as the response to an action, as it will wipe the rest of the html.

---

### DOM Forms

HTML form validation can be done by Javascript. If a form filed (fname) is empty, this function alerts a message, and returns false. Consider the following:

```Plain
function validateForm() {
  let x = document.forms["myForm"]["fname"].value;
  if (x == "") {
    alert("Name must be filled out");
    return false;
  }
}
```

The above form is used below:

```Plain
<!DOCTYPE html>
<html>
<head>
<script>
function validateForm() {
  let x = document.forms["myForm"]["fname"].value;
  if (x == "") {
    alert("Name must be filled out");
    return false;
  }
}
</script>
</head>
<body>

<h2>JavaScript Validation</h2>

<form name="myForm" action="/action_page.php" onsubmit="return validateForm()" method="post">
  Name: <input type="text" name="fname">
  <input type="submit" value="Submit">
</form>

</body>
</html>
```

### Javascript can validate numeric input

The below code is a way to ensure a form input is valid integer from 1 to 10.

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript Validation</h2>

<p>Please input a number between 1 and 10:</p>

<input id="numb">

<button type="button" onclick="myFunction()">Submit</button>

<p id="demo"></p>

<script>
function myFunction() {
  // Get the value of the input field with id="numb"
  let x = document.getElementById("numb").value;
  // If x is Not a Number or less than one or greater than 10
  let text;
  if (isNaN(x) || x < 1 || x > 10) {
    text = "Input not valid";
  } else {
    text = "Input OK";
  }
  document.getElementById("demo").innerHTML = text;
}
</script>

</body>
</html>
```

We can also make this type of validation automatic. HTML form validation can be performed automatically by the browser. If a form field (fname) is empty, the `required` attribute prevenets the form from being submitted.

An example code snippet of such an action is as follows:

```Plain
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript Validation</h2>

<form action="/action_page.php" method="post">
  <input type="text" name="fname" required>
  <input type="submit" value="Submit">
</form>

<p>If you click submit, without filling out the text field,
your browser will display an error message.</p>

</body>
</html>

```

keep in mind that the `name` attribute is quite literally a "name" for the element. The `name` attribute can be used as a reference when the data is submitted. The `type` attribute is used for different for different HTML elements. For example:

- For `<button>`elements, the type specifies the type of button being used
    - e.g. `reset`, `submuit`, etc.
- For `<input>`elements, the types specifies type of input

Reference documentation when such information is needed.

Above, we have the action attribute and it specifies the form-handler that will process the submitted form data. Form data is mostly submitted to a server-side handler, but it can also be JavaScript on the client. Above is an example of form data being sent to server-side hanlder. We can also do this on the client via js:

```Plain

<form action="javascript:submit()">
  <label>Enter your address</label> <br />
  <input type="text" name="street" placeholder="Street"><br />
  <input type="text" name="city" placeholder="City"><br />
  <input type="text" name="state" placeholder="State"><br /><br />

  <button type="submit">Submit</button>
</form>

<script>

  let submit = () => {
    let values = "";
    values += "Street = " + document.getElementsByName("street")[0].value + "\\n";
    values += "City = " + document.getElementsByName("city")[0].value + "\\n";
    values += "State = " + document.getElementsByName("state")[0].value + "\\n";
    alert(values);
  };

</script>
```

---

### Data Validation

Data validation is the process of ensuring that user input is clean, correct, and useful. Most often, the purpose of data validation is to ensure correct user input.

- Server side validation is performed by a web server, after input has been sent to the server.
- Client side validation is performed by a web browser, before input is sent to a web server.