## REST API’s and Mongoose

Now, we are able to perform CRUD operations on our database. But, this is still far from where we want to be. How can we set which fields are required? How can we set type validation for each field? How can we ensure that only a certain user is able to change their own task? To address these needs, we use the **Mongoose** library.

### Mongoose

Mongoose is an ODM, so it’s similar to what Sequelize is. They map our doe objects into models that can be inserted into the database. ODM is object-document mapping, whereas ORM is object-relational mapping. They just differ since MongoDB is a document based database, where as something like MicrosoftSQL is a relational database.

**⇒ Model**

So, this is something we’ve seen in the BOR api code, and is something we should know how to work with. Here is how we would create a model with Mongoose:

```JavaScript
const Task = mongoose.model('Task', {
    description: {
        type: String
    }, 
    completed: {
        type: Boolean
    }
});
```

This effectively creates a table called “tasks”, in which to make entries into the database, we do:

```JavaScript
const task = new Task({
    description: 'Learn Mongoose library',
    completed: false
})

task.save().then((result) => {
    console.log(result)
}).catch((error) => {
    console.log(error)
})
```

To add it to the database, we just use the `save` method, which returns a `promise`, which we can then handle.

## Data Validation and Sanitization

To make things mandatory, we can add an option called `required`:

```JavaScript
const User = mongoose.model('User', {
    name: {
        type: String,
        required: true
    }, 
    age: {
        type: Number
    }
});
```

So, if we create a User object which does not have a name, it will throw an error. On the contrary, however, not including the age is fine, as it is not required. Now, there are a lot of other things we can use to validate the parameters we are using to create our objects. On important thing is being able to make custom validation:

```JavaScript
const User = mongoose.model('User', {
    name: {
        type: String,
        required: true
    }, 
    age: {
        type: Number,
        validate(value) {
            if (value < 0) {
                throw new Error('Age cannot be less than 0')
            }
        }
    }
});
```

Now, with the above code, we ensure that whenever a User entry is made, the age parameter is valid. The `value` parameter is what we are trying to validate (so “age” in the example above). Now, when it comes to validating other things, like emails and similar data, it’s better to use tested and more tested libraries.

**⇒ Validator (NPM module)**

```JavaScript
const validator = require('validator');

.
.
.

const User = mongoose.model('User', {
    name: {
        type: String,
        required: true,
				trim: true
    }, 
    email: {
        type: String,
        required: true,
				trim: true,
				lowercase: true,
        validate(value) {
            if (!validator.isEmail(value)) {
                throw new Error('Email is invalid')
            }
        } 
    },
    age: {
        type: Number,
				default: 0,
        validate(value) {
            if (value < 0) {
                throw new Error('Age cannot be less than 0')
            }
        }
    }
});
```

