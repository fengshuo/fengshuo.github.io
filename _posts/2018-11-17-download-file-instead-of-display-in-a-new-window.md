---
title: Download File Instead of Displaying in A New Window
categories:
- Frontend
- Web
tags:
- AWS
- HTML5
- a
- download
- attachment
- S3 bucket
description: When opening a new file link, download the file instead of open a new
  tab or window
date: 2018-11-17
author_profile: true
classes: wide
---

I was working on a small feature when user click on a button, download a file directly but not open it in a new tab, the button looks like this:

```
<a href="/media/example.pdf">button</a>
```

The default behaviour is that the browser will open the pdf file in the tab or a new tab if you add `target="_blank"`, but the behaviour I want is to download this file to user's laptop directly.

The first approach I tried is to use the [download](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a) HTML5 attribute of the anchor element:

```
<a href="/media/example.pdf" download>button</a>
<a href="/media/example.pdf" download="AnotherFileName.pdf">button</a>
```

This is straightforward, however if there is no anchor element on the page to include the download attribute, you might need to add a few lines of JavaScript to make it happen:

```
const save = document.createElement('a');
if (typeof save.download !== 'undefined') {
  // if the download attribute is supported, save.download will return empty string, if not supported, it will return undefined
  // if you are using helper method, such as isNone in ember, you can also do isNone(save.download)
  save.href = fileURL;
  save.target = '_blank';
  save.download = fileName;
  save.dispatchEvent(new MouseEvent('click'));
} else {
  window.location.href = fileURL; // so that it opens new tab for IE11
}
```

However this download attribute only works for same origin URLs, what if this static file is saved somewhere else once deployed (such as AWS s3 bucket)? In this case, the download attribute won't work.

But an interesting thing I learned is that if the file is saved in S3 bucket, then you can change the property of file to download the file.

There is a HTTP Header [Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition):

> In a regular HTTP response, the Content-Disposition response header is a header indicating if the content is expected to be displayed inline in the browser, that is, as a Web page or as part of a Web page, or as an attachment, that is downloaded and saved locally.

Go to the S3 bucket, click on properties of the file, click on metadata, then add a new key-value pair with key `Content-Disposition` and value `attachment`

![s3-bucket-attachment](/assets/images/content/s3-bucket-attachment.png)

Then in the Front End code, you don't even need the anchor element, visiting the link will download the file `window.location.href = 'http://s3.amazon.com/something.pdf';`
