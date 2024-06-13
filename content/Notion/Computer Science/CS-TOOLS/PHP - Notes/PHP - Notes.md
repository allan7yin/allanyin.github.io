I've built websites using `HTML`, but more and more, the better language to use is `PHP`. This does not mean no `HTML` will be used period, but its more of a integration between the two languages.

---

There's no need to take detailed notes on the syntax of `PHP` as such things are easily remembered by reading documentation. The core concepts of `PHP` I undertsand, and so I understand and am proficient with the language. Some of the main language core elements are:

- Comments
- Variables
- Echo / Print (two are the same thing)
- Data types
- Strings
- Numbers
- Math
- Constants
- Operators
- If...else...elseif
- Switch
- Loops
- Functions
- Arrays
- Super globals

In addition, `PHP`supports object-orientated programming, so I'm able to `new`objects from classes and call member certain functions.

The main things, however, I need to know, and will be documented in this notepad, are working with forms in `PHP` and working with `MySQL` and `MySQLi` through `PHP` built-in functions.

---

The below displays a simple `HTML` form with two input fields and submit button.

```Plain
<html>
<body>

<form action="welcome.php" method="post">
Name: <input type="text" name="name"><br>
E-mail: <input type="text" name="email"><br>
<input type="submit">
</form>

</body>
</html>
```