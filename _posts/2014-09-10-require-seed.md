---
title: Use RequireJS
categories:
- Frontend
- Web
tags:
- dev
- frontend
description: loading scripts with require.js
date: 2014-09-10
author_profile: true
classes: wide
---

I came to realize the concepts of CMD and AMD while playing with NodeJS, but it didn't seem very useful for me back then since all the tiny demos I was working on are quite small ones, so module loader never occurs to me as one of the necessities. But the project I am currently involved is much bigger than those little demos, and I was just following the existing module loader patterns with RequireJS. Here I would like to take some notes of RequireJS.

For starters, the API and samples on the [official site](http://requirejs.org/) are really good, I am just trying to add more my take here.


# Basic

## Introduct RequireJS to the project

In your entry html, add a script in a page with `src` set to be require.js, and add another attribute `data-main`, which represents the entry point for js modules.

```html
<!DOCTYPE html>
<html>
    <head>
        <title>My Sample Project</title>
        <!-- The path here is relative to index.html
        		There are [other ways](http://requirejs.org/docs/api.html#jsfiles) to load the main js file
        -->
        <!-- data-main attribute tells require.js to load
             scripts/main.js after require.js loads.
        -->
        <script data-main="scripts/main" src="scripts/require.js"></script>
    </head>
    <body>
        <h1>My Sample Project</h1>
    </body>
</html>
```

## Define Modules

See API doc [here](http://requirejs.org/docs/api.html#define).

## Commonly Used APIs explained

RequireJS loads each dependency as a script tag. Make sure those dependencies are loaded in the right order. You'll need to config in advance to make it work, for example:

```js
require.config({
	//baseUrl is the root path for module js lookups
	//if this option is not defined, the default will be the location of the index.html which loads require.js
	baseUrl: './js',
	//paths mappinf for modules that cannot be found under baseUrl,
	//the paths in 'paths' are relative to baseUrl
	//you don't need to add the .js for each file
	paths: {
		'app':'../app'
		'angular' : '../app/angular.min',
		'jquery' : '../app/lib/jquery',
		'uiRouter': '../app/lib/angular-ui-router',
      'ngAnimate': '../app/lib/angular-animate.min'
	},
	//shim is to configure those tranditional scripts without AMD support available in requireJS project
    shim:{
        'angular': {
        		//deps means this dependency should be loaded before loading angular
            'deps': ['jquery'],
            //exports means after its loaded, you can use the export value as the module value
            'exports': 'angular'
        },
        'ngAnimate': {
        		//for modules like plugins that do not need to export any module value, you can either omit the export, or just write it like this
				['angular']
        },
        'uiRouter': {
            'deps': ['angular']
        }

    }
});
```


# Optimization

Since requireJS encourages to write one module per file, sooner or later you'll end up creating many files, that's why it is quite useful to optimize all those files for production.

RequireJS comes with r.js, the installation and usage is quite simple as listed in the api, here is what I did in configuration file:


```js
({
	 //all the paths in config.js is relative to the config.js
	 //use appdir to specify the original project, and baseUrl will be relative to it
    appDir: "../www",
    baseUrl: "./js",
    //use this so you don't need to write paths again here
    mainConfigFile: "../www/main.js",
    //the product project where you want to put all optimized files
    dir:"../prod",
    keepBuildDir: false,
    optimize: "uglify",
    optimizeCss: "standard",
    modules: [
        {
            name: 'common',
            //make sure include file is loaded into common.js
            //you can also use exclude to avoid duplicated dependency injection
            include:['urls']
        },
        {
            name: 'tools',
            include: ['someTool']
        }
    ]

})
```


## Links

[1](http://www.bennadel.com/blog/2404-compiling-optimizing-a-subset-of-a-requirejs-application.htm)
[2](http://tech.pro/blog/1639/using-rjs-to-optimize-your-requirejs-project)
[3](https://github.com/jrburke/r.js/blob/master/build/example.build.js)
[4](https://github.com/requirejs/example-multipage-shim/blob/master/www%2Fjs%2Fapp%2Fmain1.js)
[5](https://github.com/maxdow/angularjs-requirejs-seed/blob/master/app%2Fjs%2Fmain.js)
[6](https://github.com/fengshuo/require-angular)
