---
title: A Quick Introduction to Systemd
categories:
- DevEnv
- Shell
tags:
- systemd
- linux
- unix
- init
- booting
- systemctl
- journalctl
description: A Quick Introduction to Systemd
date: 2019-02-26
author_profile: true
classes: wide
---

## What is Systemd

### The Linux Boot Process

Before talking about systemd, we need to have a basic understanding of the linux boot process. When you start the server, BIOS will check hardware/input and output first, the next step is to select booting device, load bootloader, it will determine which kernel to load. Once the kernel is loaded, the system initiation process will run, it will execute start up script and run the operating system. Systemd is used during the system initiation process to start up the system and also manage the system.

### The Purpose of Systemd

Before systemd, there was system V and Upstart, but some advantages of systemd are: it makes booting process faster, and its ability to delay a unit startup until it is absolutely needed, it does not just operate processes and services, it can also mount filesystems, monitor network sockets, run timers, etc. Many distributions have adopted systemd.

## Unit Files

Systemd has many capabilities, each type of capability is called a unit type, and each specific capability is called a unit. Some common unit types are:

- service units: control traditional service [daemons](https://en.wikipedia.org/wiki/Daemon_(computing)) on a Unix system
- mount units: control filesystem attachment
- target units: control other units, usually by grouping them

The default boot goal is usually a target unit that groups together a number of service and mount units as dependencies.

There are two main locations for systemd configuration, the first one is usually `/usr/lib/systemd/system` (this is called *system unit directory*, avoid editing those unit files), the other one is usually `etc/systemd/system` (this is called *system configuration* directory, make local changes to this directory).

Take the `nginx.service` file for example, it looks like below: (after installing nginx, run `systemctl cat nginx.service` to see the file content)

```bash
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target
[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true
[Install]
WantedBy=multi-user.target  
```

(Note the last section install, it says this service unit is grouped in the multi-user.target)

## Systemd Operation

The primary way to interact with systemd is through `systemctl` command.

### Controll Services

```bash
systemctl enable httpd.service
systemctl disable httpd.service
systemctl start httpd.service
systemctl stop httpd.service
systemctl restart httpd.service  
```

Enabling a unit is different from activating a unit, when you enable a unit, it means the unit will autostart at boot time (by creating a symlink):

```bash
[root@xxx ~]# systemctl enable nginx.service
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.  
```

Enabling the unit does not mean activating it, you will have to use `start` to start the unit immediately.

After you modify the unit file, you should either restart it or reload with `systemctl reload application.service` , or if you are unsure whether the service has the functionality to reload its configuration, you can run `systemctl reload-or-restart application.service`.

### Check status

Use `systemctl status application.service` to find out the status of the service, for example with `httpd.service`:

```bash
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-02-23 19:07:22 UTC; 2h 54min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 4055 (httpd)
   Status: "Total requests: 1; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─ 4055 /usr/sbin/httpd -DFOREGROUND
           ├─ 4075 /usr/sbin/httpd -DFOREGROUND
           ├─ 4076 /usr/sbin/httpd -DFOREGROUND
           ├─ 4077 /usr/sbin/httpd -DFOREGROUND
           ├─ 4078 /usr/sbin/httpd -DFOREGROUND
           ├─ 4079 /usr/sbin/httpd -DFOREGROUND
           └─27145 /usr/sbin/httpd -DFOREGROUND
19:07:22 shuof3c.mylabserver.com systemd[1]: Starting The Apache HTTP Server...
19:07:22 shuof3c.mylabserver.com systemd[1]: Started The Apache HTTP Server.
```

Some other quick way to check the status:

- `systemctl is-active application.service`
- `systemctl is-enabled application.service`
- `systemctl is-failed application.service`

### Other Useful Commands

To see all active units that systemd knows about use `systemctl list-unit-files` (every available unit file, including those that systemd has not attempted to load) or `systemctl list-units` (only displays units that has been parsed and loaded)

You can add filters, such as `systemctl list-units --type=service` to see service units.

### Mask and Unmask Units

Systemd can also mark a unit as completely unstartable either manually or automatically, with `mask`

```bash
systemctl mask nginx.service
systemctl unmask nginx.service
```

## View Logs

To see the logs of systemd units, use `journalctl`, it has some useful parameters such as:

- `-f` to tail the log
- `-u` to show logs for specific service
- `-r` to show the latest log first
- `-e` to show the end of the log
- `-o verbose` or `-o json-pretty` to format the output

For instance, you might need to tail a specific service log to debug with `systemctl -f -u nginx.service`.
