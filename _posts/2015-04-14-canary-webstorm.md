---
title: LiveEdit with Chrome Canary and WebStorm
categories:
- Notes
tags:
- chrome
- canary
- webstorm
- debug
description: liveedit with chrome and webstorm
date: 2015-04-14
author_profile: true
classes: wide
---

It is common to use live edit/live reload for front end debug. As for fantastic IDEs such as WebStorm, they are bundled with these kind of plugins. The thing is when I tried to use the liveedit in WebStorm, I'd really like to open/refresh things in Chrome Canary instead of the system default browser - Chrome.

So here is how to set this up:

Open the setting dialog, in Tools>Web Browsers, add a new Browser with below properties:

- Name: Google Chrome Canary
- Path: Google Chrome Canary

Notice that the path the app name instead of the install path.

And select default browser to 'custom path', with path - 'Google Chrome Canary'.

Setting live edit:

Run>Edit Configuration, add a new debug type with the default browser, chrome canary.

Initially this wasn't working, but after I update the canary browser, it worked.
