---
title: Use Git to Undo Change and Rewrite History
categories:
- DevEnv
- Shell
tags:
- git
- rebase
- reset
- checkout
- cherry-pick
description: how to undo changes and rewrite history in Git
date: 2017-02-28
author_profile: true
classes: wide
---

# Background

I was working on a `feature` branch, which was branched off a `release-X` branch. For some unknown reason, there was some change in some minified dist-type files, I didn't know what happened and why these minified files(let's call them `micky-minified`) are not being git ignored, I wasn't paying attention at that time, and these changes were included along with the commit, let's call it `commitA`, in commitA, there were also some feature related code. Things were okay for a while, I kept working on the feature, added more commit(actually more commits than I should since I was trying different approaches to solve a problem, so the commit history is a bit messy), meanwhile, I merged other people's branch, let's call it `feature-sub-component`, also there were changes in the `release-X`, which I frequently pull and merge.

The issue was, when I finally create Pull Request(and it was a huge PR:() for the `feature` branch, I noticed that some minified files and some sourcemaps file were picked up in the diff(weird), not only this shouldn't happen in the first, but it also slowed down the BitBucket Pull Request page. After checking that these minified files are not relevant to the feature branch, I had to make the diff go away so that I can merge `feature` branch into `release-X` branch.

Since the diff I want to get rid of is minified files and sourcemaps files, my first thought is to check my git ignore file to see if it's because my git ignore file was incorrect. `.gitignore` file looked okay to me. That made me think why would there be a diff in sourcemaps files in the first place, it's being ignored as shown in the `.gitignore` file. So I thought I should update cached files in git:

```bash
git rm -r --cached .
git add .
```

I use these commands often when I am starting demo projects from scratch, I would just go ahead and write code, sometimes I get what I think is fine to push and then I realized I didn't put `gitignore` files(as soon as I typed it I realized this is not good practise at all even for demo projects). That's when I use these commands to remove what should not be tracked.

Okay, back to the problem, after I executed these commands in the project repo, shit happened, there was another folder, a folder I never used,let call it `folder-stranger`, it was being tracked as 'deleted', but I couldn't understand why, because there was nothing in the `.gitignore` file indicates this folder should be ignored, and it's some Java code I've never heard about. I got anxious because the last thing you want for a PR is to deal with some code nobody even knows about where it came from.

What I need to do is to 1) remove the commit with `folder-stranger` being deleted 2) fix the diff of `micky-minified` files, don't include them in the commit. So I tried to locate the commit where these `micky-minified` files were included, it was way back in the commit history, in the commit history of the current `feature` branch, there were commits made by me, merge commits from `feature-sub-component` and `release-X`, there were about 20 commits and the mistake commit was the fifth commit. Apparently I couldn't just use `git amend` or `git reset --hard` because I need to keep every commit except the `micky-minified` files in the fifth commit.

The first thing I could think of at that time is to rewrite commit history(which is not the best idea). It's not the situation of using `cherry-pick` for sure, it's not relevant in this case, I just like the name of it.

now let's see what other options we have: `git revert`: I don't really want to undo the mistake commit, because that `mistake-commit` contains both the wrong files and the files I need to keep. It seems that the only option is `git rebase`, because I thought one of the `git rebase -i` options can help in this case, but it didn't, you can squash, fixup or remove commits, but that's not what need.

It turned out I could just use `git checkout`, the reason behind it is that, I have some files got altered unintentionally and I don't want this change. What I did is 1) find the commit where the files got changed, 2) `git checkout [commit] /path/to/file.js` 3) after the files get reverted to the old version, add them and commit it again. This is the simplest solution in my case.

This made me think what kind features git has to change mistake commit and what are the use cases:

# git checkout

`git checkout` can be used with **branch**,**commit**,**file**, I use `git checkout` with branches often(`git checkout -b hotfix master`), but not the other two scenarios.

`git checkout [commit] [file]` is the one I used to fixed my problem, this command will turn the file into what is was like in that commit _AND_ add it to the staging area, so that you can add and commit it if necessary.

`git checkout [commit]` will match the whole working directory to the commit, this could be used to view an older state of the project without altering the current state, but if you are using IntelliJ or other IDEs you can use their native compare features to do this too.

