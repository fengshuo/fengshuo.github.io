---
title: Start A Cronjob (with Bash and Python)
categories:
- DevEnv
- Shell
tags:
- cronjob
- linux
- bash
- python
description: start a cronjob on Mac
date: 2015-04-08
author_profile: true
classes: wide
---

# What's the Scenario

For some reason, I need to send a post request to a particular url to get a ticket string, and to update this ticket string in a configuration file periodically so that I can work on this project.

The solution is pretty self-explanatory. For starters, I need to send request and save/update the string in the configuration file, which is implemented by python. And most importantly, I need to start a cronjob to run the python script every 30 mintues.

# Cronjob

I'd like to address cronjob first, because you can alter to node to do the http and file IO stuff, and the python code is just something came out of quick API search.

So for now, I assume the python and bash scripts are ready, and the only thing I need to do is to create a cronjob to run the bash script.

## Cronjob Description

Run `env EDITOR=nano crontab -e` to open the nano editor, write something like this:

```raw
*/30 * * * * sh /Users/myname/Documents/myproject/ticket-cronjob/ticketHelper.sh >> /Users/myname/Documents/myproject/ticket-cronjob/cronjob-ticket.log 2>&1
```

- `*/30 * * * *` means run this job every 30 minutes since the first place is a minute descriptor, however, if you write something like `10 * * * *` that means run this job every hour at the 10th minute;

- `sh` means run this script as `.sh` file, followed by the file path of the sh script;

- the rest means to redirect the cronjob log to another place

Then press `ctrl+o` and press enter to save the file, `ctrl+x` to close nano.

## Check Status

You can use `tail -f /Users/myname/myproject/ticket-cronjob/cronjob-ticket.log` to monitor real time content of the log file.

Besides, you can also type `crontab -l` to list all cronjobs, and `crontab -e` to read and edit cronjob descriptions.

## Remove cronjob

Regardless of the fact that cronjob will stop when the laptop goes to sleep, there will be a time when you want to stop the cronjob, use `crontab -r` then the cronjob will be deleted, but it is not as good as pause and resume, which I'll look into later.


# Python and Bash

## Python

The python script for requesting new ticket and updating the file is not difficult, but I am not that familiar with python yet, so I won't paste the code here except noting that it uses packages such as `requests` and `re`.

## Bash

As a matter of fact, in this case, it is not necessary to use bash to execute the python script since you can directly run python script in cronjob, but I'd like to give it a try anyway.

```raw
#!/bin/bash

# echo "pwd: "
# echo "\$0: $0"
# echo "basename: `basename "$0"`"
# echo "dirname: `dirname "$0"`"

python "`dirname "$0"`/getTicket.py"
```

The first line is a shebang, it tells the shell what program to interpret the script with when executed.

To make the script more flexible, I change the absolute path with `dirname` so it is executable on both local folder and cronjob.

Lastly, you need to execute a command like this `chmod u+x myscript.sh` to make this script executable.

## Links
[Creating CronJobs](http://www.maclife.com/article/columns/terminal_101_creating_cron_jobs)
