---
title: Encoded Url in Browser Address Bar
categories:
- frontend
tags:
- dev
- fix
- frontend
description: encoded url in browser
date: 2014-08-11
blog: true
author_profile: true
classes: wide
---

Lately I was sending GET request to an address with four parameters at least, there is nothing special except three points I'd like to address here:

# Encode Query String

In javascript, there are 2 pairs of methods to handle string encode and decode:
`encodeURI, decodeURI`, and `encodeURIComponent, decodeURIComponent`.

However, it bugs me that the `encodeURI` method *does not* encode some very common characters like `&` or `=`(the rest are `; , / ? : @ + $`), this makes using encodeURI to handle query strings at one time impossible, so basically I use `encodeURIComponent`(which does encode & or =) to handle query string encoding while assemble the whole URL.

There is one key-value pair `someUrl="http://www.abc.com/x=1&y=2"`, it is a little bit different than key-value pairs like 'x=1&y=2', if you don't use encodeURIComponent on this one:

```js
// the result expected would be:
http://www.example.com/index.html?someUrl="http://www.abc.com/x=1&y=2"  
// the actual result would be sth like this
http://www.example.com/index.html?someUrl="http://www.abc.com/x
```

# Long url in address bar

It is likely that the encoded query value is very long (especially there is Chinese characters in query), although according to RFC, there is no limit to the length of an URI, but limitations exist in various browsers, generally speaking, URL over 2000 characters will not work.

# GET v.s. POST request

After all, I think it would be much better to avoid really long query string in URL by sending GET request, in that case, send data in POST request body would be a better choice.

## Links

[1](http://unixpapa.com/js/querystring.html)
[2](http://getpocket.com/a/read/57489195)
[3](http://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers)
[4](http://technomanor.wordpress.com/2012/04/03/maximum-url-size/)
