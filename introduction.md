## Introduction

The goal of this course is to introduce you to the MEAN stack by creating a simple Product Hunt clone.  At the end of this course you will have gained an understanding of the MEAN stack and learned how to build a RESTful API as well as a frontend to perform basic CRUD operations against a database.

### What is a MEAN stack?

Let's start off with defining what a stack is.  In our industry, the stack, or sometimes called tech stack, generally refers to the underlying technology in your project.  The MEAN stack is a full-stack, which means that it includes both server and client side technologies.  

There are four main technologies that make up the MEAN stack:

* [MongoDB](https://www.mongodb.org/) - An open source data store which uses a "Document Oriented Design".  Unlike relational databases where you store data in tables, columns, and rows, you essentially store JSON objects.  If you've used JavaScript, then you've likely encountered JSON objects which makes using MongoDB familiar.
* [ExpressJS](http://expressjs.com/) - A framework that sits "on top" of NodeJS which provides you with an [MVC architecture](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller).  With ExpressJS, a lot of the grunt work is abstracted away from you and gives you easy ways to manage routes, requests, and serving views (among other things).
* [AngularJS](https://angularjs.org/) - A client side JavaScript framework which simplifies the development of advanced, dynamic applications.  Popular for it's data-binding and MVC structure.
* [NodeJS](https://nodejs.org/en/) - An open source server side runtime environment that is built on the shoulders of Google Chrome's V8 engine.

The beauty (and appeal) of the MEAN stack is that nearly all the underlying tech is JavaScript.  This makes the transition between the client and server side is relatively easier as both sides of the stack share the same language.

### Prerequisites

This course will work with some advanced concepts, therefore there is an assumption that you have at least some basic experience with JavaScript.  It will also be a good idea for you to have a general understanding of [RESTful APIs](https://en.wikipedia.org/wiki/Representational_state_transfer), [MVC architecture](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), and version control such as [GitHub](https://github.com/).

Once last note about the source code included in this project.  The finished project can be found on my GitHub: <https://github.com/TNowalk/my-next-project>.  At the end of each Milestone, there will be a link to a branch with the project code up to that point.  The branch names will line up with the milestone, for example to get the code for Milestone 1 you would run `git checkout milestone-1`.  In an effort to reduce the number of files under version control and decreasing the download times, `node_modules` and `bower_components` have been ignored.  So when you switch to a new branch, you'll need to run `npm install` and if when you get to the client side milestones, `bower install`.
