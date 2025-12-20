---
title: Manage SVN Repos with Git
categories:
- DevTools
- Git
tags:
- dev
- git
- svn
description: mix svn and git
date: 2015-03-03
author_profile: true
classes: wide
---

I've never used SVN for personal projects, I am a big fan of Git. However, the chances are high that you will be working with SVN if you are working in a large company instead of new startups.

One thing I find not good about SVN is the support for local changes, I tend to try new ideas/branches on my local machine frequently, and commit to remote server whenever I am ready. But using SVN for trial purposes is a horrible experience for me.

I wanted to use Git to manage version control, but there is no way for the team to abandon SVN immediately. So I found ways to mix SVN and Git.

## Checkout SVN Repo

Checkout out your project from svn:

`svn checkout http://XXX`

## Git the Folder

Initialize this folder as a git repo, but remember, *do not add anything yet*.

`git init`

## Ignore Each Other

The key to manage this mix of git and svn is to allow those two version control systems ignore each other. Therefore, let's git ignore svn and svn ignore git.

### Git Ignore SVN

Add '.svn' in the .gitignore file

### SVN Ignore Git

`svn propset svn:ignore .git .`

(Read more about svn ignore [here](http://superchlorine.com/2013/08/getting-svn-to-ignore-files-and-directories/))

## Add Things to Git

By far those two version control systems are ignoring each other, and you are all set to use git for your local experiments.

## Manage the Mix
Although it is essential to realize that it is better to keep the [git master branch clean and update](http://lostechies.com/derickbailey/2010/02/03/branch-per-feature-how-i-manage-subversion-with-git-branches/) with just the SVN repo. That is to say, do your local experiments on other branches, stash or commit on other branches, then switch to master, either update with svn or merge with your developement on other branches.
