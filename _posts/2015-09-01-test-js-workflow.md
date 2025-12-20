---
title: Testing Javascript and Continuous Deployment
categories:
- Frontend
- Web
tags:
- testing
- travis
- jasmine
- karma
- d3.js
description: Testing javascript(d3.js) with jasmine and karma, continuous test and
  deploy to npm with travis
date: 2015-09-01
author_profile: true
classes: wide
---

Recently I've been working on a d3.js project within a narrow time frame, meanwhile the product requirements changes frequently. That lead to messy codes partly because I didn't have time to refactor or write any tests, just wanted to implement changing features and fix bugs. And I've had enough with the "fix this but unexpectedly break that" scenario, this is the first reason why I want to actually integrate [TDD](https://en.wikipedia.org/wiki/Test-driven_development) into future projects.

The second reason is about d3.js. Once I started to implement multiple charts, the necessity of writing [reusable d3.js components](http://bost.ocks.org/mike/chart/) became more conspicuous, and this seems to be the perfect timing to integrate unit tests into d3.js projects.

Things I would like to achieve are: 1) Use jasmine to write tests, and let karma take care of automated tests 2) continuously (and conditionally) test the code and deploy them to npm (deploying to services like Heroku is similar, I'm just using npm here for demonstration purpose).


# Set Up Karma and Jasmine

## Install and Configure Karma

[Jasmine](http://jasmine.github.io/) provides a [standalone distribution](https://github.com/jasmine/jasmine/releases), but I think more often we would use command line to run tests. In this case, we are gonna use [Karma](http://karma-runner.github.io/0.13/index.html), thus we need to install the following packages:

```raw
npm install karma --save-dev
npm install karma-jasmine karma-chrome-launcher --save-dev
```
Another thing worth noting is that installing a karma cli (`npm install -g karma-cli`) is helpful for running local tests. But in terms of installing dependencies while running tests on Travis, I think using the karma filepath can save some time(`node_modules/karma/bin/karma start my.conf.js`).

Now Karma is up, let's initiate it by creating a config file: `karma init my.conf.js`.

All we need to do is to answer those pumped questions to get a karma config file.

There are many other options described [here](http://karma-runner.github.io/0.13/config/configuration-file.html), which you can use to override config file settings in command line.


## Write Tests with Jasmine

In previous step, the test source and target filepath is: `/dist/*.js`, `tests/*.js`. Now let's create these folders and files with a simple example:

Example source file:

```js
(function() {
  var n = document.createElement('svg');
  n.classList.add('canvas');
  document.body.appendChild(n);

})();
```

Example tests file:

```js
describe("SVG Exists", function(){
    var svg = document.querySelector("svg");

    it("should exists", function(){
        expect(svg.nodeName).toBe("SVG")
    })

    it("has classname canvas", function(){
        expect(svg.getAttribute("class")).toBe("canvas")
    })
})
```

Now let's run these two tests locally first:  `karma start my.conf.js --single-run`.

And this is what we get in terminal:

```raw
[karma]: Karma v0.13.9 server started at http://localhost:9876/
[launcher]: Starting browser Firefox
[Chrome 45.0.2454 (Mac OS X 10.10.3)]: Connected on socket 7608sGv8nPD02NmUAAAA with id 54556099
Chrome 45.0.2454 (Mac OS X 10.10.3): Executed 2 of 2 SUCCESS (0.025 secs / 0.004 secs)
```

(Note: To run tests with browser, install [firefox launcher](http://stackoverflow.com/questions/19255976/how-to-make-travis-execute-angular-tests-on-chrome-please-set-env-variable-chr) instead of chrome launcher)


# Set Up Travis

Before delving into the configuration details, we need to sign up [Travis](https://travis-ci.org/) with Github so that Travis can run/deploy our open source projects. Once you give access to Travis, it will sync all your github repos(this can take a few minutes depends on how many repos you have). After the syncing, we need to tell Travis to track this repo.

Now let's add a `.travis.yml` file in project root:

```raw
language: node_js
node_js:
    - "0.10"
script: karma start my.conf.js --single-run
# if you didn't put karma-cli in package.json, then use node_modules/karma/bin/karma to start running tests
before_install:
    - export DISPLAY=:99.0
    - sh -e /etc/init.d/xvfb start
    # Travis supports running a real browser (Firefox) with a virtual screen
before_script:
    - npm install  # install all dependencies before every suite.
on:
  tags: true # deploys when a tag is set, if you're gonna use travis setup npm later, this line will be generated automatically
notifications:
  email: false  # turn off email notifications
```

There are other available [GUI](http://docs.travis-ci.com/user/gui-and-headless-browsers/) testing methods available in [Karma](http://karma-runner.github.io/0.8/plus/Travis-CI.html).

Now let's push updates to github remote origin, meanwhile you refresh your Travis profile page to see running tests and reports.

# Deploy to NPM

Travis can also automatically release/deploy your repo, let's take releasing to NPM as an example. There are two ways to update the travis config file. I prefer to use the travis CLI to add and encrypt api keys for me(Note: if it's a new repo, you need to update repos in Travis):

```raw
# Install Travis CLI
$ gem install travis

# find your npm api key
$ cat ~/.npmrc

# let travis cli to do the work
$ travis setup npm
```

Beyond that basic setting, I think it is also necessary to add [conditional release settings](http://docs.travis-ci.com/user/deployment/#Conditional-Releases-with-on%3A) such as which branch to deploy, or deploy only when there is a tag.

# Conclusion

Source code [here](https://github.com/fengshuo/testing-workflow).
