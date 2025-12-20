---
title: Play with the MEAN Stack
categories:
- Frontend
- Web
tags:
- dev
- node
- fullStack
description: use the MEAN stack to implement a blog platform
date: 2014-09-30
author_profile: true
classes: wide
---

I've been wanting to try the MEAN stack for quite a while, but it was a little bit intimidting back then when I had nearly zero experience with Mongo, Express, Node, Angular, but heard people talk about them constantly.

Although right now I am still by no means any kind of expert in those things, I have to say this MEAN stack is fascinating since you will be dealing with code from backend to frontend all in javascript (yay).

There is a [detailed wiki](https://github.com/fengshuo/mean-demo/wiki) about this experiment, the note here is more like a introduction.


# Intro

I am going to use express, which is a popular nodeJS web framework, to provide API to the front end. The express framework will communicate with the database(MongoDB) to provide and save data comes from the front end. The front end will be all AngularJS, which is also a MVW front-end framework.

So make sure you have Node and MongoDB properly installed on your laptop first.


# Part 1 - Build With the Framework
---

In reality, the order of build front end first or back end first varies based on your team, Here I would like to build up the backend part first.

## The Entry Point

Normally there would be an entry point js file for the whole project, let's name it  `app.js`.
This file will contain some basic configuration for the app, including the template engine, dev environment settings, etc. Just for the record, you can use the `express-generator` to set up an express app scaffold. But I have some modifications so let's see what we have in this file:

```js
 /**
 * module dependencies
 * @type {exports}
 */
var http = require('http'),
    fs = require('fs'),
    express = require('express'),
    //some modules were removed from express 4, so we need to install them here.
    errorHandler = require('errorhandler'),
    bodyParser = require('body-parser'),
    methodOverride = require('method-override'),
	 //we will be using mongoose to manipulate data in MongoDB
    mongoose = require('mongoose'),
    //use handlebars as template engine
    //also set defaultLayout with 'base'
    handlebars = require('express3-handlebars').create({defaultLayout: 'base'}),
    //handle all routes in /routes/index.js, the file in route would normally where the backend codes are in
    routes = require('./routes');

var app = module.exports = express();

/**
 * app configuration
 */

//select the template mechanism you want to use
//the templates are located by default in /views/*handlebars
//it will be like app.set('view engine', 'jade') if you are using jade
//Tip: you can choose either template engine. You may not be used to simplicity of Jade at first
//so we'll be using handlebars, but remember its default injector is conflict with default AngularJS injector

//set the listening port
app.set('port', process.env.PORT || 3000);
//specify where the views are
app.set('views', __dirname+'/views');

//some setting on handlebars, it would be simpler if you are using Jade which is native to express
app.set('view engine', 'handlebars');
app.engine('handlebars', handlebars.engine);

//serve static content from the public directory
//everything in this folder would be in 'front end', which in here is our AngularJS part
app.use(express.static(__dirname + '/public'));
//use bodyParser to handle POST request content
//make sure you are following the latest bodyParser usage
app.use(bodyParser.urlencoded({'extend':'true'}));
app.use(bodyParser.json());
app.use(methodOverride());

var env = process.env.NODE_ENV || 'development';

if(env === 'development'){
    app.use(errorHandler());
}

if(env === 'production'){
    //based on production env settings
}

/**
 * Middle Wares
 */

//write all route functions in /routes/index.js
routes(app);


app.listen(app.get('port'), function(){
    console.log("server started at "+ app.get('port'))
});
```


## The Folder Structure

Based on the settings in app.js, you should have a project looks like this:

->node_modules
->public
->routes
->views
->app.js
->package.json

Let's create another folder `models` for our data, and a `lib` folder for any custom node modules.
and create a `index.js` in the `routes` folder since we'll be using it soon.


## The Handy Tools
If you've written node apps before, you know that if the app went wrong or you modify your files, you'll need to restart the app again and again, fortunately there are many tools available to solve this problem.

I am using gulp with `gulp-nodemon` and `gulp-livereload` so that if I change any files in the project, the application would restart itself and also automatically refresh the browser window.

```js
 var gulp = require('gulp'),
    less = require('gulp-less'),
    livereload = require('gulp-livereload')(),
    nodemon = require('gulp-nodemon');

//I also added another css-related task for compiling less to css
//If you don't need this, just omit it then

var cssing = function(){
    gulp.src('./public/css/*.less')
        .pipe(less())
        .pipe(gulp.dest('./public/css'))
};

gulp.task('restart',function(){
    nodemon({
        script: 'app.js',
        ext:'html css js handlebars less'
    }).on('restart',function(){
            cssing();
				setTimeout(function(){
                livereload.changed();
            },1000)
            //added a 1000ms time gap here since the restart may take one or two seconds
        });
});

gulp.task('default',['restart']);
```

##Create views

In app.js, there is one line `handlebars = require('express3-handlebars').create({defaultLayout: 'base'})`, that requires us to create a folder named `layouts` in `views` folder, to put the base.handlebars file, I won't go to the details of handlebars here, let's just say we have the `index.handlebars`, `about.handlebars`, `404.handlebars`, `500.handlebars` files for starters.

## Create routes

In the `routes/index.js` file, I am going to set up some basic routes:

```js
//var api = require('./api');
var TODO = require('../models/todo.js').todos;
var User = require('../models/todo.js').user;

module.exports = function(app){
    app.get('/', function(req,res){
        res.render('index',{pagetitle:'homepage'});
        //render '/' with the index.handlebars page
    });
    app.get('/about', function(req,res){
        res.render('about',{pagetitle:'aboutpage'});
    });
    //404 and 500 are like the 'deal-all' situation for middlewares
    app.use(function(req,res,next){
        res.status(404);
        res.render('404');
    });
    app.use(function(req,res,next){
        console.error(err.stack);
        res.status(500);
        res.render('500')
    })
};
```


# Part 2 - Build up the API

Not only for AngularJS, but also for many front end pages, providing a RESTful API means conveniency, so let's plug into the database with mongoose to provide API for AngularJS.

In this specific case, let's create a todo list App (it's kind of cliche, but it really is a good case for beginners I'd say).So we'll be:

- *Get* all todo items
- *POST* new todo item
- *DELETE* one todo item

## MongoDB and Mongoose

To save or edit the data, we need to set up the data first, create a file named `todo.js` in the `models` folder, this will be the place where we define the todo list `Schema`.

Also don't forget to start the mongdoDb first:

open the terminal with: `mongod --dbpath ~/Desktop/todoDB`, you can change the filepath to save data somewhere else;

Now let's see what's in the todo model:
```js
var mongoose = require('mongoose');
//connect to our local db
//if you want to delete the old DB:
//$ use oldeDB
//$ db.dropDatebase()
//would delete the db

mongoose.connect('mongodb://localhost/todoDB');

var Schema = mongoose.Schema;

var todoSchema = new Schema({
    text: String //define our data Schema, let's just put the todo item content for simplicity
});

mongoose.model('TODO', todoSchema);

module.exports = {
    todos:mongoose.model('TODO') //export the data model so we can use it in route
};
```


## App VERBs

Since the data model is ready, let's write the app Verbs, such as 'get' and 'post' in `routes/index.js`:

```js
 //First use the todo model we just defined
 var TODO = require('../models/todo.js').todos;

 //then add the verbs
     app.get('/api/todos', function(req,res){
     //find all results in json
        TODO.find(function(err, todos){
            if(err){
                res.send(err);
            }
            res.json(todos);
        });
    });
    app.post('/api/todos', function(req, res){
    //create new data entry with the new todo item
        TODO.create({
            text: req.body.text,
            done:false
        },function(err, todo){
            if(err){
                res.send(err);
            }
            TODO.find(function(err, todos){
                if(err){
                    res.send(err);
                }
                res.json(todos);
            })
        });
    });
    app.delete('/api/todos/:todo_id', function(req, res){
        TODO.remove({
            _id: req.params.todo_id
        },function(err,todo){
            if(err){
                res.send(err);
            }
            TODO.find(function(err,todos){
                if(err){
                    res.send(err);
                }
                res.json(todos);
            });
        });
    });
```


# Part 3 - Use APIs in FrontEnd with AngularJS

So far we got the API ready to use, now let's focus on the frontend, since AngularJS is definitely another topic worth another post, I would just cover the very basics here just to make our app ready.

Now we should do the work in `public` folder, Let's make sure the project structure looks like this:

->angApp.js //the entry point for AngularJS, named it angApp to differenciate with the Express app.js
->main.js //the main.js would be used with RequireJS, but this time, we don't need to use it.
->css
->img
->js
-->Controllers
-->Directives
-->Filters
-->Services
-->libs

## Get AngularJS Ready to Use

Since we are going to use the index page as the todo app page, let's update the index.handlebars:

```html
<!doctype html>
<html ng-app="myTodoApp">
<!-- means this page will be our angular app -->
<head>
    <meta charset="UTF-8">
    <title>Seed Home Page</title>
    <link rel="stylesheet" href="./css/main.css"/>
    <!--use angularJS-->
    <script src="./js/libs/angular.min.js"></script>
    <script src="./js/angApp.js"></script>
</head>
<body ng-controller="mainController">
<!--This body tag would be the controller scope of mainController-->
<h1>{{pagetitle}}</h1>

<!--the handlebar template will fill the pagetitle
*We need to change the default injector in either handlebars or angular so it won't conflict with each other*
I changed the Angular injector as you'll see later-->

<div class="jumbotron text-center">
    <h1>Todo App <span class="label label-info">{[{ todos.length }]}</span></h1>
</div>
<div id="todo-list">
    <div>

        <!-- Loop over the todos data in $scope.todos with ng-repeat-->
        <divng-repeat="todo in todos">
            <label>
                <input type="checkbox" ng-click="deleteTodo(todo._id)"> {[{ todo.text }]}
            </label>
        </div>

    </div>
    <div id="todo-form">
        <div>
            <form>
                <div class="form-group">
                    <!-- bind the value to scope in mainController -->
                    <input type="text" ng-model="formData.text">
                </div>

                <!-- create new todo item with the method createTodo in scope -->
                <button type="submit" class="btn btn-primary btn-lg" ng-click="createTodo()">Add</button>
            </form>
        </div>
    </div>
</div>
</body>
</html>
```


## angApp.js

Angular is indeed suitable for single page app, so let's see what should we add to the file so the directives in the previous html will work:

```js
 var myTodoApp = angular.module('myTodoApp', []).config(function($interpolateProvider){
    //solve conflict b
    $interpolateProvider.startSymbol('{[{').endSymbol('}]}');
});

//If we use requireJS as mentioned earlier, we can inject the controller or services js  here
//or we should be using angular routers
//but this time let's keep it simple as we can dive into angularJS later

myTodoApp.controller('mainController', function($scope,$http){
    $scope.formData = {};
    //show all todos when landing on the page
    $http.get('/api/todos')
        .success(function(data){
            $scope.todos = data;
        })
        .error(function(err){
            console.log(err)
        });
    //submit new todo item
    $scope.createTodo = function(){
        $http.post('/api/todos', $scope.formData)
            .success(function(data){
                $scope.formData = {}; //clean the ng-model
                $scope.todos = data;
            })
            .error(function(err){
                console.log(err);
            });
    };
    //delete todo apps
    $scope.deleteTodo = function(id){
        $http.delete('/api/todos/' + id)
            .success(function(data){
                $scope.todos = data;
                console.log(data);
            })
            .error(function(err){console.log(err)})
    }
});
```

# Wrap it Up
So the related repo is [here](https://github.com/fengshuo/epsDemo), you can fork it, start the mongodb first and use gulp in the right path to start building.

Now the basic MEAN stack is done, you can add new features or modify, like add user register or login, set due date for the item etc. As I previously mentioned, any item in the MEAN stack, Mongo, Express, Angular, Node can be another big topic, so take as much time as you need to enjoy coding in JS.


## Links

[1](https://egghead.io/articles/new-to-angularjs-start-learning-here)
[2](https://github.com/nswbmw/N-blog)
[3](http://mean.io/#!/)
[4](http://scotch.io/tutorials/javascript/creating-a-single-page-todo-app-with-node-and-angular)
[5](http://scotch.io/tutorials/javascript/node-and-angular-to-do-app-application-organization-and-structure)
[6](http://scotch.io/bar-talk/setting-up-a-mean-stack-single-page-application)
[7](https://thinkster.io/angulartutorial/mean-stack-tutorial/)
[8](http://shop.oreilly.com/product/0636920032977.do)
