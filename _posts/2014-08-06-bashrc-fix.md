---
title: Fixing Bashrc Loading Issue
categories:
- DevEnv OS
tags:
- dev
- fix
- bash
- node
description: fix bashrc loading issue
date: 2014-08-06
author_profile: true
classes: wide
---

While I was working on node 'npm link', I need to source ~/.bashrc everytime if I want my new npm link to work (even if I already got NODE_PATH in the bashrc file), it turned out to be a bashrc setting problem.

The fix is simple:

Add this snippet in `~/.bash_profile`:

```bash
[[ -s ~/.bashrc ]] && source ~/.bashrc
```

The reason why this issue happened:
Terminal opens a login shell. This means, `~/.bash_profile` will get executed, instead of `~/.bashrc`. ([stackoverflow link](http://stackoverflow.com/questions/7780030/how-to-fix-terminal-not-loading-bashrc-on-os-x-lion))