# git revert

`git revert [commit]` is to undo a commit, and add this undo as another commit:

```
commit 07ef3b
    Revert "change index.html"
    This reverts commit 735d5432a945280e16519942ca71457c885b95ba.

commit 735d5
    change index.html
```

One thing to note before moving on to the next command is that `git revert` only undo one single commit(commit X), it doesn't undo or remove all subsequent commit of the single commit X. So `git revert` keeps the history and it only targets on one commit, unlike resetting removing all subsequent commits. In retrospect, I think using `git revert <mistake-commit>` could also work to some extent(I could revert that commit and do some fixup for the code I want to keep).

Be VERY careful or ABSOLUTELY sure what you're doing when you want to revert a merge commit, because "[Reverting a merge commit declares that you will never want the tree changes brought in by the merge. As a result, later merges will only bring in tree changes introduced by commits that are not ancestors of the previously reverted merge. This may or may not be what you want.](https://www.kernel.org/pub/software/scm/git/docs/git-revert.html)", and it's just more complex than revert one commit on one branch. However, if you really want to revert a *merge* commit, then command is a bit different, it should be [something like](http://stackoverflow.com/questions/7099833/how-to-revert-a-merge-commit-thats-already-pushed-to-remote-branch) `git revert -m 1 <commit-hash> `, otherwise the revert will fail.

# git reset

Unlike `git revert`, `git reset` should be used to used to undo **local** changes(whereas you can consider `git revert` as a way to *publicly* undo a change), don't reset snapshots other people are using.

so what does `git reset` do? A general idea is that if you use `reset` to undo changes in *staging area*, and if you use `reset --hard`, it will undo changes in both *staging area* and *working directory*.

`git reset [file]` will remove this file from staging area. I think it of as the counter command of `git add [file]`.

`git reset` will reset the *staging area* to match the most recent commit without touching *working directory*, I think it of as the counter command of `git add -A`.

`git reset [commit]`, similar to these reset commands above, this will make the staging area to match but not touching working directory.

`git reset --hard` and `git reset --hard [commit]` will reset both *staging area* and *working directory*. This is normally used when you are experimenting something and it has gone horribly wrong, now I realized that I shouldn't use this too often, because once you reset hard to a commit, and add new commits, git will think your local history has diverged, which will cause merging confusion to other developers who use the same branch, the bottom line is, don't use it on published changes.

## Another way to undo a merge commit that has been pushed to remote

As mentioned earlier, applying `git revert -m` to a merge commit can be tricky, let alone if it's already pushed(rule of thumb: once it's pushed to remote, be extra careful when manipulating history), but there is actually a simple way to undo the pushed merge commit:

```
git reset --hard [the-commit-before-merge]
git push [origin] [branch] --force
```

