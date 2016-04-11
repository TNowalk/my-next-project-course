## Milestone 1 - Initialize project

Before we get started, you'll need to make sure that you have `NodeJS` and `MongoDB` installed.

> Install [NodeJS](https://nodejs.org/en/download/package-manager/) - I personally installed NodeJS via `homebrew` on my Mac.  Find the appropriate OS on the [install instructions](https://nodejs.org/en/download/package-manager/)  page on the NodeJS website.  

Installing `NodeJS` gives you access to [NPM](https://www.npmjs.com/) which is a package manager for JavaScript.  We'll be using `npm` throughout the course to install server side dependencies which will be used in our project.

> Install [MongoDB](https://docs.mongodb.org/manual/installation/) - Choose the appropriate installation based on your OS on their [installation page](https://docs.mongodb.org/manual/installation/).

### Project Overview

Let's take a minute and outline exactly what we'll be building.  Here's a quick bullet list of the main goals of we want users to be able to do in our app:

* Create new project ideas
* See all project ideas
* Upvote a project
* Add a comment to a project
* Upvote a comment on a project

> A quick note, there are many workflows when it comes to creating a new project.  My favorite phrase is, "Ask 10 developers a question and you'll get 10 different answers."  There are some folks that like to dive in and develop the UI first, then build the backend.  I personally like to start with the backend (API) and make sure all my endpoints are working.  Then, I move to the UI and build my app around the data.  This tutorial will follow the latter workflow and we'll be building the API first.  With that said, when making your own projects feel free to use whatever workflow makes sense to you.

### Version Control

In any type of development, version control is crucial.  It's a system that keeps track of changes to your files.  This is helpful for working on multiple environments, working with other individuals, and in the unfortunate event of a mishap such as dropping your laptop.  This tutorial will assume that you're using `git` and GitHub for your version control.

Head over the [GitHub](https://github.com/) and create a new repository for the project.  Make sure when you create the repo, you choose not to include a `README`, `LICENSE`, or `gitignore` files.  We'll add those towards the end of the project.

### Setting Up

Alright, so let's get started.  In an effort to save some time, we'll be using the `express-generator`.  Generators basically scaffold out a project, including folder structure, common files, and various libraries.

If you are interested in learning how to create your own NodeJS server with Express from scratch, checkout the [ExpressJS Hello World Example](http://expressjs.com/en/starter/hello-world.html).

To use the `express-generator` you'll have to install it:

```
npm install -g express-generator
```

Once installed globally (That's what the `-g` means), you can use the `express` command to generate a new project scaffolding.

```
express --ejs my-next-project
```

This will create the scaffolding in a directory called `my-next-project`.  You can `cd` into that directory and checkout the default files that it creates.  Also note that the `express` generator supports a few different template engines, such as [Jade](http://jade-lang.com/) and [Handlebars](http://handlebarsjs.com/).  For our purposes, we'll be using HTML and so the [Embedded JavaScript](http://www.embeddedjs.com/) engine makes the most sense which we set using the `--ejs` flag.

Before we can attempt to run our app, we need to install the default dependencies that are included in the generator.  To see the list of dependencies, check out the `dependencies` property in the `package.json` file.  To install the modules, we'll be using `npm`.

```
npm install
```

You should now have a new directory called `node_modules`, if you take a peek inside there you'll see a bunch of modules which will be used throughout your project.  While we may not be directly using every single module in our project, the modules we will be using such as `express`, `mongoose`, and `passport` which depend on other modules.  This is where `npm` stands tall, when you require a depepency in your `package.json` it will install that module as well as any of it's dependencies so you don't have to worry about gathering all of those modules manually.

The generator gives us a good starting point, but there are still some packages that we'll need to install manually.  The first is [MongooseJS](http://mongoosejs.com/) which is an object modeling tool for interacting with a MongoDB.  We'll dive more into what Models and Schemas are in our next Milestone.  For now though, let's install the module.  This time though, we're going to use the `--save` flag which will add `mongoose` to the `package.json` as a dependency and be automatically installed for anyone who downloads your app and runs `npm install`.

```
npm install --save mongoose
```

To test to make sure that everything is working, let's fire up your shiney new server.

```
npm starting
```

With the default settings, you should now be serving your app at <http://localhost:3000>.  If all is working, you should see a "Welcome to Express" message.

One last thing we'll do is add a `.gitignore` file.  This is used to ignore certain files and directories so they don't get added to our version control.  Things like `node_modules` and `bower_components` are files that we don't necessarily want to maintain in our own repo.  It can bloat your repo size which can affect download times when cloning as well as lead to module version inconsistencies.  In general, when you clone the repo the first thing you do is `npm install` and if there are `bower_components` then you'd run `bower install`.  

Create a new file called `.gitignore` and add the following to it:

```
npm-debug.log*
node_modules
bower_components
.DS_Store
```

Now `git` will ignore these files so they won't make it into version control.

### Save Your Progress

That wraps up our first milestone.  Now would be a good time to commit your changes to GitHub.  Earlier you should have created a new repo in GitHub, so all you should have to do is associate this directory with your repo.  For more info on adding an existing project to GitHub, checkout [GitHub's Tutorial](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/).

When looking at your repo on GitHub you should see an text input that contains a url that looks something like: `https://github.com/GITHUB_USERNAME/REPO_NAME.git`.  That is your remote repository URL.  Once you have that, you can perform the following from the root of your project:

```
git init
git remote add origin https://github.com/GITHUB_USERNAME/REPO_NAME.git
```

 To confirm that you correctly added the remote URL you can run `git remote -v` and you should see something like this:

 ```
 $ git remote -v
origin	https://github.com/GITHUB_USERNAME/REPO_NAME.git (fetch)
origin	https://github.com/GITHUB_USERNAME/REPO_NAME.git (push)
```

Once you have your remote set up, you can `add`, `commit`, and `push` your changes up to GitHub.  On your  initial push, you will probably have to set up your upstream with `git push --set-upstream origin master`.  After that, you'll be able to use `git push`.

> See example code for this milestone in this [branch](https://github.com/TNowalk/my-next-project/tree/milestone-1)
