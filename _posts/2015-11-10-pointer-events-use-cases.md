---
title: CSS Pointer-events User Cases
categories:
- Frontend
- Web
tags:
- pointer-events
- SVG
- css
description: Three user cases of css pointer-events
date: 2015-11-10 22:48
author_profile: true
classes: wide
---

# SVG Mouse Events

(Update: the codepen for this post was deleted)

Background: In the first svg, I grouped a star and text together, and I intended to add mouse hover event to this group as a rectangle just like a div.

The first thing is nothing happens if I move cursor into the rectangle area but not the shape or text. This is because I mistakenly perceive `<g>` as something similar to `<div>`, that is, everything inside the group will expand the area the group, but actually `<g>` just **logically** group different svg elements together, there is no actual shape rendered for `<g>`.

The second thing is that even if the cursor is hover onto the star shape, nothing happens. The reason behind this is that, in SVG content, the default value for `pointer-events` is `visiblePainted`, in my case, the `fill` of the star is set to `none`, thus the pointer events is not captured. To fix this, we can either set `pointer-events: all` or `pointer-events: fill`, that way the values of `fill` and `visibility` [won't affect events processing](https://developer.mozilla.org/en-US/docs/Web/CSS/pointer-events).
However, this works on most modern browsers, but it won't work on IE9 or IE10, the workaround I used is to set `fill: transparent` so that there is nothing rendered visually and the svg element can react to pointer events.


# Custom Select

Another use case of `pointer-events` is to implement a customized select.

If `appearance: none` is working as we want, we usually need to add some custom arrow to complete this customized select, and set the custom arrow to `pointer-events: none`, so that the element underneath it (which should be the select element) will catch the event as you can see in this [post](http://red-team-design.com/making-html-dropdowns-not-suck/).

Generally speaking, the prime use case for pointer-events is to allow click or tap to “pass through” an element to another element below it on the Z axis.

# 60fps Scrolling

Also, as described in this [article](http://www.thecssninja.com/javascript/pointer-events-60fps), `pointer-events` can be helpful to enhance performance while use scrolling the web page.
