---
title: Use Webpack with D3 Dashboard Pages
categories:
- Frontend
- Web
tags:
- webpack
- d3
- reusable
- dashboard
description: Integrate webpack with d3 charts reusability to construct modules used
  among dashboard pages
date: 2015-06-27
author_profile: true
classes: wide
---

# Introduction

Although I found it a little bit tedious to set up margin conventions, scales, axis while studying the demos, I didn't put chart reusability on the top of my todo list until I finished the [Air Quality](http://fengshuo.co//air-quality-index-chart/) demo. Rewriting some basic or common settings can be time consuming given that you have to spend a huge amount of time to clean and process data per se.

Apart from reuse components or settings of charts on the same page, it is more efficient if these functions can be used across pages, say you have a relatively large multipage or single page dashboard app, each page has its own purpose of presenting data, but they certainly have something in common, be it the layout or styles of path or line.

That's why I want to work on reusing d3 components within and across pages.

# Within the Page - Reusable D3 Chart

Mike wrote [an article](http://bost.ocks.org/mike/chart/) addressing reusable charts a while ago, this post is a really good start to learn to add reusability to your chart despite existing different [opinion](http://bocoup.com/weblog/reusability-with-d3/). To understand this methodology better, there are some [examples](https://github.com/mbostock/d3/wiki/Gallery#charts-using-the-reusable-api) dedicated to this topics.

Another quite new tool is [d3kit](https://github.com/twitter/d3kit) that I mentioned earlier. I haven't tried it yet since I am still trying to figure which level of customization is best for me. But give it a try if you want since the maintainers said that it is already been used in [production](https://github.com/twitter/d3kit).

# Get Started with Webpack

With regards to efficiency across pages, or packaging files, there are many outstanding tools including grunt, gulp, browserify, components etc. They all have its own pros and cons which I won't address here. The reasons I prefer [webpack](http://webpack.github.io/) including splitting files, packing other static resources like css and images, etc.

The general goal is to write javascript for each chart on one dashboard page, , reuse common modules, and packaging publicly used vendors/modules across pages in one app.

This [webpack-howto](https://github.com/petehunt/webpack-howto) post can provide you with a quick view about what webpack can do and how to do them.


# Scenarios & Examples

## Setting Up Webpack

The [official tutorial](http://webpack.github.io/docs/tutorials/getting-started/) is fairly straightforward and simple, but let's get more practical here(assuming you have all necessary packages correctly installed):

*Project Structure*:

```raw
/app
--/js
----/components
------margin.js
------axis.js
----main.js
--/style
----base.less
----style.less
/build
--page1.html
webpage.config.js
/node_modules
```

By and large, the `build` folder is the ultimate page while `app` folder is where we write modules and utilize these modules. The most important file is the `webpage.config.js` in which we config webpack.

Another thing worth noting is that in the [demo repo](https://github.com/fengshuo/webpack-d3-charts), there is no style file in build folder, which is different than outcomes of other packaging tools I used before, we'll look at it in a minute.

The minimal codes for webpack.config.js are:

```js
module.exports = {
	entry: "./app/js/main.js",
	output: {
		"path": "./build",
		"filename": "bundle.js"
	}
};
```
That is to say webpack will take the `main.js` in `app` folder and the dependencies, and generate a `bundle.js` in build folder.

Meanwhile, let's create a `page1.html` in build:

```raw
<head>
	<meta charset="UTF-8">
	<title>Page1</title>
</head>
<body>
<h1>page 1</h1>
<script src="bundle.js"></script>
</body>
```

Then write sth in the `margin.js` module and `main.js`:

```js
// margin.js
module.exports = {
  console.log("this is margin")
}
```

and use this component in `main.js`:

```js
var margin = require('./components/margin.js');
console.log(margin);
```

Open your browser and check that message in console.


## Webpack Dev Server

The approach of open html files directly with browser is obviously out of date. We can use `webpack-dev-server` instead.

So what we need for local development? 1) recompile files once updated 2) show source map files 3) live reload:

`webpack-dev-server --devtool eval --progress --colors --content-base build --port 3030`

What this line of command does is to:

1) add source url so errors will point to the right file(`--devtool eval`, `--d` is also okay but slower);
2) show compiling progress and add some colors to output message (`--progress --colors`)
3) specify the directory to serve, the build folder in this case (`--content-base build`)
4) override default port 8080 to 3030

