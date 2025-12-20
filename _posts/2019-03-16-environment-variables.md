---
title: Environment Variables
categories:
- DevEnv
- Shell
tags:
- environment variables
- shell variables
- process
- operating system
- command
description: understanding environment variables, shell variables, the difference
  between them and how to unset
date: 2019-03-13
author_profile: true
classes: wide
---

# Environment Variables

# Introduction

When a process starts, its environment is inherited from its parent. It has the command  line arguments used to invoke the running program, and it has a list of key-value pairs environment variables, however the *environment* is not held in kernel memory but is stored in the process's own user-mode address space. That means every shell or program or every terminal window has its *own distinct copy* of the environment that can be separately modified. We will see it in demonstration later.

# Environment Variable VS Shell Variable

Every time a shell session spawns, a process takes place to gather information that should be available to the shell process and *its child processes*. As mentioned earlier, there is a list of key-value pairs, it looks like:

    KEY=value
    # multiple values
    KEY=value1:value2
    # if the value contains white space, use quotations
    KEY="hello world"
    # if there are special characters use single quotes
    KEY='hi there!'

Note that there is no space near the `=` and the convention is to use upper case for the key name.

The major difference between environment and shell variable is that environment variable are available for both the current shell AND it's child shell/processes. But shell variables are only available within the shell in which they are set.

Before we see the difference in action, let's take a look at those variables first.

To see all environment variables, you can use `printenv` or `env` , they are basically similar. To see shell variables, we can use `set`, although if you run `set` without parameters, it will list all shell variables, environmental variables and shell functions, so usually you can use `set | less` or `set | grep myenv` to find the information you need.

# Set Variables

We often need to set environment variables such as third party tool api token, now let's set shell variables first:

    TEST=123

now check if it's in the shell variable with:

    echo $TEST
    # gets 123

Since shell variables are only available to where it gets define, we can verify that by spawn a new shell with `zsh` or `bash`

    bash
    # now in a new shell
    echo $TEST
    # nothing gets printed

    # now let's get back to the previous shell
    exit
    echo $TEST
    123

We can also verify that it is just a shell variable not environment variable by running:

    printenv | grep TEST
    # no result

Let's try setting environment variables now with `export`

    export ZTEST='hello'
    # check if it's in environment variable
    printenv | grep ZTEST

Since environment variables will be inherited by it's child processes, let's spawn a new shell:

    bash
    echo $ZTEST
    # will print hello
    exit
    echo $ZTEST
    # still prints hello

If you do `printenv` now, you should see `ZTEST`

However, since every terminal window has its *own distinct copy* of the environment that can be separately modified, let's open a new terminal window or tab, try `printenv` again and `ZTEST` won't be there.

Another tip is when you are making changes to important variable, it is safer to save a copy of the value first, then make the change:

    echo $HITSIZE> ~/valueOfHITSIZE.txt

# Set Environment Variables at Login

Usually we want to have important variables up every time we start a new shell session, the way to achieve that is to edit files such as `~/.bashrc` or `~/.zshrc` :

    # in ~/.bashrc or ~/.zshrc
    export ZTEST=123

Save the change and the next time you start a new shell session, those env var will be read, or you can force your current session to read the file, do:

    source ~/.zshrc
    # or ~/.bashrc

Now the `ZTEST` is available in new shell sessions you started. This is how you can make variable changes 'permanent'.

# Unset

What if you want to remove or disable an environment variables?

For instance, if you have already set an api token ZKEY in `~/.bashrc` or `~/.zshrc` , but you don't want to use it for a special request, you can use:

    unset ZKEY

It will remove ZKEY in the current environment, which means it is still available in other terminal window.

# Import Environment Variables

One of the most important variables is the `PATH` variable, it controls where on your system your shell will look for commands you are trying to run, such as `cd`, `ls`, etc. Basically it's a list of directories that the system will check when looking for commands:

    echo $PATH
    /usr/local/bin:/usr/bin:/bin
    # most comands are locaed in the sbin or bin subdirectory

For example, after I install [rbenv](https://github.com/rbenv/rbenv#how-rbenv-hooks-into-your-shell), the PATH looks like `/Users/shuo/.rbenv/shims:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin`

The directories in `PATH` are searched from left to right, so a matching executable in a directory at the beginning will precede another one at the end.

The way to add new value to the `PATH` variable looks like:

    PATH=$PATH:/a/new/path

    # instead of PATH=/a/new/path, this will make your
    # PATH variable only contain the a/new/path

# Links

[How To Read and Set Environmental and Shell Variables on a Linux VPS](https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-a-linux-vps)
