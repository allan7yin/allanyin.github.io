  

Now, now that we have successfully created the application. We shift our attention towards connecting NodeJS to a database.

  

  

### MongoDB and NoSQL databases

  

So, what’s difference between SQL and NoSQL. Here is a graph that shows this:

![[IMG_4110.jpg]]

  

So, how do we use MongoDB? First, we need to first download MongoDb. After downloading it, we can start the database by typing: `/Users/allanyin/mongoDb/bin/mongod --dbpath=/Users/allanyin/mongoDb-data`. Note that this is for my instance of mongodb downloaded, will be different depending on where the file is placed.

  

Now, to make accessing/working with the database to be more efficient and accessible, we can install a database GUI viewer. Basically, lets you view the tables, and everything that are inside of your database. For this course, we’ll use studio 3T.

  

Now, we want to connect to the mongoDB database through NodeJS. To do so, we use the mongoDB driver. Here is some example code. In this, we make collections in the database:

```JavaScript
const mongodb = require('mongodb');
const MongoClient = mongodb.MongoClient;

const connectionURL = 'mongodb://127.0.0.1:27017';
// the above is the same as writing 'mongodb://localhostL27017', instead, we have used the ip address instead of local host as it is fatser, we don't know
// why that is, it just is
const databaseName = 'task-manager';

MongoClient.connect(connectionURL, { useNewUrlParser: true }, (error, client) => {
    if (error) {
        return console.log('Unable to connect to the database');
    }

    const db = client.db(databaseName);
    db.collection('tasks').insertMany([
        {
            description: 'Finish NodeJS course',
            completed: false
        },
        {
            description: 'Practice LeetCode',
            completed: false
        },
        {
            description: 'Diet',
            completed: false
        },
        {
            description: 'Study hard',
            completed: false
        }
    ]);
});
```

What we see in the database GUI is:

![[Screen_Shot_2022-10-12_at_8.56.14_PM.png]]

  

Now, you may have noticed that each item in the collection has a unique object id. This is one of the differences between MongoDB and sql databases. IN SQL, things in a table are ordered/stored by numerical indexes. However, here, MongoDB has an algorithm that generates these objects ID’s which are unique.

⇒ **Reading From a Database**

So, above, we are now able to create a database by instaling MongoDB, as well as creating entires inside of the db. Now, let’s look at how we can query the database. In particular, we’ll look at find, and findOne. For instance, say we ave the users collection:

![[Screenshot_2022-10-29_at_1.46.17_PM.png]]

And we want to retrieve Sophia’s information from the DB. We can do:

```TypeScript
db.collection('users').findOne({ name: 'Sophia'}, (error, response) => {
        if (error) {
            console.log('Unable to fetch');
        }

        console.log(response);
    })
```

Now, we always get the first item that matches our search criteria. So, if I looked for an entry with the name Allan, I would get the first one. If I wanted a different one, I would need to search by the unique ID. Similarly, if I search for something that is not in the collection, I would get a return value of null.

⇒ **fineOne()**

The findOne method is fairly straightforward. It take on two parameters, the first one is the search parameter, and the second one is what is found/response.

Here is the API documentation: [https://mongodb.github.io/node-mongodb-native/4.11/classes/Collection.html#findOne](https://mongodb.github.io/node-mongodb-native/4.11/classes/Collection.html#findOne)

⇒ **find**

The find is different in that it returns a cursor. We can think of a cursor as a pointer to data, which we can then call other methods to retrieve the data that fits the kind we are looking for. We can call many methods, msuch as count, etc. A very important method that is used is toArray, which stores all matching results in the array. Then, inside of toArray, we will have the callback function, which will look something like:

```TypeScript
db.collection('users').find({ age: 19 }).count((error, response) => {
        console.log(response);
    })
```

---

### Promises

Promises are something that is being introduced as they solve many of the problems callbacks may introduce with our code. They improve on many of the short comings that callback functions have. Consider the code below:

⇒ **Callback Method**

```JavaScript
const doWorkCallback = (callback) => {
    setTimeout(() => {
        // callback('This is my error!', undefined)
        callback(undefined, [1, 4, 7])
    }, 2000)
}

doWorkCallback((error, result) => {
    if (error) {
        return console.log(error)
    }

    console.log(result)
})
```

Assume we had the callback function above. So, after 2 seconds, we call the callback function. Now, how do we know if we were successful? Well we know this through what we pass to the calback. If we have an error, we pass an error along, and leave the result (what we expect to come back) as undefined. Similarly, if we do get a successful call, then we return our result as well as the error parameter as undefined.

  

The issue with this is that it can be hard to tell what the code is doing simply by reading it. In other words, the semantics are not good for the callback function method. Now, here is what a promise looks like:

**⇒ Promise Method**

```JavaScript
const doWorkPromise = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve([7, 4, 1])
        // reject('Things went wrong!')
    }, 2000)
})

doWorkPromise.then((result) => {
    console.log('Success!', result)
}).catch((error) => {
    console.log('Error!', error)
})

//
//                               fulfilled
//                              /
// Promise      -- pending --> 
//                              \
//                               rejected
//
```

This is what working with a promise would look like. Instead of passing a callback to the function, we instantiate a Promise object, and pass it a function when the asynchronous action is done. Now, right off the bat, the semantics of this method are much better. It is much clearer what each function means, and the error handling measures are much more understandable. Two parameters will be passed to the function, and both are very clear on what they do. If an error occurs, we call the `reject` function, and pass it the error. If the action was successful, then the `resolve` function is called, and is passed what is supposed to be returned.

⇒ The syntax is slightly different for the function waiting on the promise. `.then(function)` is if the `resolve` function is called, and the `.catch(function)` is if the `reject` function is called.

  

---

**⇒ Updating Database Entries**

Now, let’s look at how to update documents in our database. To update an entry in the database, we can use the functions `updateOne` and `updateMany`. Now, when learning this, let’s use promises instead of callback functions. The syntax of updating entries in a database are fairly similar to the ones of findOne, and find. Here is how we would use them:

```JavaScript
db.collection('users').updateOne({ _id: new ObjectId('6345f645eb439883b8754bf9')}, {
        $set: {
            name: 'Bob'
        }
    }).then((result) => {
        console.log(result)
    }).catch((error) => {
        console.log(error) 
    });


db.collection('tasks').updateMany({ 
        completed: false
    }, {
        $set: {
            completed: true
        }
    }).then((result) => {
        console.log(result.modifiedCount);
    }).catch((error) => {
        console.log(error);
    })
```

**⇒ Deleting Database Entries**

Now, like update and find, we will be ale to delete a single entry, or delete multiple at the same time. Here is how we could delete multiple, the way of searching for things to delete is the same as finding. So, the syntax is very straightforward:

```JavaScript
db.collection('users').deleteMany({
        name: 'Allan'
    }).then((result => {
        console.log(result)
    })).catch((error) => {
        console.log(error)
    })
```