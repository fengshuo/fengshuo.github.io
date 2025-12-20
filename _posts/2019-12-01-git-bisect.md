---
title: Use git bisect to find first bad commit
categories:
- DevEnv
- Shell
tags:
- git
- bisect
- version control
- debug
- binary search
- algorithm
author_profile: true
classes: wide
---

Today I learned that there is a way to find first bad commit with git. Before learning about `git bisect`, whenever I see a commit is buggy after introducing a new feature, I usually go back to a commit that I think introduces the bug and manually test it, this manual tracing back method is okay is there are only a few commits and you are sort of sure where it went wrong, but if there are many commits in between the first bad commit to current commit, this manual way won't work.

While learning about binary search, I found that there is a git command `git bisect` which utilizes **binary search** under the hood, to find the first bad commit in less time. I won't be listing how binary search works here but I'll make a note of how to use `git bisect`.

Say you find that the latest commit is broken, and you know that 30 commits before the code works fine, and you want to know which commit introduces the bug:

```bash
git bisect start # to enter the bisect mode

# now tell git which version is a bad one
git bisect bad [a commit that has bug]
# now tell git which version is a good one
git bisect good [a commit that is working]

# now git will find a version in between these two commits and checkout that commit
# it will print out a message like below
bisecting: 6 revisions left to test after this (roughly 3 steps)
[commit revision] [commit message]

# now you can check if the current commit is good or bad and tell git
git bisect bad
# or
git bisect good

# git will keep trying to find the first bad commit
# once you find the commit you want to find, you can go back to the original state with
git bisect reset

# to make it more efficient, you can run a script for each revision to tell if it's good or bad
git bisect run rspec spec/features/my_broken_spec.rb
```

This is very handy for finding the first bad commit, and using binary search under the hood takes less time.
