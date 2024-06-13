## Section 13: Sorting, Pagination, and Filtering ( Task App )

Now, to continue to build into our application. To begin, one useful feature we should look to add is to provide some more control to the client. What does this mean? Right now, when we request for all of the tasks of a user, we receive everything in the order it was added to the database, which is less than ideal. Ideally, we would want to be able to provide a search bar, in which the user can add keywords, so that the client can return more relevant information, such as only tasks that have that keyword. Maybe we want sort the tasks alphabetically, and maybe we only want completed tasks. In addition, what if we want to introduce pagination, so that only some data is shown on one screen, making it so that we do not need to load all tasks at once? These are all capabilities we wish to provide to the client.

**⇒ Working with timestamps**

We want to include timestamps inside of our user model, to know when the user was created/updated. Now, to add this this, it is a very simple add on the user and task schema, where we just add a parameter onto the schema:

```JavaScript
const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true
    },
    email: {
        type: String,
        unique: true, // guarantees only one email per user, will need to wipe db to use an index to keep track 
        required: true,
        validate(value) {
            if (!validator.isEmail(value)) {
                throw new Error('Email is invalid')
            }
        }
    },
    password: {
        type: String,
        required: true,
        trim: true,
        minlength: 7,
        validate(value) {
            if (value.toLowerCase().includes('password')) {
                throw new Error('Password cannot contain the word "password"');
            }
        }

    },
    age: {
        type: Number,
        validate(value) {
            if (value < 0) {
                throw new Error('Age cannot be less than 0')
            }
        }
    },
    tokens: [{
        token: {
            type: String,
            required: true
        }
    }]
},
    {
        timestamps: true
				// above, sets the timestamps to true 
    }
);
```

Now, let’s look at something else.

**⇒ Filtering Data**

As this application grows, the number of tasks in the database will continue to grow and grow. However, this becomes a problem when we wish to retrieve all of the tasks, as this process can become slow and unpleasant to use, if w’re getting data we don’t want (such as completed tasks). So, it is therefore important to allow the user to have the ability to narrow down their search criteria, to filter the type of data we want to be presented to the user. To do this, we make use of the **query string**.

```JavaScript
// GET /tasks?completed=false, only get the incomplete tasks
// GET /tasks?completed=true, only get the complete tasks
router.get('/tasks', auth, async (req, res) => {
    try {
        if (req.query.completed) {
            const completed = (req.query.completed === 'true'); // this is an ineterestig JS syntax use case 

            const tasks = await Task.find({
                owner: req.user._id,
                completed
            });

            res.status(200).send(tasks);
        } else {
            const tasks = await Task.find({
                owner: req.user._id,
            });

            res.status(200).send(tasks);
        }
    } catch (error) {
        res.status(500).send(error);
    }
})
```

This has required some changes to made to the code in the route handler, so the above code, at the moment, is not written well. Nonetheless, it should demonstrate the point of this section. Now, to fetch tasks for a user, we can make 3 requests:

```JavaScript
GET /tasks?completed=false // this gets all uncompleted tasks 
GET /tasks?completed=true
GET /tasks
```

So, this allows us to do that.

**⇒ Pagination**

Now, we don’t always want to load every single to a web page, as it is unnecessary and can make our application very slow. Now, there are many forms in which pagination can materialize. For instance, Google pages have pagination at the bottom, as Google loads 10 results onto the page for each search. This is the most obvious use of pagination. Some other less apparent areas are things like Instagram. As you may have noticed, Instagram does not load all posts when a user is scrolling. Instead, it loads the next few when you are approaching those posts. To set up these kinds of limits and give control to the client, we again, need to add some changes to the tasks router handler. Here’s a quick model of what this would look like:

```JavaScript
// GET /tasks?limit=10&skip=10
```

So, what do these parts of the query string do? For now, we’ll model this after google. So, each page will show at most 10 results (hence we set limit to 10). Then, skip to 10. This is so that if we want to move onto the next page, we skip the first ten results (the ones we have right now), and display the next 10 results. Here is what the updated router for get /tasks would look like with pagination in place:

```JavaScript
router.get('/tasks', auth, async (req, res) => {
    const match = {
        owner: req.user._id,
    }

    if (req.query.completed) {
        match.completed = req.query.completed === "true"
    }

    try {
        const tasks = await Task.find(
            match,
            null,
            {
                limit: parseInt(req.query.limit), // this is from query string, a string, need integer, this js function converts this for us
	                skip: parseInt(req.query.skip),
            }
        );

        res.status(200).send(tasks);
    } catch (error) {
        res.status(500).send(error);
    }
})
```

According to the Mongoose documentation find is looking for the parameters in this order: [conditions], [projections], [options], [callback]. So, we provide null for projections, and then pass in the object with options we want to add, like limit and skip. Through this, we have now effectively set up pagination for getting tasks. Now, let’s look how we can sort our data, such as by date created.

**⇒ Sorting Data**

This is doable as we earlier had timestamps enabled.

```JavaScript
// GET /tasks?sortBy=createdAt_asc
// GET /tasks?sortBy=createdAt_desc
```

Now, to actually be able to sort the order in which the tasks are returned when they are called, is by adding an additional parameter to the query string, and then analyzing it from there:

```JavaScript
router.get('/tasks', auth, async (req, res) => {
    const match = {
        owner: req.user._id,
    }
    const sort = {}

    if (req.query.completed) {
        match.completed = req.query.completed === "true"
    }

    if (req.query.sortBy) {
        const parts = req.query.sortBy.split('_'); // this splits the string by the character passed, which in our case is the underscore, so now we know the type of sorted needed to be done 
        sort[parts[0]] = parts[1] === "desc" ? -1 : 1
    }

    try {
        const tasks = await Task.find(
            match,
            null,
            {
                limit: parseInt(req.query.limit), // this is from query string, a string, need integer, this js function converts this for us
                skip: parseInt(req.query.skip),
                sort
            }
        );

        res.status(200).send(tasks);
    } catch (error) {
        res.status(500).send(error);
    }
})
```