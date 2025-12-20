---
title: The font-face Polyfill
categories:
- Frontend
- Web
tags:
- dev
- frontend
description: polyfill for font-face
date: 2014-12-15
author_profile: true
classes: wide
---

The polyfill syntax for `@font-face` would be something like:

```raw
@font-face {
  font-family: 'NOTDEF';
  src: url('NOTDEF.eot?#iefix') format('embedded-opentype'),  
       url('NOTDEF.otf')  format('opentype'),
	   url('NOTDEF.woff') format('woff'),
	   url('NOTDEF.ttf')  format('truetype'),
	   url('NOTDEF.svg#NOTDEF') format('svg');
  font-weight: normal;
  font-style: normal;
}
```


```raw
@font-face {
  font-family: 'MyWebFont';
  src: url('webfont.eot'); /* IE9 Compat Modes */
  src: url('webfont.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
       url('webfont.woff2') format('woff2'), /* Super Modern Browsers */
       url('webfont.woff') format('woff'), /* Pretty Modern Browsers */
       url('webfont.ttf')  format('truetype'), /* Safari, Android, iOS */
       url('webfont.svg#svgFontName') format('svg'); /* Legacy iOS */
}
```

It is likely that you only have one type of font file, that means you need to generate other formates like `woff`, `eot`, There are plenty of online font converters out there, but I've tried a few, this [web font generator](https://www.web-font-generator.com/) seems to be better. This is a [list](http://www.queness.com/post/14873/19-most-useful-font-face-generators-for-converting-fonts-to-web-safe-fonts) with other services.


## Links
- [1](http://css-tricks.com/snippets/css/using-font-face/)
- [2](http://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/)