To save time, you can add the above command line to package.json:

```raw
"scripts": {
    "dev": "webpack-dev-server --devtool eval --progress --colors --content-base build"
},
```

One thing I need to point out is that if you use webpack-dev-server, it won't generate the bundle.js file in build folder because [modified bundle is served from memory](http://webpack.github.io/docs/webpack-dev-server.html).

## More Than Just JavaScript

As mentioned [here](http://webpack.github.io/docs/motivation.html#why-only-javascript), other static resources also need to be handled, such as stylesheets, images, template html etc. For example, we need to apply the style.less to page1.html, all we need to do is add this in `main.js`:

`require('../style/style.less')`

and config loaders in config file:

```raw
...
module: {
		loaders: [
			{test: /\.less$/, loader: 'style!css!less'}
		]
},
...
```

And don't forget to install `style-loader`,`css-loader`,`less-loader` before compiling. What happened is that for every `.less` file, it will be processed by `less-loader`,`css-loader`,`style-loader`(apply the styles in the css file).


## Add External Libraries

Suppose we need d3.js to render data presentation and jQuery for DOM manipulation or other things, let's integrate them into our app by download them first: `bower install d3 jquery --save`

And update config file:

```raw
...
resolve: {
		alias: {
      'jquery': bower_dir + '/jquery/dist/jquery.min.js',
      'd3': bower_dir + '/d3/d3.min.js'
    }
}
...
```

Furthermore, we can [add a function in config](http://christianalfoni.github.io/javascript/2014/12/13/did-you-know-webpack-and-react-is-awesome.html#common) to make it easier to add external libraries:

```raw
addVendor: function(name, path){
		this.resolve.alias[name] = path;
		this.module.noParse.push(path);
},
...

config.addVendor('jquery', bower_dir + '/jquery/dist/jquery.min.js');
config.addVendor('d3', bower_dir + '/d3/d3.min.js');
```

Restart the webpack, and require the external libraries in your js files:

```js
// main.js
var d3 = require("d3");
console.log(d3.version)
```

## Make Modules Available Everywhere

Since we're going to use d3.js almost everywhere, it would be cumbersome to require d3 in every file. Fortunately, webpack allows us to make it available in every file as a customized variable by using the `ProvidePlugin` plugin:

```raw
plugins: [
  // This plugin makes a module available as variable in every module
  new webpack.ProvidePlugin({
    d3: "d3",
    $: "jquery"
  })
]
```

Then we can just simply use `d3` as usual in components files.

## Extract Common Chunks

Another scenario is to extract commonly used javascript chunk into a seperate file, for instance, we need all external libraries compressed in a `vendors.js` so we can use it in other pages, this can be achieved by the `CommonsChunkPlugin` in webpack:

```raw
...
entry: {
		app: ["./app/js/main.js"],
		vendors: ["d3","jquery"]
},
...
plugins: [
	 new webpack.ProvidePlugin({
			d3: "d3",
			$: "jquery"
		}),
	  // use consistent key name vendors
		new webpack.optimize.CommonsChunkPlugin('vendors','vendors.js')
]
```

Run webpack, it will generate another file `vendors.js` in build folder which contains the d3 and jquery libraries.


## Multiple Entry Points

Let's say we want to use different bundles for `page1.html` and `page2.html`. Assuming page2.html contains maps, thus needs extra libraries such as topojson.js

```raw
...
entry: {
		page1: ["./app/js/main.js"],
		page2: ["./app/js/main2.js", "topojson"],
		vendors: ["d3","jquery"]
	},
output: {
	"path": "./build",
	"filename": "[name].bundle.js"
},
...
config.addVendor('topojson', bower_dir + '/topojson/topojson.js');
```

Furthermore, you can also use `CommonsChunkPlugin` to [extract common chunks from different sets of js files](https://github.com/webpack/docs/wiki/optimization).

Will update advance usage of webpack later.

## Links
[Smarter CSS Builds with Webpack](http://bensmithett.com/smarter-css-builds-with-webpack/)
