---
title: Build A Blog Platform With MongoDB and Express
categories:
- Frontend
- Web
tags:
- dev
- node
- database
description: use the MEAN stack to implement a blog platform
date: 2014-10-25
author_profile: true
classes: wide
---

I've been spending some time on this [mean-demo](https://github.com/fengshuo/mean-demo) to understand what it is like using full-stack javascript, although there is no AngularJS(in MongoDB, Express, Angualr, Node) flavor in this demo yet, once the API is ready, the angular part shouldn't be difficult to tackle with.

There are also some document [here](https://github.com/fengshuo/mean-demo/wiki) for the initial version of this demo where I used the native mongodb node.js driver and didn't integrate with Passport.js.


# Overview

## Project Structure

```raw
-> server.js  //this where the main entry point for the app
-> routes/ //most of our code will be in this folder, it will be used for handling request and response
-> views/ //all the templates
-> public/ //this is what will be presented to users with those rendered templates
-> models/ //save all the 'data' stuff here, be it mongoose schema or mongo-node prototype/methods
-> config/ //this is the place we place configuration files, such as the database settings, passport settings, it is not mandotory, but it is eaiser to orignize those 'configuration' files in one folder
-> node_modules/
```

# Database

MongoDB is definitely another topic to discuss, for the brevity of this post, I'll assume that you've already have MongoDB properly installed, and you've started the connection here.

In the initial version I was using the native MongoDB Node.js driver, it worked fine until I tried to integrate with Passport.js like this:

```js
var passport = require('passport');
var LocalStrategy = require('passport-local').Strategy;
//var user = require('./models/user.js');

passport.use(new LocalStrategy({
    passwordField:'user-pw'
},function(username, password, done){
    if(username == 'user4' && password=='000'){
        console.log("verified");
        return done(null,{username:'user4'})
    }else{
        console.log("oppsss....");
        return done(null,false);
    }
}));

passport.serializeUser(function(user,done){
    done(null,user.username);
});

passport.deserializeUser(function(username,done){
    done(null,{username:username});
});

module.exports = passport;
```

The error message is *db connection already exists*, so I checked my db files, taking user.js as an example:

```js
var mongodb = require('./db');

function User(user){
    this.name = user.name;
    this.password = user.password;
    this.email = user.email;
};

module.exports = User;

User.prototype.save = function(callback){
    var user = {
        name: this.name,
        password: this.password,
        email: this.email
    };
    mongodb.open(function(err,db){
        if(err){
            return callback(err);
        }
        db.collection('users', function(err,collection){
            if(err){
                mongodb.close();
                return callback(err);
            }
            collection.insert(user,{safe:true},function(err,user){
                mongodb.close();
                if(err){
                    return callback(err);
                }
                callback(null, user[0]);//success?
            });
        });
    });
};

User.get = function(name,callback){
    mongodb.open(function(err,db){
        if(err){
            return callback(err);
        }
        db.collection('users',function(err,collection){
            if(err){
                mongodb.close();
                return callback(err);
            }
            collection.findOne({
                name: name
            },function(err,user){
                mongodb.close();
                if(err){
                    return callback(err);
                }
                callback(null, user);
            })
        });
    });
};
```

As you can see, for each data-manipulating method, I would open the mongodb at first, then close the db after query or save. In passport.js, It seems that I was querying and serializing some user data, and the db didn't have the chance to close before the next open, because when I remove either the serialize code or the query code, the db is already in use error is gone. Go back to user.js, It seems that whenever I open and close db, the `mongodb.close()` is in the callback function. So I would say that some callback functions were not called before the code went to implementing the next method.

After googling for a while, it seems that the way of opening and closing db frequently is not quite ideal/convenient, so I switched to mongoose(will address this later).

# Server.js

If you use the `express-generator`, you'll see the boilerplate of this entry point file looks similar to this:

```js
var express = require('express');
var path = require('path');
var favicon = require('static-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var routes = require('./routes/index');
var users = require('./routes/users');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

app.use(favicon());
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded());
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use('/', routes);
app.use('/users', users);

/// catch 404 and forward to error handler
app.use(function(req, res, next) {
    var err = new Error('Not Found');
    err.status = 404;
    next(err);
});

/// error handlers

// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
    app.use(function(err, req, res, next) {
        res.status(err.status || 500);
        res.render('error', {
            message: err.message,
            error: err
        });
    });
}

// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
        message: err.message,
        error: {}
    });
});


module.exports = app;
```

Basically, what we do in this file including starting the app with `var app = express();`, making sure those essential middlewares are used, such as the routers, bodyParser, also setup things like the view engine, views, port, in addition to the express generator app.js, we also need to connect to the database at the beginning.


# Users Management

As far as I'm concerned, user management is always interesting, there are many ways of dealing user authentication, the bottom line is, find the one fits your requirement best. The very basic implementation would be asking user to provide information of username, email, password etc and just save them into db, and use one of those fields as the unique identifier for querying. However, if the authentication system supports OAuth verification(with passport.js), you need to consider how to save the user data. Initially I tried to save things like in case I need to associate social media accounts:

```raw
{
	'_id':XXXXX,
	local:{
		name:xxx,
		email:xxx,
	},
	github:{
    	id:xxx,
		name:xxx,
		email:xxx,
		otherinfo:xxx
	},
	facebook:{
		id:xxx,
		name:xxx,
		email:xxx,
		otherinfo:xxx
	}

}
```

But it turned out I don't need to associate accounts after all, then this structure gave me a hard time for trying to save user info so that I can easily query them later. So I changed the structure as a unified user Schema:

```raw
{
	_id:xxx,
	username:xxx
	email:xxx,
	pw:xxx,
	avatar:xxx,(often appears in oauth info)
	location:xxx(often appears in oauth info)

}
```

That means if there is some extra info provided, I'll save them into DB, else just leave them blank. Some may say, this kind of thing is much easier to understand or implement in sql database. But generally I'd like to stick to mongodb this time and not mixing sql and nosql for now.


# Session and Flash

Before I get to know the web dev framework, I didn't realize how essential the session could be. It is quite useful to keep one user logged in between pages, so we can use this piece of information to show related information or keep track of the user somehow. Beyond that, there are many package built on 'session', such as the `connect-flash`, it brings some information to the redirected url.

# File Upload

File upload is also very interesting, because you have to consider your business logic/requirement before decide which plan to use. Usually people would save them into some folders on the server, and distribute those files into different servers depends on the amount of those files. Additionally, you can use cloud services like Amazon cloudfront to either transfer file on DB to cloud or transfer files directly.

Besides, there used to be a file upload part in Express 3.x, but it was removed in Express 4.x, but there are some nice middlewares such as formiddable or busboy to do this job well.

# Deploy

Once you finished local development, you may want to deploy your project. The traditional way is to install node and mongo on Ubuntu or CentOS(Ubuntu is better I think), and scp the project file into your server. Honestly, I think if you don't want to spend too much time on those OP stuff, There is nothing wrong using cloud hosting platforms like `MongoLab` or `Heroku`, they are much easier to setup and all provide some kind of free trial storage.

# Summary

I've learned a lot from this little demo project(although I haven't spend time on the front end code yet). But still a long way to go to make it a 'real' demo.
