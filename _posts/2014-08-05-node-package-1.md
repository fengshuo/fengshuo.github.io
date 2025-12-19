---
title: "Node Package 101"
layout: single
date: 2014-08-05
categories: node javascript
tags:
- dev
- node
author_profile: true
classes: wide
description: learn to write node package
---

This is a short note about how to get started with node package. Specifically, create an npm package which will install a `warm` command, and you can pass arguments with this command, for example:  
{% highlight js %}
$ warmwhere China  
	China is warm now
$ warmwhere China --south  
	in China, south is warm
{% endhighlight %}

# Step 1 - Create an npm package

## edit package.json

There are many useful key-value pairs in package.json, but in this case, I want to create an npm package so that I can require the module in other javascript files, which means we need to confirm the "name", and "main" key standing for the package name and the file loaded by default respectively:  

{% highlight javascript %}
//in package.json
{
	"name":"warm",
	//..sth else
	"main":"index.js"
}

{% endhighlight %}

[Link](https://www.npmjs.org/doc/files/package.json.html)  

Let's see how it works:

`$node -e 'console.log(require("./index"))'`  

or you can require the directory(which will use the value of "main" as default file):

`$node -e 'console.log(require("./"))'`  

### npm link

Here is the tricky part for me, since we all use `var somePlugin = require("somePlugin")` after npm install, we would expect to use require with the package name directly, that's where **[npm link](https://www.npmjs.org/doc/cli/npm-link.html)** comes in handy. The api doc is self-explanatory itself, the key here is the symbolic link to the current folder:

`$npm link`  
`/usr/local/lib/node_modules/warm -> /Users/fengshuo/Documents/dev_fs/nodeTemp/warm`  

One thing to notice is the export PATH, if you ever counter something wrong with npm link, check your `~/.bashrc` file, add `'export NODE_PATH=/usr/local/lib/node_modules'`, and don't forget to `source ~/.bashrc`.  

Now, you can require your package by its name directly.  
To unlink:
`npm unlink -g warm`;  

## Step 2 - Provide commands
In addition to providing a package, it is quite often for us to write a package with command just like the example in the very beginning.  

npm provides us with the convenience to do this by modifying the `bin` field in [package.json](https://www.npmjs.org/doc/files/package.json.html), such as:  

{% highlight javascript %}
"bin":{
	"warmwhere":"./bin/index.js"
}
{% endhighlight %}

Basically, this will create a symlink, from your index.js file to `/usr/local/bin/npm` where our system check for any terminal commands.  

### create index.js
There are a few tips for creating executable files:

`#!/usr/bin/env node   `
//other things like require your previous package here

$ chmod a+x ./index.js  to make it executable

let's do npm link again, getting:  
`/usr/local/bin/warmwhere` -> `/usr/local/lib/node_modules/warm1/bin/index.js`  

Now you can use the 'warmwhere' command.  

## Step 3 - Handle command arguments

To handle arguments, the native process.argv is available, but the [minimist](https://github.com/substack/minimist) package is also easy to use, just use minimist to handle command arguments.

That's it for local development boiler template. Upload package to npm registry if you want to share it.
