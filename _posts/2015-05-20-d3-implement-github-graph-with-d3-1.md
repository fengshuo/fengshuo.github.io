---
title: Implement Github Graphs with D3.js (Part One)
categories:
- Frontend
- Web
tags:
- d3
- github
- api
- express
- dataVisualization
description: Use express with Github api to implement repository graphs
date: 2015-05-20
author_profile: true
classes: wide
---

As most Github user might notice, there is a [graph page](https://github.com/twbs/bootstrap/graphs/contributors) for each repository, which includes some charts to represents your coding data.

How about using d3.js wrapping with Github API to implement these charts?

# OAuth Authentication

Well, it all starts with reading the [official document](https://developer.github.com/guides/). By and large, if we want to represent more than just public data, it is necessary to be authenticated.

Although it is possible to authenticate by using username and password, it is rarely used in reality, whereas OAuth authentication is commonly used nowadays.
So basically OAuth2 is a protocol that allows external apps request authorization to private details in your Github account without getting your password. All application should be [registered](https://github.com/account/applications/new) before getting started.

Before diving into specific codes, we need to understand what's under the hood of the OAuth2 authentication process. In short, we send a request to `oauth/authorize` with `client_id` and `redirect_uri`, after user accept this request, Github will redirect to the url with a `code` parameter. This `code` parameter will be used with `client_id` and `client_secret` (which is in your application details) to exchange for an `access_token`. Once you have the access_token, you can make request on the user's behalf, namely, get these available user data.

**Read More on OAuth2: [1](https://developer.github.com/guides/basics-of-authentication/) [2](https://gist.github.com/technoweenie/419219)**


# Wrapper Library

## Simple OAuth2

The authentication is not very complicated, but in this case, we won't be spending too much time on this since what we want is *data*.

I used [Simple OAuth2](http://andreareginato.github.io/simple-oauth2/):

```js
var oauth2 = require('simple-oauth2')({
  clientID: your-client-id,
  clientSecret: your-client-secret
  site: 'https://github.com/login',
  tokenPath: '/oauth/access_token'
});

// specify redirect url, which should be consistent with the callback url in application setting
// specify scope, namely, which set of data or operation you'd like to get
// state is a random string for security reasons

var authorization_uri = oauth2.authCode.authorizeURL({
  redirect_uri: 'http://localhost:3030/callback',
  scope: 'notifications,user,public_repo',
  state: '3(#0/!~'
});

// Initial page redirecting to Github
router.get('/auth', function (req, res) {
    res.redirect(authorization_uri);
});

// Callback service parsing the authorization token
// and asking for the access token
router.get('/callback', function (req, res) {
  var code = req.query.code;
  oauth2.authCode.getToken({
    code: code,
    redirect_uri: 'http://localhost:3030/callback'
  }, saveToken);

  function saveToken(error, result) {
    if (error) { console.log('Access Token Error', error.message); }
    // get access token
    token = oauth2.accessToken.create(result);
    // now you can use this token to call Github APIs

  }
});
```

## Node-Github

As mentioned in the official document, there are many third party wrapper libraries for Github API, I used [node-github](https://github.com/mikedeboer/node-github).

Once you have the access token, calling api with `node-github` is fairly easy, for example, to get a user's repositories:

```js
// assuming it is already authenticated
github.repos.getAll({
		type:'owner'
	},function(err,data){
		console.log(data)
	})
```


# Use Express for Route and API

As for the web frame, I used `Express` with `HandleBars` as the view engine.

`$ npm init`
`$ npm install express --save`

Use `--save` to add as an dependencies in package.json.

It is feasible to use Express Generator to automatically generate the express frame, but for the sake of keeping things minimal viable, let's create things manually.

## Folder Structure

```raw
- app.js (entry file)
- views (using HandleBars)
- routes (this is where we create route and api)
- public (static files, e.g. stylesheets, javascript files.)
```

Aside from regular things such as favicons, cookies etc, let's keep this file simple:

```js
// express config
var express = require('express');
var path = require('path');

// app setting
var app = express();
app.set('views', __dirname+'/views');
app.set('view engine', 'hbs');
app.use(express.static(path.join(__dirname, 'public')));

var routes = require('./routes/index');

app.use('/', routes);

app.listen('3030', function(){
    console.log("server started at 3030");
});

module.exports = app;
```


So we'll be updating the routes/index.js for apis.

## Route

For page rendering:

```js
router.get('/user/:username/:repo/punchcard-page', function(req,res){
	res.render('punchcard')
})
```

For api:

```js
router.get('/codefrequency', function(req, res){
	github.repos.getStatsCodeFrequency({
		user: req.query.username,
		repo: req.query.repo
	},function(err,data){
		if(err){res.send(err)}
		res.json(data)
	})
});

// to implement: select repo -> see data related to this repo
// we need to pass username and repo as parameters to call the getStatsCodeFrequency API
```


By far, we should be able to call an api and get response data, next we'll try to imitate the Github repo graphs with d3.js. (part of the source code is [here](https://github.com/fengshuo/github-stats))
