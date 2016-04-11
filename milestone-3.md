## Milestone 3 - Adding Controllers and Routes

In this milestone, build up our controllers and add express routes so our API will actually do something.  In the previous milestone we created data models, this handles the CRUD logic of us.  Think of the controller as the glue between the view (or in our case routes) and the models.

### Project Controller

Let's start off with the project controller, create a new file in `api/project/` named `project.controller.js`.  Inside the new file, add the following:

```
var Project = require('./project.model');

// Get all projects
exports.index = function(req, res) {
  Project.find(function(err, projects) {
    if (err) return res.send(500, err);
    return res.status(200).json(projects);
  });
};
```

We're creating a simple controller here which exports a single method that will be used by the routes we'll set up in just a bit.  We import the project model which will be used to interact with the database.  The method is called `index` and is used to retrieve all of the projects in the database.  Using `Product`, which is the model (A mongoose schema model o be exact), we call `.find` which will return all objects in the project collection.  We check for an error, which would be in `err`, if there was an error we respond with a `500` status code.  Otherwise, we return a `200` status code and a JSON payload with the array of projects.

### Routing Requests

We will be adding more methods later on, for now though, let's set up a route that uses the controller so we can test it.  Create a new file in the same directory, `api/project/index.js`.  This is where you'll tell the Express router what controller methods to use for what routes.  Type the following into your new file:

```
var express = require('express');
var controller = require('./project.controller');

var router = express.Router();

router.get('/', controller.index);

module.exports = router;
```

First you're going to be using the Express router, so you need to import it with `require('express')`.  Then you import the controller you just made, this gives you access to the `index` method by using `controller.index`.  Next, you create a `router` variable and define a `get` route.  Here's where things might start to get a little confusing.  We're defining a get route of `/`, this doesn't mean http://localhost/.  In a minute, we'll be importing our routes and when we do we'll be telling the app that when the route `/api/projects` is detected, use our project routes.  So our first route here, for `index` will be executed when the route is `/api/projects/`.  A little confusing, I know, but it should make more sense in a minute.  Anyways, when the route is detected it will execute `controller.index` which will return all the projects in the collection.

Let's bring it all back into focus now, open up the `/app.js` file.  Around line 29 or 30, you should see these two lines:

```
app.use('/', routes);
app.use('/users', users);
```

After `app.use('/users', users);`, add this:

```
app.use('/api/projects', require('./api/project'));
```

This is where we tell the app that any time it sees the URL `/api/projects` it's going to use the project routes we defined.  The routes that we defined for the project will all be relative to the `/api/projects` URL.  You only have to add this one time per model, any new routes will be added to the data model's `index.js`.

### Testing the Route

If all went well, you should be able to fire up your server with `npm start`.  Now it's time to do a quick test, you have a couple of options here.  You can test your routes from your terminal using cURL calls or if you prefer a more graphical approach you can check out [Postman](https://www.getpostman.com/).  For this course, the examples will be run from the terminal.  With your app running, open up a new tab or window in your terminal and type the following:

```
curl http://localhost:3000/api/projects
```

If it worked, then it will be a bit boring as it will return an empty array.  That makes sense, we haven't added any projects yet.  So let's take care of that next.

### Creating Data

Up to this point, you've successfully created a server with a RESTful interface that allows you to retrieve a list of projects in the database.  That doesn't do you much good if you don't have a way to create new projects in the database.  Open up `api/project/project.controller.js` and at the bottom of the file, add this new method:

```
// Creates a new project
exports.create = function(req, res) {
  Project.create(req.body, function(err, project) {
    if(err) return res.send(500, err);
    return res.status(201).json(project);
  });
};
```

