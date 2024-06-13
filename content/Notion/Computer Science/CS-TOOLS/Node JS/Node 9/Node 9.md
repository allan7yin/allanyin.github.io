## API Authentication and Security

Currently, for this task application, we have passwords saved just as the strings they are, which for obvious reasons, is a terrible idea. So, we want to look for a way to store sensitive data in a secure way.

**⇒ Hashing**

The hashing algorithm we will take a look at is b-crypt. [https://www.npmjs.com/package/bcrypt](https://www.npmjs.com/package/bcrypt). To hash a password, we will do something like this:

```JavaScript
const bcrypt = require('bcryptjs')
const myFuction = async () => {
    const password = 'Alln12345';
    const hashedPassword = await bcrypt.hash(password, 8);
}
```

The second argument (8), represents the number of time we want the hashing algorithm to be run. This is also the number recommended by creator, as it reaches both a very secure hash, as well as a pretty fast time.

**⇒ Hashing Vs. Encryption**

The above hash yields this:

```JavaScript
password: Alln12345
hashedPassword: $2a$08$cqnR3Rk2Bj5czmQPgs95XOLOwrAFgl4P264ra5AJFVTElyeACeD..
```

While the too share similarities, hashing is like literally, half of encryption. We can hash our passwords into that random string above. However, it is irreversible. So, given a hashed string, we cannot un-hash it. On the other hand, when we encrypt our passwords, we can always decrypt it. Then, to compare, say login passwords, with the one stored in the database, we can again, use a `bcrypt` method:

```JavaScript
const isMatch = await bcrypt.compare('Allan12345', hashedPassword);
    console.log(isMatch);
```

**⇒ Adding password hashing to the code base**

In order to add password hashing to the code base, some changes are needed. This is because some of the functions that interact with the database. In particular, we need to change the update function. This is because the `findByIdAndUpdate` function bypasses Mongoose. It performs a direct operation on the database, and as a consequence, over looks the middleware hashing we want to incorporate. So, we change the update function into this:

```JavaScript
router.patch('/users/:id', async (req, res) => {
    const updates = Object.keys(req.body);
    const allowedUpdates = ['name', 'email', 'password', 'age'];
    const isValidOperation = updates.every((update) => {
        return allowedUpdates.includes(update);
    });

    if (!isValidOperation) {
        return res.status(400).send({ error: 'Invalid Update' });
    }

    try {

// new part 
        const user = await User.findById(req.paras.id);
        updates.forEach((update) => {
            user[update] = req.body[update]
        });

        await user.save()
// end of new part 

        // const user = await User.findByIdAndUpdate(req.params.id, req.body, {
        //     new: true,
        //     runValidators: true
        // });

        if (!user) {
            return res.status(404).send();
        }

        res.send(user);
    } catch (error) {
        res.status(400).send(error);
    }
});
```

With this set in place, we can now add the middleware logic to hash our passwords inside of the user model. We need to do a bit of refactoring. Before, We just had a model called ‘User’, and put everything inside of it, such as the email, name, etc. Now. instad of putting this information inside of a const, we place it inside of a schema. Before, we had something like this:

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

Now, we change into this:

```JavaScript
const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true
    }, 
    age: {
        type: Number
    }
});

const User = mongoose.model('User', userSchema);
```

Just a small change. But, this is important as it allows us now to do this:

```JavaScript
userSchema.pre('save', async function (next) {

    if (this.isModified('password')) { // this ensures that we onlt hash when the password is modified, mongoose provides us this function 
        this.password = await bcrypt.hash(this.password, 8); // remember, this is the current thing callig the fuction, in which case, is the user 
    }

    console.log('alsdkjasldk');

    next();
})
```

Which means, before we perform any saving, we want to run this function. The `next` is very important, though its not important to know what it does. This pretty much wraps up this section of looking into how we can save passwords and how we should be hashing them.

## Logging in Users

The most basic place to start with being able to login users is to add this to the User schema. To do so, we can add functions to the User schema, as so:

```JavaScript
userSchema.statics.findByCredentials = async (email, password) => {
    const user = await User.findOne({ email });

    if (!user) {
        throw new Error('Unable to login');
    }

    const isMatch = await bcrypt.compare(password, user.password)

    if (!isMatch) {
        throw new Error('Unable to login')
    }

    return user;
}
```

This makes it so we can call methods off the User modal in our routes:

```JavaScript
router.post('/users/login', async (req, res) => {
    try {
        const user = await User.findByCredentials(req.body.email, req.body.password);
        res.send(user);
    } catch (e) {
        res.status(400).send();
    }
})
```

**⇒ JSON Web Token**

So, here is something that you have probably seen before during co-op, it’s working with JWT. Now that are able to login a user, we’ll want to to start changing the publicity of our routes. Some should only be available if someone is logged in, while others can be public. In addition, some tings like deleting tasks should only be able to be done by the tasks owner, not some other user. So, how do we use it? To start, we can install `jsonwebtoken` as a `npm` package to use it:

```JavaScript
const jwt = require('jsonwebtoken');

const myFunction = async () => {
    const token = jwt.sign({ _id: 'abc123' }, 'thisismynewcourse', { expiresIn: '7 days' });
    console.log(token);

    const data = jwt.verify(token, 'thisismynewcourse');
    console.log(data);
}

myFunction();
```

The first line, makes our token. It is made with some string, we pass to it only id, as that’s all we really need. The next string is the key, and is what is used to help generate the JWT. After that, we can add an optional parameter that tells when the JWT expires. Now, what we can do this, we can allow a token to be generated whenever a use logins in, or is creating an account for the first time:

```JavaScript
router.post('/users', async (req, res) => {
    const user = new User(req.body);
    try {
        const token = await user.generateAuthToken();
        await user.save(); // if this throws an error, the code below it will not be run, one of the benefit of async and await 
        res.status(201).send({ user, token });
    } catch (error) {
        res.status(400).send(error);
    }
})

router.post('/users/login', async (req, res) => {
    try {
        const user = await User.findByCredentials(req.body.email, req.body.password);
        const token = await user.generateAuthToken();
        res.send({ user, token });
    } catch (e) {
        res.status(400).send();
    }
})
```

The generateAuthToken is not called from the User model, but rather, on an instance of the User model. Here is what is would look like on the User schema:

```JavaScript
userSchema.methods.generateAuthToken = async function () {
    const token = jwt.sign({ _id: this._id.toString() }, 'thisismynewcourse');

    this.tokens = this.tokens.concat({ token });
    await this.save();

    return token;
}
```

Notice the difference. For functions on the entire model, we would use `userSchema.statics`, where as for functions of an instance of the model, we use `userSchema.methods`. There is one more thing we want to add, and this an array of tokens to an account. We now want to update the user schema to this:

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
})
```

We are now adding a tokens parameter onto the schema. Why? We need this to provide a way for users to log out and invalidate tokens. To fix this, we track tokens generated by users by creating an array of them. This will allow a user to login from multiple devices (such as a laptop or a phone) and when logging out of one of them, stay logged in on the others.

## Express Middleware

Now that our application is creating and keeping track of authentication tokens being generated, its time to look at how we use these tokens to authenticate other requests. At the core of allowing us to accomplish this, is express middleware. Here’s a simple diagram on how it works:

```JavaScript
// without middleware: 
// new request -> run route handler 

