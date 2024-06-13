  

### Section 8: Accessing API from Browser (Weather App)

  

Now, we want to be able to build an application where the user can enter something in the browser, and we retrieve the data for that input.

  

### The Query String

As you know, we can pass query strings into browser addresses, such as:

```JavaScript
http://localhost:3000/products?key=value&&name=allan
```

The key value pairs after the ? represent the query string. It is always a key value pair. This query can be used by express, such as the following:

```JavaScript
app.get('/products', (req,res) => {
    console.log(req.query); // req.query is the query in he address bar, this is stored into this string 
    res.send({
        products: []
    });
});
```

  

Now, we say we have built a weather page. Naturally, we would need the address of some place to fetch the weather. So, we can build these requirements into the code:

```JavaScript
app.get('/weather', (req,res) => {
    if (!(req.query.address)) {
        res.send({
            error: 'You must provide an address!'
        })
    } else {
        res.send({
            forecast: 'It is currently sunny',
            location: 'Oakville',
            address: req.query.address
        })
    }
});
```

  

### Building a JSON HTTP Endpoint

Now, we can connect the above app.get with HTTP requests made in the earlier module. Effectively, we can now take create a something, where user enters a form, which enters the needed address into the query, and now, we can access API and retrieve the data and send it back to the client.

  

---

(**We take a short break and introduce a new JavaScript feature, default function parameters)**

  

Really, its just the same thing as what we would expect from C++.

```JavaScript
const greeter = (name = 'user', age) => {
    console.log('Hello ' + name);
}

greeter('Andrew');
greeter();

// greeter is the staple example of what it looks like to have default parameters in code
// even more than that, we can desttructure objects as default parameters 

const transaction = (type, {label, stock = 0 } = {}) => {
    console.log(type, label, stock);
}

transaction('order'); // this logs in the console: order undefined 0
// we see order, as that is the type that is passed in
```

As seen above, we can even define default values for an object.

  

---

  

### Browser HTTP Requests with Fetch

Fetch is something we would use in our client-side javascript files. This file is not interacting on Nodejs. So, fetch is not something files that will be run on the Node server can understand. This is used in the JavaScript file that will be used as a source for the html page. if you recall, you probably have seen this being used in some React tutorials. The syntax would be like so:

```JavaScript
fetch('http://puzzle/mead/io/puzzle').then((response) => {
	response.json() .then((data) => {
		console.log(data)
	})
})
```

  

Now, with this, we can finally have a form on the web page, have it submitted, and have its contents used to fetch data, by using the express request methods we have defined, and then finally accessing the API data, returning it, and rendering it to the screen. The above “response” is precisely what we .send from the app.get. Here is the full client-side JavaScript code:

```JavaScript
const weatherForm = document.querySelector('form');
    // we use dom elements to do this
    // now, we need to add an event listeners to this element 
const search = document.querySelector('input');
const messageOne = document.getElementById('message-1');
const messageTwo = document.getElementById('message-2');



weatherForm.addEventListener('submit', (e) => {
    e.preventDefault(); // prevents default behaviour, which is refreshing the brower

    const location = search.value;

    messageOne.textContent = 'Loading...';
    messageTwo.textContent = '';


    console.log(location);
    // this is how to use fetch and fetching from a url. Now, we can render to the screen
    fetch('http://localhost:3000/weather?address=' + location).then((response) => {
        response.json().then((data) => {
          if (data.error) {
              console.log(data.error);
             messageOne.textContent = data.error;
          } else {
              messageOne.textContent = data.location;
              messageTwo.textContent = data.forecast;
            }
        })
    })
})
```