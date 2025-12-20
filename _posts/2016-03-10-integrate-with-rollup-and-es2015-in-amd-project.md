---
title: Integrate ES2015, Rollup in legacy 'AMD' project
categories:
- Frontend
- Web
tags:
- rollup.js
- ES2015
- AMD
description: Describe how to integrate rollup.js, ES2015 in a legacy AMD driven project,
  and related issues
date: 2016-03-10
author_profile: true
classes: wide
---

# Background

I've been working on an AMD-driven project for a while, now there is a requirement to build a relatively independent subsystem. I thought this might be a good timing to start write javascript in ES2015 style and try out some new tools.

But the reality is, it's not building a new project from scratch where I can choose a whole new architecture or structure, it is vital that the new subsystem should be conformed with existing project structure, especially all these setups built on top of AMD modules.

So my plan is simply use compile ES2015 modules use Babel and Gulp(since gulp is already used in the project).

# Integrate with ES6 in Gulp

The setup here is pretty straightforward, I just added a new task in gulp to convert ES2015 to AMD modules.

```js
var babel = require('gulp-babel');
var sourcemaps = require('gulp-sourcemaps');

gulp.task("es6", function () {
  return gulp.src("js/modules/es6/*.js")
    .pipe(sourcemaps.init())
    .pipe(babel({
      "presets": ["es2015"],
      "plugins": ["transform-es2015-modules-amd"] // need to convert ES6 to AMD
    }))
    .pipe(sourcemaps.write()) //relative to the destination
    .pipe(gulp.dest("js/modules/build"));
});
```

This is how I import modules in the new ES2015 modules:

```js
import d3 from 'd3';
import myModule from 'modules/newSystem/localization';

// ...
```

Two things worth noting here, the first is that common libraries are already defined in requirejs config(such as d3.js), and the reason why I need to import `myModule` from a 'parent' directory is that I would like the dependencies in the complied AMD file is consistent.

```js
define(['d3', 'modules/newSystem/localization'], function (_d, _myFunc) {
  'use strict';
  // ...
});
```