// with middleware
// new request -> do something -> run route handler
```

To use this middleware, expess actually provides us a third point when handling routes:

```JavaScript
router.get('/users/me', auth, async (req, res) => {
    // now, when we get users, first runs auth, the middleware, and if next is called, then processes the request 
    return req.user;
})
```

The auth above, is an imported function we defined. Here is what it looks like:

```JavaScript
const jwt = require('jsonwebtoken');
const User = require('../models/user');
const auth = async (req, res, next) => {
    try {
        const token = req.header('Authorization').replace('Bearer ', '');
        const decoded = jwt.verify(token, 'thisismynewcourse'); // verify returns the decoded payload, or throws an error, which will be caught in the catch block
        const user = await User.findOne({ _id: decoded._id, 'tokens.token': token })
        // the tokens.token verifies that the user indeed does have te token for authenticatio

        if (!user) {
            throw new Error()
        }

        req.user = user; // we've already retreated once from the db, so we can add this property to the request 
        next()
    } catch (e) {
        res.status(401).send({ error: 'Please Authenticate ' })
    }
}

module.exports = auth;
```

Essentially, we do a couple of things. First, we check the parse the request header to obtain the JWT. Then, we get the payload to obtain the _id value to query the database with. Errors are thrown and caught where necessary.

---

We take a short break from and look at some features of Postman we will want to use move moving forward:

**⇒ Environment Variables**

So, instead of typing [localhost](http://localhost):3000 each time in each request in Postman, we can set some environment variables:

![[Screenshot_2022-11-13_at_5.01.57_PM.png]]

Here, we can set some environment variables:

![[Screenshot_2022-11-13_at_5.02.51_PM.png]]

Now, another thing we would want to configure is to implement a way in which we can automatically update the authToken through some scripts:

```JavaScript
// runs after request is made 

