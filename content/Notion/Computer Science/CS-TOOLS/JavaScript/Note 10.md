# JavaScript HTML DOM Collections

### The HTMLCollection Object

The method `getElementsByTagName()` returns a `HTMLCollection` object. An `HTMLCollection` object is an array-like-list (collection) of HTML elements.

Consider the following code:

```Plain
const myCollection = document.getElementsByTagName("p");
// to access the second <p> element, we can
myCollection[1]
// we access the indexes of the array as we would with a conventional array
```

### HTML HTMLCollection length

```Plain
myCollection.length
```

It is important to note that while the collection may look like an array, it is not. While we can loop through it using index notation, array methods will not work on them.

---

### JavaScript HTML DOM Node Lists

- A `NodeList` object is a a list (collection) of nodes extracted from a document.
- A `NodeList` object is almost the same as an `HTMLCollection` object.
- Some(older) browsers return a `NodeList` object instead of an `HTMLCollection` for methods like `getElementsByClassName()`
- **All** browsers return a `NodeList` object for the property `childNodes`.
- Most browsers return a `NodeList` object for the method `querySelectAll()`

The following code selects all `<p>` nodes in a document:

```Plain
const myNodeList = document.querySelectorAll("p");
// to access the second p element, we can write
myNodeList[1]
```

Accessing the size of the `NodeList` is the same as before:

```Plain
myNodelist.length.
```

### `HTMLCollection` VS. `NodeList`

So, both of these are very similar. They are both array-like containers. They both have the `length` method and both cannot call normal array methods.