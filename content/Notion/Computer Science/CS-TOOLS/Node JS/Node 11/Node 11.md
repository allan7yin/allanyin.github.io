## Section 14: File Uploads

Now, something else we can now look at is file uploads. This is a feature we will want to add to our task manager application. To begin, let’s try to upload a picture, as a profile image for user. First, we need to configure our application to be able to do so.

**⇒ Setting up**

By itself, express does not support file uploads. However, the creators of express have created a NPM module for this: [https://www.npmjs.com/package/multer](https://www.npmjs.com/package/multer).

```JavaScript
const multer = require('multer');
const upload = multer({
    dest: 'images' // folder where all of the uploads will be stored, can name it images for this project, as only storing images 
});
app.post('/upload', upload.single('upload'), (req,res) => { // the second parameter is middleware, need it
    res.send()
})
```

When we are trying to make a Postman request to upload an image, we must use form-data. Then, we need to provide a key value pair. To do so, look at the setup code for multer. Notice that we passed the word “upload” to upload.sinle(). That is the key mutler is looking for, so that is the key we must use in Postman, with a value of the file

![[Screenshot_2022-11-27_at_10.23.58_AM.png]]

After making that POST request, and receiving a 200, we can now look inside of our task-manager application, and see the folder named ‘images’, created accordingly, with the uploaded image inside of it.

![[Screenshot_2022-11-27_at_10.25.42_AM.png]]

Now, the above is merely an example of how we want to use this. Now, let’s actually add this to our router for the user, to allow them to upload this image. Here is how we would need to change the router file:

```JavaScript
const upload = multer({
    dest: 'avatars' // folder where all of the uploads will be stored, can name it images for this project, as only storing images 
});


router.post('/users/me/avatar', upload.single('avatar'), (req, res) => { // the second parameter is middleware, need it
    res.send()
})
```

So, the code is the same as above, all we’ve done now is place this inside of the user router, and rename the file to avatar, to match its purpose more. Now, let’s look at how we can validate the files being uploaded to our server. We can look at two validations: file size and file type.

**⇒ Validating file uploads**

In order to do this, we attach some additional parameters to the multer object, as follows:

```JavaScript
const upload = multer({
    dest: 'avatars', // folder where all of the uploads will be stored, can name it images for this project, as only storing images 
    limits: {
        fileSize: 1000000 // 1mb max, this file size limit is in bytes 
    },
    fileFilter(req, file, cb) { // this gets internally called by multer when attempting to upload a file 
        // req is thr request being made 
        // file is information about the file being uploaded 
        // cb is the callback, use this to tell multer when we're done filtering the file 
        if (!file.originalname.match(/\.(jpg|jpeg|png)$/)) { // we can use regex expressions to match multiple types of files 
            return cb(new Error('Please upload an image'))
        }

        cb(undefined, true);
    }
});
```

Now, there is something we should probably address, and that is express errors. When we try to upload something and it fails, Express sends back an error, that can shows more information than we’d like. We can address this:

**⇒ Handling Express Errors**

To handler express errors, we append an error catching function to the route handler:

```JavaScript
router.post('/users/me/avatar', upload.single('avatar'), (req, res) => {
    res.send()
}, (error, req, res, next) => {
    res.status(400).send({ error: error.message })
});
```

The second function with `(error, req, res, next)` is how we are able to manipulate the error Express sends back.

**⇒ Adding images to User Profile**

Now that we are able to upload images, we want to be able to make them a part of the user model. First, we need to make a parameter on the user model that will store said data:

```JavaScript
}
    },
    tokens: [{
        token: {
            type: String,
            required: true
        }
    }],
    avatar: {
        type: Buffer // can store binary data here for image 
    }
},
    {
        timestamps: true
    }
);
```

We can simply add the field, Buffer, which is data type of images, once saved. To save the photo here, we make modifications to the user routes code base:

```JavaScript
const upload = multer({
    // dest: 'avatars', need to get rid of this in order to get access toe image binary data, since if not saving here, it gets passed to cb in post route 
    // We are doing this as we want to save images not to one directory, but to the user model 
    limits: {
        fileSize: 1000000
    },
    fileFilter(req, file, cb) {
        if (!file.originalname.match(/\.(jpg|jpeg|png)$/)) {
            return cb(new Error('Please upload an image'))
        }

        cb(undefined, true);
    }
});


router.post('/users/me/avatar', auth, upload.single('avatar'), async (req, res) => { // multiple middleware 
    req.user.avatar = req.file.buffer; // can only access file.buffer when  we are not using dest 
    await req.user.save();
    res.send();
}, (error, req, res, next) => {
    res.status(400).send({ error: error.message });
});
```

As seen, we need to remove the dest field of multer. This is because, we don’t want out pictures to be saved to a common folder, but as binary data to each user. Now, if we upload an image, and get the user profile, this is what we will see in the database:

![[Screenshot_2022-11-28_at_10.37.48_AM.png]]

Now, say we wanted to use this in the front-end. Traditionally, your probably used to using a url link as a source for an image, as the browser can retrieve the image from the provided url. With binary data, we can have images in the front-end using the binary data:

```JavaScript
<body>
	<h1> Test </h1>
	<img src="data:image/jpg;base64,(here goes the data from above, for avatar, something like /9j/4AA...)">
</body>
```

Now, here another way of doing this, that is more preferable. Here are the routes for the user to delete and get the avatar:

```JavaScript
router.delete('/users/me/avatar', auth, async (req, res) => {
    req.user.avatar = undefined; // removes the image if user is authorized 
    await req.user.save();
    res.send();
})

router.get('/users/:id/avatar', async (req, res) => {
    try {
        const user = await User.findById(req.params.id);

        if (!user || !user.avatar) {
            throw new Error();
        }

        res.set('Content-Type', 'image/jpg'); // sets response header, provide a key value pair 
        res.send(user.avatar);
    } catch (e) {
        res.status(404).send();
    }
})
```

As seen, the delete route is straight forward, and just removes the avatar from the user in the database. The second one is a route to get the avatar of a user given their id. This is very useful when it comes to providing this image to the front end. Now, instead of the doing what the previous example did, we can simply use this url as a way for the browser to fetch this image:

```JavaScript
<body>
	<h1> Test </h1>
	<img src="http://localhost:3000/users/637d16d66d66f9fa92b15f51/avatar">
</body>
```

**⇒ Auto-Cropping and Image Formatting**

Now, something good to have would be the ability to auto format. To do so, we install the library Sharp, and use it to format the images the moment they are uploaded:

```JavaScript
router.post('/users/me/avatar', auth, upload.single('avatar'), async (req, res) => { // multiple middleware 
    const buffer = await sharp(req.file.buffer).resize({ width: 250, height: 250 }).png().toBuffer();
    // .png converts it to a png format, resize is self-explanatory 

    req.user.avatar = buffer;
    await req.user.save();
    res.send();
}, (error, req, res, next) => {
    res.status(400).send({ error: error.message });
});
```