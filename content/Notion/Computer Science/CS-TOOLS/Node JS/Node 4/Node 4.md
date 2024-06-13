### Section 7: Web Servers (Weather App)

Here, we take another step in the MERN Stack, and will explore **express**, a NPM library that makes it easy to create web servers with Node.

  

### Express Intro + Sending HTML & JSON data

```JavaScript
const express = require('express');
// express library exposes a function, so the above express is actually a function, and not we call it to make a express application

const app = express(); // express function has no parameters, so moving forward, we just now tell our express application what to do

// say that we owned the domain app.com, then it may have some other pages like:
// app.com/help
// app.com/about 

/*
 // thie lets us tell server what to do when someone tries to get a specific resource at a url
app.get('', (req, res) => {
    res.send('Hello express!');
});

// first paremetr is a route, second is a function that detaisl what we do when someone visits that route
// inside the function, we have two thingsm req - request, and res - response. 

// now, lets start the server, only do this once in entire application

app.listen(3000, () => {
    console.log('Server is up on the port 3000.');
});

// to set up another path, we just need to set up another app.get
// app.com/help
app.get('/help', (req,res) => {
    res.send('Help Pages');
});

// goal, as an excercise, lets set some new routes for: about, weather, 

app.get('/about', (req,res) => {
    red.send('About page');
});

app.get('/weather', (req,res) => {
    res.send('Weather page');
});

*/
// now lets look at how to serve up HTML and JSON 


app.listen(3000, () => {
    console.log('Server is up on the port 3000.');
});

app.get('', (req, res) => {
    res.send('Hello express!');
});

app.get('/help', (req,res) => { // we can also send back JSON 
    res.send({
        name: 'Allan',
        age: 19
    });
});

app.get('/about', (req,res) => {
    red.send('About page');
});

app.get('/weather', (req,res) => {
    res.send('<h1> The Weather </h1>'); // we can send back HTML code 
});
```

  

### Serving up Static Assets

Previously, we saw what it was like to respond with HTML and JSON. However, this is obviously not a good solution/idea. It can become quite disorganized and prone to error to send such raw data. Instead, we will look at how we can serve up static assets. It would be ideal if we could send something like an HTML file or something like that…

  

The solution to this is to configure Express to serve up an entire directory of assets that could contain HTML files, CSS files, client side JS videos, images, and more. To illustrate this, we create the directory called `public`, which will the directory we will serve up with express.

  

To illustrate this, here is the code that shows this:

```JavaScript
const publicDirectoryPath = path.join(__dirname, '../public');
app.use(express.static(publicDirectoryPath)); 
// we'll look more at it later, customizes the server
// express.static takes a path to what we want to serve
```

While, at the moment, we have not dived into what app.use does, this is the way we serve up static assets. So, in the public directory, we can retrieve files from there. Two important things to know:

```JavaScript
console.log(__dirname); // the directory name 
console.log(_filename); // the file name 
console.log(__dirname, ../..); // second argument manipulates path
// in the case above, I've gone up two directories 
```

So, essentially, we first generate the path to our public folder via `const publicDirectoryPath = path.join(__dirname, '../public');`, then, we provide it to static which, in some way, configures our express application (`app.use(express.static(publicDirectoryPath));`). Now, there is no longer a need to write:

```JavaScript
app.get('', (req, res) => {
	res.send('<h1> Weather </h1>')
})
```

So now, the route handler above will no longer run, and so, we don’t need it. Instead, that page is now coming from that static directory. We could do the same, like for About or help page too. However, remember these are static, so they are just html pages. So path would look like:

```JavaScript
localhost:3000/about.html
```

If we don’t want this, we need to have dynamic pages:

  

### Dynamic Pages with Templating

We use template engines to render dynamic web pages using express. To do this, we first needed a template engine. The template engine we will use for this course is handlerbars. So, download and install the npm package. To use handlebars, we need a folder called “views”, inside of the project folder. Inside here, we store files, such as `about.hbs`. Simply, we have out html code inside of the hbs file, but we can pass values to the file from the javascript file that is using express to display the page:

```JavaScript
// in app.js, in the js file using express 
app.set('view engine', 'hbs'); // this requires that we make a directory called "views" in the heart of the project, so web-server in this case

app.get('', (req, res) => {
    res.render('index', {
        title: 'MERN Stack Weather Application',
        name: 'Allan Yin'
    });
})

app.get('/about', (req, res) => {
    res.render('about', {
        title: 'About Me',
        name: 'Allan Yin'
    });
})

app.get('/help', (req, res) => {
    res.render('help', {
        message: 'This is the help page'
    })
})
```

Now, this will trigger dynamic pages. Now, we can delete our index, about, and help.html files, since we no longer need them. Now, in the those .hbs files, if we want to access our css and js files, which are in public, we do:

```Plain
<link rel="stylesheet" href="/css/styles.css">
```

Now, notice that we have the path beginning with a `/`. So, what is the difference between a `/` and `./`? The first one is an absolute path (hard drive for a computer), but in this case, it is the root folder for the web server, which would be public, which was setup by express:

```JavaScript
const publicDirectoryPath = path.join(__dirname, '../public');
// this sets it to public 
```

So, even if our `.hbs` folders are in view, they can still access the tings they need in public. So now, we have dynamic pages with templating. Again, it may be unclear here, so to see whole code, check out the learning NodeJS folder.

  

### Customizing the Views Directory

Now, what if we wanted to change the name of the directory from `views` to something like `templates`? If we try to do this without changing anything, an error will occur as express is set up to only look for a directory named `views`.

  

We can do this by having the following:

```JavaScript
const viewsPath =  path.join(__dirname, '../templates);
// we want to mke express not need views, but instead, need templates
app.set('views', viewsPath);
```

  

### Advanced Templating

  

Now, we look at creating partials with handle bars. Partials allows us to re-use certain elements. For example, suppose I want to keep the same header or footer over multiple pages, and I don’t want to copy paste markup from each file. We can use partials to solve this. The general idea of this is to create multiple “views” directories, and nest them.

  

First step in doing so, is to require the `hbs` module:

```JavaScript
const hbs = require('hbs');
```

  

![[Screen_Shot_2022-09-28_at_8.49.34_PM.png]]

This is what the directory looks like after proper creation of the files. So, we now need in our main javascript file:

```JavaScript
const partialsPath = path.join(__dirname, '../templates/partials');
hbs.registerPartials(partialsPath);
```

Next, we create what files we want as partials in the partials folder. Each `.hbs` file is not a full markup page, but just snippets of html we want to insert. The, in the `.hbs` files in views, if there is some component we want to resuse, we just:

```JavaScript
<body>
    <h1> {{title}} </h1>
    <p> Creted by {{name}} </p>

    {{>header}} <!-- Partial insert --> 

{{>footer}}
</body>
```

Above, header and footer are partials that have been included in this file. So, re-used.

  

### 404 Pages

Now, how do we set up a 404 page, this is displayed when we are not sure what to return, maybe invalid url. The way we do this is actually very simple. It goes like this:

```JavaScript
// TO SETUP A 404 PAGE (NEEDS TO COME LAST)

app.get('*', (req, res) => { // the * means matching anything 
    res.send('My 404 page');
})

// this needs to be last because of the way express is set up. When express gets a request, it looks for a matching route handler. With each of
// the app.get's above, express looks for a match, in sequential order. So, the *, if placed first, would always be used. We need it to be last, as a sort of 
// "last ressor"
```

  

### Brief CSS Styling Review

Check the project folder for this information.

  

We are nearing end of completion of the weather application. Spend some time making sure that the front end styles are good enough and then move on. This will be something that will go on your Github.