if (pm.response.code === 200) {
    pm.environment.set('authToken', pm.response.json().token)
}
```

Now, we would only need this Test (something that runs after a request is made) when we are creating a user or logging one in. In addition, we don’t want to configure headers for each request, as that would be very laborious. Instead, we can set some for the entire collection of requests.

![[Screenshot_2022-11-13_at_5.05.47_PM.png]]

---

Now, lets jump back into the use writing code for authorization. We should probably adjust the data we are sending back when a user logins in. Right now, here is what is being returned:

```JavaScript
{
    "user": {
        "_id": "636ec1e9f1cd8a2bb4fa3fce",
        "name": "Allan Yin",
        "email": "a7yin@uwaterloo.ca",
        "password": "$2a$08$S0u6gIG1oeCDvlWNCj3pV.TitjZzqWvLCecZa8dTytk3c/hwmbo0K",
        "tokens": [
            {
                "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MzZlYzFlOWYxY2Q4YTJiYjRmYTNmY2UiLCJpYXQiOjE2NjgzODI5NTZ9.fjHBiAyIIMJseL7zaMMDwQf3nBoar63ITIxEi9i_swA",
                "_id": "637180ec7972ea16f8fa1d7f"
            }
        ],
        "__v": 20
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MzZlYzFlOWYxY2Q4YTJiYjRmYTNmY2UiLCJpYXQiOjE2NjgzODI5NTZ9.fjHBiAyIIMJseL7zaMMDwQf3nBoar63ITIxEi9i_swA"
}
```

However, there are some things we should pieces of data we should probably never be sending, such as password and the tokens array, as these are both unnecessary and dangerous. So, we want to be able to hide certain data when sending it back from the client. The trick is to actually override a function. When we do `res.send`, Mongodb is actually first calling `JSON.stringify(something)` first. So, we can add a method called `toJSON` to the object to control what the JSON being sent back can include. Here’s what it would look like now:

```JavaScript
userSchema.methods.toJSON = function () {
    const userObject = this.toObject(); // Mongoose methods, turns the user calling this method into an object 
    delete userObject.password;
    delete userObject.tokens;

    return userObject;

}
```

With this solution, we won’t need to change any code in our routers, as the allowed data to be sent back is now being filtered when turning the object into a JSON. So, this is a quick way to hide certain data from being sent back.

**⇒ The User/Task Relationship**

The relationship between the 2 is a owns-a relationship. A user can own many tasks, but a task can only have one owner. Likewise, a task cannot exist independently, and must belong to some user. On the contrary, a user can exist without owning tasks. For this reason, we’ll store the id of the owner in the task’s model. We could have an array of the ids of the tasks owned in the user model, both work.

**⇒ Establishing the relationship**

This area is kind of similar to how we could establish relationships between classes in C++. Here is what we would do. First, let’s update the task schema:

```JavaScript
const validator = require('validator');
const mongoose = require('mongoose');

const Task = mongoose.model('Task', {
    description: {
        type: String,
        required: true,
        trim: true,
    },
    completed: {
        type: Boolean,
        default: false
    },
    owner: {
        type: mongoose.Schema.Types.ObjectId,
        required: true,
        ref: 'User' // this creates a reference from this field to another model, which in this case, is the User model (spelling and capitals matter)
        // this will allow us to fetch an entire user from this field of a task 
    }
});

module.exports = Task;
```

As seen, we now add a new parameter, called owner, which has a type of objectId. This is supplied my Mongoose. We add the ref to help establish the relationship between this and the user model. More importantly, however, this ref allows us to access the user through one of their tasks:

```JavaScript
const task = await Task.findById('63718ded752fe86f53e5fa27');
await task.populate('owner'); 
// this converts the owner from simply being an id, and into the actual owner, it populates data from a relationship, from owner 
console.log(task);
```

Now, if we did not populate, we would simply only get the owner’s object id back. To be able to retrieve the entire owner’s data, we use populate. So, the above code will log:

```JavaScript
{
  _id: new ObjectId("63718ded752fe86f53e5fa27"),
  description: 'Get Girlfriend',
  completed: false,
  owner: {
    _id: new ObjectId("63718c8cf28ccf135338c247"),
    name: 'Allan Yin',
    email: 'a7yin@uwaterloo.ca',
    password: '$2a$08$4cM0ha4Uro4ryKoclw2vDOcc/9Bf1rzSklDRZgc4vkVecYg6AwMXu',
    tokens: [ [Object] ],
    __v: 2
  },
  __v: 0
}
```

Now, we also want this relationship to be accessible from the owner side. That is, we want to be able to access the tasks that belong to a certain user. To do so, we need to use something called virtual property. We add the following into user model file:

```JavaScript
// so, why do we have the below? It exists as a virtual property of the schema. Meaning, it is not an actual atribute of the schema, but a way for 
// Mongoose to understand the relationship between a task and a User 
userSchema.virtual('tasks', {
    ref: 'Task',
    localField: '_id',
    foreignField: 'owner'
});
```

Then, to grab the tasks of a user, we can:

```JavaScript
const user = await User.findById('63718c8cf28ccf135338c247');
await user.populate('tasks');
console.log(user.tasks);
```

In this case, without the populate, tasks is not a property of a user, and will just return an undefined. Only after we populate it using the virtual property we defined, will it log:

```JavaScript
[
  {
    _id: new ObjectId("63718ded752fe86f53e5fa27"),
    description: 'Get Girlfriend',
    completed: false,
    owner: new ObjectId("63718c8cf28ccf135338c247"),
    __v: 0
  }
]
```

And, if we create a new task, we’ll get:

```JavaScript
[
  {
    _id: new ObjectId("63718ded752fe86f53e5fa27"),
    description: 'Get Girlfriend',
    completed: false,
    owner: new ObjectId("63718c8cf28ccf135338c247"),
    __v: 0
  },
  {
    _id: new ObjectId("63719c906058c6139d9fdcdd"),
    description: 'Study for Math237 Final',
    completed: false,
    owner: new ObjectId("63718c8cf28ccf135338c247"),
    __v: 0
  }
]
```

Now, we have defined the relationships between the two models. We now move on to authenticating the task API endpoints.

**⇒ Authenticating Task Endpoints**

Now, check the repo to see how authentication was added to the tasks. Essentially, we query the database for both the task id, and to make sure there is the correct owner. Now that we’re done with establishing the API endpoints for the tasks and user, one last thing we should implement is cascading delete. That is, when we decide to delete a user, their respective tasks should also be deleted. To do so, we make a new userSchema method, this will allow us to delete all of the tasks that belong to a user once we delete that user:

```JavaScript
userSchema.pre('remove', async function (next) {
    await Task.deleteMany({ owner: this._id })
    next()
});
```

For this, before we remove a user, we delete all of their tasks, hence successfully cascading delete to be more clean.