Above, after installing and including `validator`, we can use its methods. In addition, above, there are some small things added that are part of Mongoose. This includes things like `trim` (which removes any unnecessary white spaces), `lowercase` (converts to all lowercase), and `default` (which sets a default value if that parameter is not passed in when making the object.

## Structuring a REST API

REST is a set of architectural constraints, not a protocol or a standard. API developers can implement REST in a variety of ways.

When a client request is made via a RESTful API, it transfers a representation of the state of the resource to the requester or endpoint. This information, or representation, is delivered in one of several formats via HTTP: JSON (Javascript Object Notation), HTML, XLT, Python, PHP, or plain text. JSON is the most generally popular file format to use because, despite its name, it’s language-agnostic, as well as readable by both humans and machines.

Something else to keep in mind: Headers and parameters are also important in the HTTP methods of a RESTful API HTTP request, as they contain important identifier information as to the request's metadata, authorization, uniform resource identifier (URI), caching, cookies, and more. There are request headers and response headers, each with their own HTTP connection information and status codes.

![[Screenshot_2022-10-31_at_10.50.19_PM.png]]

The above diagram is a brief picture of what happens in an API call. REST APIs take need CRUD.

![[Screenshot_2022-10-31_at_10.55.48_PM.png]]

Above, is what we can usually expect, and would need, when designing an API. The one thing to point out is that for reading data, we often provide two methods, one for multiple pieces of data, and one for a singular piece of data.

![[Screenshot_2022-10-31_at_11.00.59_PM.png]]

The above picture is a nice demonstration of what encapsulates an REST API. As seen, we make POST request (create), and in it, we have the respective route handler (/tasks), as well as headers. Now, headers are an important part of REST API’s. They contain **meta-data**.

> _**Metadata is "data that provides information about other data", but not the content of the data, such as the text of a message or the image itself**_

`Accept: application/json` represents the type of the data being posted, `Connection: Keep-Alive` just means to keep the connection open as we expect more requests to be made, and `Authorization` just means we need to be authorized. Then, in the body, is the actual content we want to provide to the API. The response is straightforward as well. It just contains some data like when the request was created, the server that hosts the API, as well as the type and what is being returned.

**⇒ Postman (API Testing Tool)**

We can use Postman to test our http requests. This is probably the most popular tool to use to test API frameworks. Here is what the interface looks like:

![[Screenshot_2022-11-01_at_4.35.27_PM.png]]

## Resource Endpoints

Now, let’s start making our API endpoints. Namely, we make these for each of the CRUD operations. Before starting, we place our models in a new location, and export them to maintain better modularity within the project.

**⇒ Create**

```JavaScript
app.post('/tasks', (req, res) => {
    const task = new Task(req.body)

    task.save().then(() => {
        res.send(task);
    }).catch((error) => {
        res.status(400).send(error);
    })
})
```

**⇒ Read**

```JavaScript
app.post('/users', (req, res) => {
    const user = new User(req.body);

    user.save().then(() => {
        res.send(user);
    }).catch((error) => {
        res.status(400).send(error); // can chain these as they return promises 
        //res.send(error);
    })
})

app.post('/tasks', (req, res) => {
    const task = new Task(req.body)

    task.save().then(() => {
        res.send(task);
    }).catch((error) => {
        res.status(400).send(error);
    })
})
```

So, the above two methods are the `post` methods for each table.

---

We take a short break from designing the API endpoints, and instead, look at Promise Chaining

## Promise Chaining

Now, before, we’ve looked at promise chaining, in the form of things like concatenating `.catch` after `.then` when using Mongoose commands. Now, let’s look at a more advanced promise concept, promise chaining.

Say I want to perform one async task within another one. What would you do? You might have thought of something like this:

```JavaScript
add(1,2).then((sum) => {
	console.log(sum);
	add(sum, 5).then((sum2) => {
		console.log(sum2);
	}).catch((e) => {
		console.log(e);
	});
}).catch((e) => {
	console.log(e);
});
```

While the above example certainly gets the job done, it is far from being a good solution. It’s redundant, and fairly messy/confusing to look at. Hence, is why we need better promise chaining. Here is what a better solution would look like:

```JavaScript
add(1,1).then((sum) => {
	console.log(sum);
	return add(sum,4)
}).then((sum2) => {
	console.log(sum2);
}).catch((e) => {
	console.log(e);
});
```

The benefit of this is that our code is no longer nested. So, this second code snippet allows us to accomplish what the above code does, just without chaining, and maintain much better organization and clarity. The main point of this example is to show that we can chain promises by returning a promise from one. Now, how can we use this when working with the database. Here’s an example:

```JavaScript
require('../task-manager/src/db/mongoose')
const Task = require('../task-manager/src/models/task');

Task.findByIdAndDelete('6361533a72b824c75f13a3e7').then((task) => {
    console.log(task);
    return Task.countDocuments({ completed: false })
}).then((result) => {
    console.log(result);
}).catch((e) => {
    console.log(e);
});
```

As seen above, we can use promise chaining to use the data that is returned from the database query. Now, before we get back to writing our API endpoints, we first introduce `async` and `await`.

## Async + Await

These two are some of the most important tools added to Javascript. Consider the code below:

```JavaScript
const doSomething = () => {}
console.log(doSomething());

// console
undefined 
```

By default, all functions return something, and if you do not explicitly return something, `undefined` is returned by default. However, look what happens if mark the function as async:

```JavaScript
const doSomething = async () => {}
console.log(doSomething());

// console
Promise { undefined }
```

Async functions always return a promise, with the promise bing fulfilled with the value being returned from the function. Essentially, `async` and `await` serve the same purpose as promise chaining, but they are much more clearer to read and understand, which is the main flaw in promise chaining. We add `async` in front of things that will be asynchronous, and await at the place we are calling the `async` function. Now, these two do not introduce any performance improvements, it just makes the code a lot more clear on what its trying to do. Now, here a final look at how we can refactor the promise chaining code from above to use async and await:

```JavaScript
const deleteTaskAndCount = async (id) => {
    const task = await Task.findByIdAndDelete(id);
    const count = await Task.countDocuments({ completed: false });
    return count;
}

deleteTaskAndCount('6361534ac037304d1549b686').then((count) => {
    console.log(count);
}).catch((e) => {
    console.log(e);
})
```

So, now that we’ve learned this, we can refactor our API endpoints to user async and await. Reference the application code base to see the changes that were made.

---

**⇒ Update**

```JavaScript
app.patch('/tasks/:id', async (req, res) => {
    const updates = Object.keys(req.body);
    const allowedUpdates = ['description', 'completed'];
    const isValidOperation = updates.every((update) => {
        return allowedUpdates.includes(update);
    });

    if (!isValidOperation) {
        return res.status(400).send({ error: 'Invalid Update' });
    }

    try {
        const task = await Task.findByIdAndUpdate(req.params.id, req.body, {
            new: true,
            runValidators: true
        });

        if (!task) {
            return res.status(404).send();
        }

        res.send(task);
    } catch (error) {
        res.status(400).send(error);
    }
}); 
```

**⇒ Delete**

```JavaScript
app.delete('/tasks/:id', async (req, res) => {
    try {
        const task = await Task.findByIdAndDelete(req.params.id);

        if (!task) {
            return res.status(404).send();
        }

        res.status(200).send(task);
    } catch (error) {
        res.status(500).send(error);
    }
});
```

So, that wrap up how we can define our API endpoints for the CRUD operations and working with the database. Now, one last thing to fix is the design of the code base. Ideally, we don’t want our routes to all be fixed inside of one file. So, one router for the `users` route, and the one router for the `tasks` route. To begin, we make the add the following to the file where our server is started on:

```JavaScript
const userRouter = require('./routers/user');
const taskRouter = require('./routers/task')
.
.
.
app.use(userRouter); // places our routes in external modules now 
app.use(taskRouter);
```

Essentially, we’ve now told express we want to use these routes. Now inside these files, is where we will place our CRUD API endpoints.

```JavaScript
const express = require('express');
const router = new express.Router();
const Task = require('..//models/task');

router.post('/tasks', async (req, res) => {
    const task = new Task(req.body)

    try {
        await task.save();
        res.status(201).send(task);
    } catch (error) {
        res.status(400).send(error);
    }
})

router.get('/tasks', async (req, res) => {
    try {
        const tasks = await Task.find({});
        res.status(200).send(tasks);
    } catch (error) {
        res.status(500).send(error);
    }
})

router.get('/tasks/:id', async (req, res) => {
    const _id = req.params.id;

    try {
        const task = await Task.findOne({ id: _id });

        if (!task) {
            res.status(404).send();
        }

        res.status(200).send(task)
    } catch (error) {
        res.status(500).send(error);
    }
})

router.patch('/tasks/:id', async (req, res) => {
    const updates = Object.keys(req.body);
    const allowedUpdates = ['description', 'completed'];
    const isValidOperation = updates.every((update) => {
        return allowedUpdates.includes(update);
    });

    if (!isValidOperation) {
        return res.status(400).send({ error: 'Invalid Update' });
    }

    try {
        const task = await Task.findByIdAndUpdate(req.params.id, req.body, {
            new: true,
            runValidators: true
        });

        if (!task) {
            return res.status(404).send();
        }

        res.send(task);
    } catch (error) {
        res.status(400).send(error);
    }
});

router.delete('/tasks/:id', async (req, res) => {
    try {
        const task = await Task.findByIdAndDelete(req.params.id);

        if (!task) {
            return res.status(404).send();
        }

        res.status(200).send(task);
    } catch (error) {
        res.status(500).send(error);
    }
});

module.exports = router;
```

We first declare a router, and then write these endpoints off of the router, instead of the app itself. Nothing else changes. No additional functionality is added nor removed. This just helps our project become more organized — very useful and helpful imo.