The initial goal is basically completed since I didn't introduce many new things, now I can import modules in ES6 style, and this setup doesn't require any other change in [AMD config files](http://stackoverflow.com/a/35884176/2749820).

However, this ES6-to-ES5 compiling task generate more files in the project folder, which sort of slows down the release process.

So to take one step further, I'd like to bundle all the ES6 modules I created for this new subsystem into one file, so that I can compile just this one bundle file into one AMD file instead of compiling all ES2015 module files.

So basically the project structure should look like this:

```raw
-es6/
--all es6 module files/
--es6Entry.js
-build/
--bundle.js
-AMDEntry.js
```

# Introduce Rollup

There are several module bundler tools available, such as Browserify and Webpack, although browserify is more suitable for commonjs. Webpack is awesome, but I think that I won't be needing many features of webpack this time, all I need is a ES6 module bundler to bundle es2015 modules into an AMD file. [Rollup](http://rollupjs.org/) seems to be a good fit.

There is nothing complicated to use rollup in a 'hello world' tutorial as described [here](http://rollupjs.org/guide/). However, it is relatively new, Below are the problems I encountered.

## Resolve File Path

Initially I set up the `rollup.config.js` with minimal configuration, and created a `entry.js` and `moduleA.js`in the `es6` directory as mentioned above:

```js
/**
 * es6 folder
 * entry.js
 * moduleA.js
 */

// entry.js
import myModule from moduleA

// moduleA.js
export default myModule (){
  console.log('my module')
}
```

The weird thing is, after running the `rollup -c` command, it logged `Treating moduleA.js as external dependency`. That's because currently [Rollup currently considers all non-relative paths to be references to external modules](https://github.com/rollup/rollup/issues/104), so you'll have to write relative paths to import modules, bummer. Luckily, as mentioned in the issue thread, this problem can be solved by using this plugin - [rollup-plugin-includepaths](https://github.com/dot-build/rollup-plugin-includepaths), let's update rollup config file:

```js
import includePaths from 'rollup-plugin-includepaths';

var includePathOptions = {
  paths: ['es6'] // I put all es6 module files here
};

export default {
  entry: './es6/entry.js', // relative to where you execute rollup command line
  plugins: [
    includePaths(includePathOptions)
  ],
  format: 'amd', // choose format
  dest: 'build/bundle.js',
  sourceMap: true
};
```

Now rollup should bundle moduleA into the bundle.js without treating it as external dependency.


## Bundle Third Party Libraries

I switched to [i18next](http://i18next.com/) for localization instead of the requirejs loader in the new subsystem, not because it is more compatible with different module setup, but also it has more useful API for localization. But it turned out to be the most frustrating process while integrating with Rollup.

### Approaches Didn't Work

At first, I thought it should be fine to download the compressed `i18next.min.js`, and treat them like a regular module file due to its UMD format. That didn't work, because rollup is a ES6 module bundler, the prerequisite of bundling files is that those files have to be in ES6 module format. Considering I might need more third party libraries, and not every library is written in ES6, I need to find a way to make them compatible with Rollup.

There is a plugin listed in Rollup wiki - [rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs), it is supposed to convert CommonJS to rollup compatible format, so I update rollup config:

```js
import commonjs from 'rollup-plugin-commonjs';

// add commonjs in plugin config
plugins: [
    includePaths(includePathOptions),
    commonjs({
      include: './vendors/**'
      // I was stupid to put standalone minified i18next.min.js in a new folder
    }),
    babel()
  ],
```

Running rollup will only get an error message like this: `Module [directory]/vendors/i18nextXHRBackend.min.js does not export default (imported by [directory]/es6/localization.js)`. No matter how I tried to tweak it just wouldn't work.

So I figured it might be the wrong way to use standalone minified js(eventually), I then installed the libraries via npm, but in the parent folder of `es6`:

```js
-subSystem
--es6
---entry.js
---localization.js
--node_modules
--build
--AMDEntry.js
--rollup.config.js
```

And error message is either `can't find module` or `does not export default`. So I post it as an [issue](https://github.com/rollup/rollup/issues/547) on Github, luckily rollup authors responded soon. I went to see the source code of [i18next](https://github.com/i18next/i18next/tree/master/src), and found that a) the source code is already in ES6, so there shouldn't be a compatible issue, b) there is no `require === function` test in the minified js, so it is not the [same issue](https://github.com/rollup/rollup-plugin-commonjs/issues/38).

At that time, I was thinking perhaps the way I was using npm package was incorrect since the rollup author suggested install via npm. So I installed npm packages in the same directory of entry.js:

```raw
-subSystem
--es6
---node_modules
----i18next
---entry.js
---localization.js
--build
--AMDEntry.js
--rollup.config.js
```

And updated rollup config (as the response said, used node resolve to handle requiring libraries):

```raw
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs';
import includePaths from 'rollup-plugin-includepaths';
import nodeResolve from 'rollup-plugin-node-resolve';

var includePathOptions = {
  paths: ['es6']
};

export default {
  entry: './es6/entry.js',
  plugins: [
    includePaths(includePathOptions),
    nodeResolve({
      jsnext: true,
      main: true,
      browser: true
    }),
    commonjs({
      include: './es6/node_modules/**'
    }),
    babel()
  ],
  format: 'amd',
  dest: 'build/bundle.js',
  sourceMap: true
};
```

Finally, this setup is working! Here is the sample `entry.js`:

```js
// entry.js
import localization from 'localization';

localization();

console.log('entry file loaded....')
```

and `localizaiton.js`:


```js
import i18next from 'i18next';
import XHR from 'i18next-xhr-backend';
import i18nextjquery from 'jquery-i18next';

// i18next settings

var i18nextInstance = i18next
  .use(XHR)
  .init({
    backend: {
      loadPath: '/mtools/js/locales/{{lng}}/{{ns}}.json'
    },
    lng: window.defaults.localeLanguage === 'ZHS' ? 'zh' : 'en',
    fallbackLng: 'zh',
    //debug: true
  }, (err, t) => {
    $('body').localize();
  });


i18nextjquery.init(i18nextInstance, $, {
  tName: 't', // --> appends $.t = i18next.t
  i18nName: 'i18n', // --> appends $.i18n = i18next
  handleName: 'localize', // --> appends $(selector).localize(opts);
  selectorAttr: 'data-i18n', // selector for translating elements
  targetAttr: 'data-i18n-target', // element attribute to grab target element to translate (if diffrent then itself)
  optionsAttr: 'data-i18n-options', // element attribute that contains options, will load/set if useOptionsAttr = true
  useOptionsAttr: false, // see optionsAttr
  parseDefaultValueFromContent: true // parses default values from content ele.val or ele.text
});

export default function () {
  console.log('localization.js')
}
```

And the localization function works, yay!


## Include Existing Libraries

One more thing I need to test is to use existing libraries which are already defined in require.js config. For example, I added this line in localization.js:

```js
import d3 from 'd3';
//...
```

Initially I thought rollup will treat d3 as an external dependency just like when I import moduleA.js, but it didn't work, the error message is `can't find module d3 in directory/es6`, I suppose it is because 'rollup-plugin-includepaths' kind of 'override' the default rollup behavior? Anyway, since rollup said it can't find the module, I had to specify where to find the module:

```js
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs';
import includePaths from 'rollup-plugin-includepaths';
import nodeResolve from 'rollup-plugin-node-resolve';

var includePathOptions = {
  paths: ['es6'],
  include: {
    'd3': '../../../' + 'global/js/' + 'base/d3.min'
    /*
    d3 is in another global folder, the path is already defined in require.js, since I am going to compile it to an AMD bundle eventually, I think using this path should be fine since the 'real' path in different environment will be handled by require.js
    */
  },
  external: ['d3']  // exclude it from the bundle
};

export default {
  entry: './es6/entry.js',
  plugins: [
    includePaths(includePathOptions),
    // it seems that it is better to put includePaths as the first plugin
    nodeResolve({
      jsnext: true,
      main: true,
      browser: true
    }),
    commonjs({
      include: './es6/node_modules/**'
    }),
    babel()
  ],
  format: 'amd',
  dest: 'build/bundle.js',
  sourceMap: true
};
```

This time rollup didn't report any errors.

## Issue with i18next

One last issue is that even though there is no error message from rollup, when I loaded the bundle file in browser, there was an error message from i18next-xhr-backend saying 'utils.defaults is not a function', so I digged into the source file of i18next-xhr-backend, everything seems to be fine, but it failed to import `utils` module here:

```js
var _utils = require('./utils');
```

So I tried to change the require to:

```js
//NOTE: remove relative path then it works, not sure which plugin cause this problem
var _utils = require('utils');

var utils = _interopRequireWildcard(_utils);
```

Then it works, honestly, I don't know if this is rollup problem, node-resolve problem, or commonjs plugin problem, don't think it's problem of the package though. The author of i18next is [working on](https://github.com/i18next/i18next-xhr-backend/issues/73) to improve the library to avoid this kind of error.

## Watch File Change

For now, rollup doesn't have any watch options yet, but many developers shared their alternative solutions in this [thread](https://github.com/rollup/rollup/issues/284). I prefer the [watch](https://www.npmjs.com/package/watch) approach, So I installed 'watch' in the subsystem folder (I tried to install it in the es6 folder and update config, but it didn't work, I think it is fine to just use the existing folder structure anyway):

```js
-subSystem
--build
--es6
--node_modules
--.babelrc
--rollup.config.js
--package.json // details below
```

In the outer directory, package.json:

```js
// ...
  "scripts": {
    "build": "rollup -c",
    "watch": "./node_modules/.bin/watch \"npm run build\" es6/"
  },
// ...
```

This way, any changes in es6 folder will trigger rollup to re-bundle files.

Additionally, Gulp can also watches the bundle file change for other tasks.

## Add Classic jQuery Plugins as ES2015 module

See details [here](http://stackoverflow.com/questions/35136874/extend-es6-plugin-to-jquery-prototype).