The `--force` push to origin will make the remote history the same as local, which is `[the-commit-before-merge]`, then you will "undo" the commit. However, make sure there is no commits after `the merge commit`, because they will be removed and you can not put them back again(maybe with `reflog` but don't do this).

# git clean

`git clean` is also kinda 'dangerous' command. It will remove untracked files from the working directory, and it not undoable(that's why I think it's also a dangerous move). Consider use `git clean` with `git reset --hard`. Assume the goal is to go back to a clean slate, use `git reset --hard` will affect tracked files, and use `git clean` to affect untracked files.

`git clean -n`, this will show you what will be removed but not actually remove it: `Would remove index copy.html`.

`git clean -f`, this will remove untracked files from the *current directory*, but not remove untracked folders or files specified by `.gitignore`.

`git clean -f [path]`, remove untracked files but limit that to a specific path.

`git clean -df`, remove untracked files and directories from current directory.

`git clean -xf`, remove untracked files in current directory and files git ignores.

# git commit --amend

This is the convenient way to fix up the most recent commit, I should use it more often because it happens everyday that you forget to add something in your commit.

`git commit --amend` will _replace_ the last commit with the new commit, and you can edit the commit message if necessary, otherwise it will just use the previous commit message. Since it replacing the most recent commit, it is altering the history, so don't do this to published commits. And this is exactly why if you try to amend a *already* pushed commit, git will ask you to pull in change before push. In that case, the best way is not modify published history, although if you can push the change by use `--force` if you have to.

# git rebase

Before going into the details of `git rebase`, I want to put a reminder here of when to use `git rebase`, the main purpose of it is to keep a linear/clean commit history when update with upstream changes. Say if you want to pull the latest change in an upstream branch into your local feature branch, you could do a `git merge`, however, it will also merge commits from the upstream branch into your feature branch, your feature branch commit history will be bloated with commit history, although I believe you could use different parameters of `git log` to see the actual commit on the feature branch, but still your history is a bit 'messy'. So what `git rebase` can do is to put your change on the tip of the new _base_.

`git rebase [base]`: the _base_ can be a branch or a commit.

If you are on a feature branch which is branched off master, and master has new commits while you add new commits on feature branch, now you want to base your work on the tip of the new master, you just use `git rebase master` on your new branch.

There are a few things to notice, one is that if there are conflicts, you can use 1) `git rebase --abort` to abort the rebase process 2) fix the conflicts and `git rebase --continue`, sometimes even if you fixed the conflicts and added it to staging, [it keeps asking you if you have everything added](http://stackoverflow.com/questions/22814602/cant-resolve-rebase-conflict), you can 3) `git rebase --skip`, but make sure there is nothing needs to be added.

There are also parameters like `git rebase --onto XXX XXX XXX`, I think it might over complicate the case, sometimes you could branch off the commit then use `git rebase [base]` in the [simple form](http://stackoverflow.com/questions/7744049/git-how-to-rebase-to-a-specific-commit).

For me, I want to avoid solving rebase conflicts but take advantage of it as much as I can, what I can think of is that, the Java code and JS code are in the same branch, if the new update in remote branch is just Java change then I can just rebase(for instance, use `>feature $ git log ..master` to see what commits are available on master but not on feature branch, or use `git pull --rebase` to keep a linear history), and if it's JS code updates and they might result in conflicts I can use the three-way merge. Let me try that and see if it works well.


# git rebase -i

`git rebase -i [base]` is also very handy, if you execute this command, it will list all the commits and give you options to 'fixup' or 'squash' or remove the commit, note that unlike `git log`, the commits history here is different, the most recent commits will be at the bottom.

If you have a local branch and you want to manipulate the commit history before push it to remote, you can use `git rebase -i HEAD~10`, you will see the latest 10 commits in the local branch(here is the [vim shortcut key](http://vim.wikia.com/wiki/Copy,_cut_and_paste) to cut and paste lines). I tried to use it and found a few things I can improve: 1) Do fast forward will help you to have a cleaner commit history, which means it's easier to change the commits in rebase console, you can use `git log --author="my Name"` to find out your commits 2) if you already know there will be conflicts because you and your coworker are collaborating on the same file, then *update frequently* and think twice before you *commit*, otherwise you'll have to manually solve the conflicts, it's not very fun though (good thing is IntelliJ has a solve conflicts feature which is very helpful for this), basically git rebase will tell you which commit can not be applied, and you can use `git status` to see which files are unmerged, then solve these conflicts, and after solving all conflicts, you have to `git add [files]`, then `git rebase --continue`. If anything went wrong and it's better to rewrite the history in other ways, you can use `git rebase --abort`, this will go back to the state before you started the rebase process 3) if you have something that is not fully done but you have to switch branches, use `git stash` instead of using commits.

# git cherry-pick

I have to say I like the name of this command. When can I use it? Say you commit something on the wrong branch, and now you switch back to the branch you actually need the commit, you can cherry-pick it from the wrong branch, that is, choose a commit from one branch and apply it onto another with `git cherry-pick [commit]`.

# git reflog

I didn't know `git reflog` when I was trying to fix the problem I mentioned at the beginning of this post, otherwise I won't need to create a backup branch(although it's I believe a safe practise to create a backup branch before rewriting commit history). Basically `git reflog` is the 'safty net', if you do a `git reflog` on your branch, you can see **everything** happened on this branch, and you can use the commit hash here to go back to a state.


## Links
[Atlassian tutorial](https://www.atlassian.com/git/tutorials)
[Deleting a git commit](http://www.clock.co.uk/blog/deleting-a-git-commit)
[Git remove commited file after push](http://stackoverflow.com/questions/18357511/git-remove-commited-file-after-push)
