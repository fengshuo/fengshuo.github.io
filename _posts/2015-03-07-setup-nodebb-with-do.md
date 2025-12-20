---
title: Setup NodeBB with DigitalOcean and Nginx
categories:
- DevEnv
- StaticSite
tags:
- nginx
- digitalocean
- nodebb
description: Setup NodeBB with DigitalOcean and Nginx
date: 2015-03-07
author_profile: true
classes: wide
---

Suppose you have a homepage site 'www.xyz.com(xyz.com)' is hosted by Github with Jekyll, but a few days later, you realized that you need a forum, such as 'forum.xyz.com' to post stuff within the community.

Here is what we need to do(high level):

1. Setup [NodeBB](https://github.com/NodeBB/NodeBB)
2. Configure Nginx on sever
3. Configure subdomain to point to the server with NodeBB

## Setup NodeBB

You can install NodeBB by following this [tutorial](http://burnaftercompiling.com/nodebb/setting-up-a-nodebb-forum-for-dummies/) along with the [official document](https://docs.nodebb.org/en/latest/index.html).

(Note: While installing NodeBB, you will be prompted with questions like Admin username or Admin password, don't forget to set administrator info here).

You won't need to do any customized configuration yet, just follow the steps i the tutorial, if everything is right, you should be seeing [your server's IP address]:4567 with nodeBB.(say 192.123.222.111:4567, 4567 is the default port for nodeBB).


## Configure Nginx

Typically you don't want visitors to see the port, so let's [use nginx](https://docs.nodebb.org/en/latest/configuring/proxies.html) as a proxy.
### Install Nginx

See [tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-14-04-lts) here.

Remember you have to use a regular, non-root user with `sudo` priviledges for this.

## Configure Nginx

### Update nginx.conf

This file is normally located in `/etc/nginx` in Ubuntu.

Add a upstream section in http section like this:

```raw
http {

  #....default settings

	include /etc/nginx/conf.d/*.conf;
  #not necessary
  #include /etc/nginx/sites-enabled/nodebb.conf;
	include /etc/nginx/sites-enabled/*;

  upstream nodebb {
    #update with your serve ip address with port
    server 111.222.333.444:4567;
  }
}
```

Note, you might need to run `sudo vim nginx.conf` instead of `vi nginx.conf` due to potential permission issues.

### Add conf file

Now let's create a `nodebb.conf` file in `/etc/nginx/sites-enabled` with below content:

```raw
server {
    listen 80;

    server_name forum.xyz.com;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;

        proxy_pass http://127.0.0.1:4567/;
        proxy_redirect off;

        # Socket.IO Support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Test and Restart

[Test the nginx](https://community.nodebb.org/topic/3375/nginx-proxy/4) config after changes(as root):  `nginx -t`

and [reload config](http://www.cyberciti.biz/faq/nginx-linux-restart/):  `service nginx reload`

### Update Config.json in NodeBB

Lastly, you need to update `config.json` in the root of nodeBB you just installed:

```raw
{
    "url":"http://forum.xyz.com",
    "secret": "xxxxxxxx",
    "database": "redis",
    "port":"4567",
    "bind_address":"0.0.0.0", //new
    "redis": {
        "host": "127.0.0.1",
        "port": "6379",
        "password": "",
        "database": "0"
    },

    "use_port":false //don't forget this setting

}
```

If everything looks fine, you should use [forever](https://www.npmjs.com/package/forever) tp run the script continuously. In your nodeBB root director:  `npm install -g forever`  then  `forever start app.js`

Also, you can install `supervisor` to restart node app while debugging:  `npm install -g supervisor`  and `supervisor app`.


## Setup Subdomain

The last step will be create a subdomain for this nodeBB forum.

Let's point subdomain to another web hosting(digital ocean this time, instead of the homepage hosted by Github).

I use Godaddy for domain management, but I assume the steps are similar among other services.

In the setting page of www.xyz.com:

1. click the DNS zone file tab
2. click Add Record
3. add a `A(host)` type record, the host is 'forum.xyz.com', points to your digitalocean server address.
4. save the change.


Give it a couple of minutes, and visit http://forum.xyz.com.