This will create a new method called `create`, inside it we use the mongoose model `.create` method to add the project to the database.  If an error is encountered, we respond with a `500` status code, otherwise we respond with `201` and a JSON payload of the new project.  For more information what the difference between 500, 200, and 201 status codes are, head over to the [W3 Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

Now that we have a new method to use, let's wire it up to a specific route.  Open `api/project/index.js` and after the line where you defined the `router.get('/', ...)`, add this line:

```
router.post('/', controller.create);
```

Now when a POST request is sent to `/api/project/`, the `controller.create` method will be called.  Let's test it out, restart your server and type the following in a new terminal tab/window (Make sure you substitute PROJECT_NAME, PROJECT_DESCRIPTION, and PROJECT_CREATOR with your own text):

```
curl --data 'name=PROJECT_NAME&description=PROJECT_DESCRIPTION&creator=PROJECT_CREATOR' http://localhost:3000/api/projects
```

In response, you should see a JSON object was returned with the information you provided it.  Now if you do the same GET request you did earlier, you'll see the newly created project:

```
curl http://localhost:3000/api/projects
```

```
/* Example Response */
[
  {
    "_id": "570bf5e1917b3688621e18f1",
    "name": "Bunk Beds",
    "description": "Make new bunk beds for my daughters",
    "creator": "Trevor",
    "__v": 0,
    "comments": [],
    "upvotes": 0
  }
]
```

Take a quick moment to look at the `_id` field.  This wasn't anything you modeled in the schema, this is a primary identifier that Mongo adds to every single object that is created.  Each `_id` is unique across the entire database and it's how we look up various objects.  In future examples, I'll be performing cURL calls using `_id` values, you will have to substitute the value with an `_id` in your own collection otherwise you'll end up with 404 errors.

For the rest of the routes/methods that we'll be creating, we're going to be using route parameters.  Essentially, we'll be passing some sort of ID into the URL which will be stored in a route parameter which we can use to look the object up in the database.  Instead of constantly having to rewrite the same code to look up the route parameter for each method, we can write what's called middleware.  As the name implies, middleware sits in between the route and the method that gets executed.  So, when a GET request comes in to `/api/projects` we can pass that request through the middleware which will pre-populate the request with the project if necessary, this then get's passed to `controller.index` to be processed and send back a response.

Open up `api/project/project.controller.js` and before you define `exports.index`, add the following:

```
// Populate a project from a route parameter
exports.populate = function(req, res, next, id) {
  Project.findById(id).exec(function(err, project) {
    if (err) return next(err);
    if (!project) {
      err = new Error('Cannot find project');
      err.status = 404;
      return next(err);
    }
    req.project = project;
    return next();
  });
};
```

This creates a method called `populate` which will be the middleware that will populate a project from an ID in the route parameter.  It uses `.findById` to lookup an object by the ID and `.exec` will execute the query.  If there is an error, we call `next` which advances the request to the next step (most likely the controller method).  If we couldn't find the project, we call `next` with an error indicating so.  And finally, if we found the project we populate `req.project` with the project object so it will be available to our controller methods.

To use the middleware, we need to open up `api/project/index.js` and before `router.get('/', controller.index);` add the following:

```
// Project Middleware
router.param('project', controller.populate);
```

Now, whenever we define a route that has `:project` in it the middleware will automatically populate the object for us.  Alright, so there are a couple more routes to make.  We can get a list of projects and create a new project, but we need a way to look at a single project.  Open up `api/project/project.controller.js` and add the following method to the end of the file:

```
// Retrieve a single project
exports.get = function(req, res) {
  return res.status(200).json(req.project);
};
```

A simple little method, thanks to the middleware we can be sure that `req.project` will have the project object in it so we can simply respond with that.  Let's wire up the route, open `api/project/index.js` and after `router.post('/', controller.create);` add this:

```
router.get('/:project', controller.get);
```

Restart your server and type this in to the terminal:

```
curl http://localhost:3000/api/projects/570bf5e1917b3688621e18f1
```

```
/* Example Response */
{
  "_id": "570bf5e1917b3688621e18f1",
  "name": "Bunk Beds",
  "description": "Make new bunk beds for my daughters",
  "creator": "Trevor",
  "__v": 0,
  "comments": [],
  "upvotes": 0
}
```

> Remember to replace `570bf5e1917b3688621e18f1` with the ID that is in your database, otherwise you'll end up getting a 404 error

### Leveraging Schema methods

Until now, you've been using core Mongoose Schema methods, such as `find`, `save`, and `findById`.  Now, we're going to create a quick method that will be used to add an upvote to a project.  Open up `api/project/project.model.js` and before `module.exports = ...`, add this new method:

```
ProjectSchema.methods.upvote = function(cb) {
  this.upvotes++;
  this.save(cb);
};
```

Simple enough, we're adding an `upvote` method to the `ProjectSchema.methods`.  It takes a single argument, `cb` which is short for `callback`.  For more info on what a callback is, check out [This Post](http://javascriptissexy.com/understand-javascript-callback-functions-and-use-them/).  Inside the method, we increment `upvotes` and then call the core `save` method, passing in the `cb` function.  That's it, now we can create a controller method that uses this new schema method.  Navigate over to `api/project/project.controller.js` and add this to the end of the file:

```
// Add an upvote a project
exports.upvote = function(req, res) {
  req.project.upvote(function(err, project) {
    if (err) return res.send(500, err);
    return res.status(200).json(project);
  });
};
```

Once again, thanks to the middleware `req.project` is pre-populated so we can immediately call the new `upvote` method, we're passing in an anonymous function which is what gets stored in `cb`.  When the `save` is complete, we either respond with an error or the updated JSON object for the project.  Ok, you should start seeing a pattern here.  We create a controller method, then it's time to wire it to a route.  So open up `api/project/index.js` and add this new route:

```
router.put('/:project/upvote', controller.upvote);
```

For semantic reasons, we're using a PUT request because we are putting a new value in place.  

> For more info on when to use what types of requests, head over to the [REST API Tutorial](http://www.restapitutorial.com/lessons/httpmethods.html).  

To test this, restart your server and send this cURL request:

```
curl -X PUT http://localhost:3000/api/projects/570bf5e1917b3688621e18f1/upvote
```

```
/* Example Response */
{
  "_id": "570bf5e1917b3688621e18f1",
  "name": "Bunk Beds",
  "description": "Make new bunk beds for my daughters",
  "creator": "Trevor",
  "__v": 0,
  "comments": [],
  "upvotes": 1
}
```

### Add Some Comments

Now that we've finished up adding all the necessary routes to our projects.  It's time to add some that are focused on project comments.  We'll be working in `api/project/project.controller.js`, create a new `createComment` method:

```
// Creates a new comment on a project
exports.createComment = function(req, res) {
  // Add the project to the request body
  req.body.project = req.project;

  // Save the comment
  Comment.create(req.body, function(err, comment) {
    if (err) return res.send(500, err);

    // Add the comment to the project
    req.project.comments.push(comment);

    // Save the project
    req.project.save(function(err, project) {
      if (err) return res.send(500, err);
      return res.status(200).json(comment);
    });
  });
};
```

The first thing we do is add the project to the `req.body`, which is the data for the comment.  Then we create a new comment using the Comment model, when that returns we add the new comment to the project.  Finally save the project and return the comment object.  Because we're using the Comment model, we're going to have to import it into our controller.  At the top of the controller, after the line that imports the Project model, add this:

```
var Comment = require('../comment/comment.model');
```

Once more, we need to create a route.  In `api/project/index.js`, add the following route:

```
router.post('/:project/comments', controller.createComment);
```

Restart your server and test out the new route:

```
curl --data 'text=I really like this project idea!&creator=Kelly' http://localhost:3000/api/projects/570bf5e1917b3688621e18f1/comments
```

```
/* Example Response */
{
  "__v": 0,
  "text": "I really like this project idea!",
  "creator": "Kelly",
  "project": {
    "_id": "570bf5e1917b3688621e18f1",
    "name": "Bunk Beds",
    "description": "Make new bunk beds for my daughters",
    "creator": "Trevor",
    "__v": 1,
    "comments": [
      "570c0bc0e248b30967daf981"
    ],
    "upvotes": 1
  },
  "_id": "570c0bc0e248b30967daf981",
  "upvotes": 0
}
```

### Upvoting Comments

We just have one more route to create before we're done with our API.  But before we do that, we'll need to create some middleware for comments that does the same auto-populating as the project middleware.  For this, you'll need to create a new file: `api/comment/comment.controller.js` with the following code:

```
var Comment = require('../comment/comment.model');

// Populate a comment from a route parameter
exports.populate = function(req, res, next, id) {
  Comment.findById(id).exec(function(err, comment) {
    if (err) return next(err);
    if (!comment) {
      err = new Error('Cannot find comment');
      err.status = 404;
      return next(err);
    }
    req.comment = comment;
    return next();
  });
};
```

This is nearly identical to the project middleware, the only difference being that we're using the Comment model and the variables say `comment` instead of `project`.  Now that we've go the middleware, let's wire it up to the route param.  Open `api/project/index.js`, on the line after you import the project controller add this:

```
var comment = require('../comment/comment.controller');
```

And, on the line after you add the project middle ware add this:

```
// Comment Middleware
router.param('comment', comment.populate);
```

Next up, we'll add an `upvote` method to the Comment model just like we did for the Project model.  Before the `module.exports = ...` in `api/comment/comment.model.js`, add the following method:

```
CommentSchema.methods.upvote = function(cb) {
  this.upvotes++;
  this.save(cb);
};
```

Now we can call this method from the project controller, so open up `api/comment/comment.controller.js` and add this new method at the end of the file:

```
// Add an upvote to a comment
exports.upvote = function(req, res) {
  req.comment.upvote(function(err, comment){
    if (err) { return next(err); }
    res.status(200).json(comment);
  });
}
```

And now we can wire up the route.  We'll have to create a new file, `api/comment/index.js` with the following code inside:

```
var express = require('express');
var controller = require('./comment.controller');

var router = express.Router();

// Comment Middleware
router.param('comment', controller.populate);

router.put('/:comment/upvote', controller.upvote);

module.exports = router;
```

And last but not least, we need to tell our app how to use our new routes.  Open up `app.js` and right after the line where you added the project routes, add this:

```
app.use('/api/comments', require('./api/comment'));
```

Phew!  That was a lot, let's cross our fingers and restart the server.  As long as we didn't screw something up along the way, your server should be running and we can test our last route with this cURL call:

```
curl -X PUT http://localhost:3000/api/comments/570c0bc0e248b30967daf981/upvote
```

```
/* Example Response */
{
  "_id": "570c0bc0e248b30967daf981",
  "text": "I really like this project idea!",
  "creator": "Kelly",
  "project": "570bf5e1917b3688621e18f1",
  "__v": 0,
  "upvotes": 1
}
```

### Populating Nested Collections

If you recall, earlier we talked about setting up our Project schema model so it contained a collection of comments.  The reason we did this was so we could leverage Mongoose's built in `populate` method which can automatically expand all the comments in the collection.  To see what I mean, try running this cURL call:

```
curl http://localhost:3000/api/projects/570bf5e1917b3688621e18f1
```

```
/* Example Response */
{
  "_id": "570bf5e1917b3688621e18f1",
  "name": "Bunk Beds",
  "description": "Make new bunk beds for my daughters",
  "creator": "Trevor",
  "__v": 1,
  "comments": [
    "570c0bc0e248b30967daf981"
  ],
  "upvotes": 1
}
```

Notice how the `comments` is just an array, in this case with a single comment ID in it.  That's not very helpful, that would mean we'd have to make a second call to look up those comments and map them to their IDs.  Luckily, Mongoose gives us the ability to automatically expand those objects.  To do this, we'll need to update the project controller's `get` method, so open up `api/project/project.controller.js` and update the `get` method to look like this:

```
// Retrieve a single project
exports.get = function(req, res) {
  req.project.populate('comments', function(err, project) {
    if (err) return res.send(500, err);
    return res.status(200).json(project);
  });
};
```

Now, you're using the `populate` function which takes two arguments.  The first is the field you want to expand, in this case, `comments` and the second is our callback.  If all went well, we'll return a JSON array of our projects with any comments expanded for us.  let's retry the cURL command from above and you should see something like this:

```
{
  "_id": "570bf5e1917b3688621e18f1",
  "name": "Bunk Beds",
  "description": "Make new bunk beds for my daughters",
  "creator": "Trevor",
  "__v": 1,
  "comments": [
    {
      "_id": "570c0bc0e248b30967daf981",
      "text": "I really like this project idea!",
      "creator": "Kelly",
      "project": "570bf5e1917b3688621e18f1",
      "__v": 0,
      "upvotes": 1
    }
  ],
  "upvotes": 1
}
```

Not that we don't do this when looking up all the projects, this would probably cause a performance hit when pulling down larger datasets.  The `populate` method would be more idea for looking up single entities.  

### Conclusion

And that's it.  I know that was a long milestone, but if you got here then you should have a fully functioning RESTful API.  Kudos!  From here, we'll move our attention over to the client side and start setting up our AngularJS app to utilize our fresh new API endpoints.

> See example code for this milestone in this [branch](https://github.com/TNowalk/my-next-project/tree/milestone-3)
