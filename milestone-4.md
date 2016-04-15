## Milestone 4 - Creating a User Interface with AngularJS

In this milestone we will shift our focus from the server to the client as we create our user interface.  Using [AngularJS](https://angularjs.org/) and [Bootstrap](http://getbootstrap.com/), we will be creating a simple web interface for our users to see and add new projects as well as upvote and comment on other projects.

> In an effort to limit the opinions in this course we will simply be using CDN's for our vendor files.  There is a bit of a debate going on about using `npm` to manage server side and `bower` for client side vs using just `npm` to manage both and then using something like `webpack` or `browserfiy` to allow for client side modules.  I'm choosing not to pick a side and instead will let you do your own research and come to your own conclusion.

Alright, so to get started, if you recall when we first created our express project we set it up to use the `EJS` template system.  If you look inside the `views` directory you should see two files, `index.ejs` and `error.ejs`.  Now, take a quick peek at `routes/index.js` - in particular this bit of code here:

```
/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});
```

This is what sets up the route for the root diretory (The home page).  This says when the server sees a request to http://localhost:3000/, it's going to render the `index` EJS template.  Behind the scenes, express uses some configurations that were done in `app.js` to find a file called `views/index.ejs`, renders the template, and responds with the HTML output.  You'll notice that by default, the `index.ejs` has some funny looking pieces in it like `<title><%= title %></title>`.  The `<%= ... %>` is a template placeholder, when the view is rendered we can pass in data to the view and any placeholders like this will be replaced with the data we sent in.  Look back at the default route that renders the `index`, notice the `{ title: 'Express' }` part.  Here, we're telling ExpressJS to render the `index.ejs` template and replace any `<%= title %>` with "Express".

For this project though, we don't really need any template placeholders.  In fact, we're going to erase whats in `view/index.ejs` and replace it with this:

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>My Next Project</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7" crossorigin="anonymous">
  </head>
  <body
    ng-app="myNextProjectApp"
    ng-controller="HomeCtrl as vm">

    <h1>{{vm.message}}</h1>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.10/angular.min.js"></script>
    <script src="javascripts/app.js"></script>
    <script src="javascripts/home/home.js"></script>
    <script src="javascripts/home/home.controller.js"></script>
  </body>
</html>
```

Most of this should look familiar, we're simply building the base HTML for your index.  On the `body` tag, we use `ng-app` to initialize our AngularJS module that we will create called `myNextProjectApp` and `ng-controller` to bind the `HomeCtrl` to the `body`.

We keep our `script` tags toward the bottom of the page to ensure that our DOM is fully loaded and ready before loading our JavaScript files.  Next, we need to make the files that we are linking to.

> Note that we are using the "controller as" syntax for this project.  For more information about this syntax, read [Todd Motto's](https://toddmotto.com/digging-into-angulars-controller-as-syntax/) and [John Papa's](http://www.johnpapa.net/angularjss-controller-as-and-the-vm-variable/) posts.

`public/javascripts/app.js`
```
'use strict';

angular.module('myNextProjectApp', []);
```

Here we are defining our angular app `myNextProjectApp`.

> To prevent confusion between the `public/javascripts/app.js` and `/app.js` files, any reference to `app.js` in this milestone will refer to the client side file that contains our Angular application located in `public/javascripts/app.js` unless instructed otherwise.

`public/javascripts/home/home.js`
```
'use strict';
```

Nothing much in this file yet, we'll circle back to this one in a bit.

`public/javascripts/home/home.controller.js`
```
'use strict';

angular
  .module('myNextProjectApp')
  .controller('HomeCtrl', [function() {
    var vm = this;

    vm.message = 'Hello World!';
  }]);
```

And here we create a simple controller.  If you're not used to seeing the "controller as" syntax, you might be wondering where the `$scope` is.  Using "controller as" means that instead of using `$scope` to pass data back to the view, anything that needs to be exposed to the view would go in `this`.  Here's where JavaScript gets tricky, `this` can be different based on it's context.  For example:

```
angular.module('myApp').controller('ThingCtrl', function() {
  // `this` is in the context of the controller
  this.name = 'My Thing';

  $http.get('/api/some/thing').success(function(thing) {
    // Be careful!  `this` here is in a different context
    // so the `thing` property will not be added to the
    // controller's context
    this.thing = thing;
  });
});
```

To help alleviate the context issues, `this` is put inside a variable.  A common convention is to use `vm` for the variable which represents the `ViewModel` portion of the (M)odel, (V)iew, (V)iew(M)odel.

If you safe your files, start your server with `nodemon npm start`, and navigate to http://localhost:3000/ you should see "Hello World!" outputted to the screen.

### Don't Repeat Yourself

Thinking about what our app is going to do, we're going to have to display a list of projects and eventually a list of comments.  AngularJS provides a helpful directive called `ng-repeat` which we can use to iterate over an array of things and display them to the user.  To see what I'm talking about, let's update the `home.controller.js` file by adding this after the "Hello World" line:

```
vm.projects = [{
  name: 'Project 1',
  upvotes: 5
},{
  name: 'Project 2',
  upvotes: 7
},{
  name: 'Project 3',
  upvotes: 3
},{
  name: 'Project 4',
  upvotes: 8
},{
  name: 'Project 5',
  upvotes: 2
}];
```

And in the `index.ejs` file, after the line with the `h1` add this:

```
<div ng-repeat="project in vm.projects | orderBy:'-upvotes'">
  {{project.name}} - Upvotes: {{project.upvotes}}
</div>
```

Refresh your browser, you should see the same "Hello World" header but now you should also see 5 new lines each printing out a project in the projects list.  Using the `orderBy` filter, we can sort our results in descending order by `upvotes`.
