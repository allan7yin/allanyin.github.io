# Section 9: Application Deployment

  

Now that the weather application is up and running, we now learn how to deploy it. We’ll be looking at Heroku and GitHub

  

### Git

Git provides us version control and keeps track of project progress.

  

![[IMG_3891.jpg]]

  

### Integrating Git

To integrate git, we being by entering, in the project directory, `git init`. This creates a .git file which will store our commits.

![[Screen_Shot_2022-10-05_at_10.13.49_PM.png]]

Doing so, Vscode by default makes our untracked files in green font. As we will never manually interact with the .git file, Vscode has automatically hidden that file from us. Running `git status` logs the status of our project, as so:

  

![[Screen_Shot_2022-10-05_at_10.16.54_PM.png]]

  

However, we don’t want git to track the `node_modules` directory. To do so, we create the file `.gitignore` which we then place `node_modules/` inside of it. To ad things to the staging area, we simply:

```JavaScript
// git add (folders or file paths go here)

// e.g.
git add /src
```

![[Screen_Shot_2022-10-05_at_10.21.58_PM.png]]

  

To add everything, we just do `git add .` . To commit, we need:

```JavaScript
git commit -m "Init commit"
// the -m is the message which is required 
```

  

Now, we have one commit in our repository.  
  

Now that we have committed our code, we can send it to some third-party applications like Github and Heroku. To do so securely, we need to set up a SSH key pair.

  

### Setting SSH Key Pairs

We can begin with the command in the terminal:

```Bash
ls -a -l ~/.ssh // this will list out ssh files 
ssh-keygen -t rsa -b 4096 -C "allanyin17@gmail.com" 
// this generates the ssh files
```

![[Screen_Shot_2022-10-05_at_10.58.55_PM.png]]

  

The id_rsa file is the one that we should never share with others. It exists only as our secret. The id_rsa.pub is the public one, and so, the one we can provide Github and Heroku. Next, we need to prepare the keys, so we do: `ssh-add -K ~/.ssh/id_rsa`. Now, we can begin pushing our code to Github. First, we create a repo on Github. Then, we select:

![[Screen_Shot_2022-10-06_at_8.45.07_AM.png]]

  

since we already have the repository on our local machine.

![[Screen_Shot_2022-10-06_at_8.47.52_AM.png]]

The key is in `cat ~/.ssh/id_rsa.pub`

  

Now, the `git push -u origin main` will work. Now, our code has been pushed to Github. Now, let’s see how to push to Heroku.

  

### Heroku

  

![[Screen_Shot_2022-10-06_at_8.55.02_AM.png]]

![[Screen_Shot_2022-10-06_at_8.56.32_AM.png]]

![[Screen_Shot_2022-10-06_at_8.57.53_AM.png]]

So, we could just do `npm run start`, to start our project locally. We do this so we can tell Heroku how to start our application. After making the neccesary changes and comitting those changes, we can run `git push heroku main`. Now, we have successully run it on Heroku: [https://a7yin-weather-application.herokuapp.com/](https://a7yin-weather-application.herokuapp.com/)

  

**DONE**

  

Now, say when developing this application, I don’t want to continually have to write `nodemon src/app.js`, well, we can write a script for this. `npm i nodemon --save-dev`, makes this a local dependency. Before, if we have nodemon be global, well, it won’t be listed as a dependency, so if someone cloned this repo, they would not be able to run the code, as they also don’t know what version of nodemon I used. To avoid this we need to install nodemon as a local dependency. This way it gets added to package JSON, and so the person downloading my code will be able to download the needed dependencies. Now, we have this in package.json:

  

![[Screen_Shot_2022-10-06_at_10.26.06_AM.png]]

  

Now, one small change. As it is no longer global, we can no longer run nodemon CLI in the terminal. But, the script will still be able to access it.

  

So, now, when anyone clones the repo, and `npm install`, they will be able to run everything.