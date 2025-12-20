---
title: How to make better pull request and reviewing code
categories:
- DevTools
- Git
tags:
- git
- pull request
- code review
- collaboration
description: Some good practise to follow when making pull request or reviewing code
date: 2017-03-04
author_profile: true
classes: wide
---

I was working on a new feature recently, and even though we tried to split a story into multiple subtasks, the PR for this story is still huge. Why?

1) There are Java code changes in that PR, this can be solved by creating different branches for backend code and UI code, most of the time(sometimes the splitting causes inconvenience because you have to merge back and forth to use the api endpoint).

2) UI code is difficult to split into smaller PR, we use Redux&React at work, so basically whenever there is a new feature we need to create the whole set of 'action', 'reducer', and static component, normally we would split the UI task into two categories, one is creating Redux related code, the other is creating presentational React component. But to verify everything is working, we need these two sets of code wired together. (I am thinking use dummy data in React component so that we can separate these two tasks).

3) All the tests, we have unit tests and end-to-end tests, and we have localization files, even if you are just changing one word, it will result in 6 file being detect as a diff. These files aren't exactly code logic related, but it bloats the pull request.

4) We do code review with BitBucket's browser diff tool, and I don't know if it's possible to review files by type, because then as a code reviewer, I can take a look at import code logic files first, then look at simpler changes like localization stuff.

That's why I've been reading articles about how to make a better pull request, here are some practise I think might be applicable in my case:

# Make it smaller
But what if the branch is for a whole new feature, is it good practise to merge in half finished code?

# Do only one thing
I agree with this, but what's the definition of one thing, it's pretty straightforward if it's a bugfix, what if it's a feature branch.

# Make sure the code works on your local machine
this is the minimum requirement before PR.

# Make sure all tests are passing and no linter error
To be honest, most of the time I make sure all tests and linter are passing, but sometimes when I realize I need to change some other things in the code, sometime I forget when the tests/linter message was outside of the terminal window of the build task.

# Document
Of course, code should be self-documenting, we should only add extra comments when it's absolutely necessary to add more comments, the priority would be 1) self-documenting code 2) code comments 3) commit messages 4) pull request comments.

## Commit messages

I am guilty of not providing high-quality commit messages, I commit quite often so I give myself excuse to write shorter commit message, but that's not helpful if you want to revisit a commit to understand what happened. What I find might be useful is to *add a JIRA issue ticket number*(or whatever issue track tool you use, I use [a a sh file](https://gist.github.com/lnolte/4016361) to do the copy and paste current git branch name and it works well for me) at the beginning of each commit message, to keep a reference to the context.

Also, commit messages should talk about what changed and why, not why.

## Pull Request messages/comments

If you are using BitBucket, it will automatically allocate all the commit message and put it into the PR description, but it will still make the reviewer's life easier if there are more context. For instance, you could include the title of the PR(_This fixes handling of…_), and add more details in the descriptions.

The other thing is that you can add visual screenshots as a PR comment, I find this very useful for UI code review, so the reviewer doesn't need to build on their local machine and find this element.

# Keep the commit history clean

I mentioned in another [post](http://fengshuo.co//use-git-to-undo-changes-and-rewrite-history/) of using `git commit --amend` or `git rebase` to keep commit message clean. In retrospect, I think I do commit often, but I also push it to remote even though the commit is not needed or no other developers are collaborate on this branch, this means it will be tricky for me to rewrite history, because `git commit --amend` and `git rebase` works best with **local** changes. So I think for me, I need to think before I push commits to remote.(commit locally often, but don't push that often unless when it's necessary).

I find links below are helpful, and I need to revisit these practises/suggestions often to make better PR and also code review.

# Links
[The (written) unwritten guide to pull requests](http://blogs.atlassian.com/2016/07/written-unwritten-guide-pull-requests/)
[How to write the perfect pull request](https://github.com/blog/1943-how-to-write-the-perfect-pull-request)
[Code Review](https://github.com/thoughtbot/guides/tree/master/code-review)
[10 tips for better Pull Requests](http://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/)
[GitBestPractices](http://sethrobertson.github.io/GitBestPractices/)
[Effective pull requests and other good practices for teams using github](http://codeinthehole.com/writing/pull-requests-and-other-good-practices-for-teams-using-github/)
[Giving better code reviews](https://medium.com/@mrjoelkemp/giving-better-code-reviews-16109e0fdd36#.tp1g9kp9a)
