---
title: Responsive D3 Charts
categories:
- DevOps
- Containers
tags:
- d3
- responsive
- dashboard
description: Make d3.js charts responsive
date: 2015-08-19 22:48
author_profile: true
classes: wide
---

I've been working on a matrix which requires:

- responsive to window size to some extent;
- the whole chart can go full screen since there might be many details on the chart;

By far, I've mainly tried two approaches. The first one is to modify SVG attributes including `viewBox`, `preserveAspectRatio` and `width,height`. The second one is to bind listener to window resize event.

# SVG Attributes

## Width and Height

Specifying width and height is part of a basic routine as mentioned in [margin conventions](http://bl.ocks.org/mbostock/3019563), but in most examples on [bl.ocks](http://bl.ocks.org/), the width and height of the SVG are explicit numbers. But it is good to know that you can set these two attributes to *100%* in case the style of the chart container is responsive, and you want the chart in it to be responsive as well.

## ViewBox and PreserveAspectRatio

As for detailed explanation of `viewBox` and `preserveAspectRatio`, I highly recommend Joni Trythall's [A Look At preserveAspectRatio in SVG](http://codepen.io/jonitrythall/blog/preserveaspectratio-in-svg) and Sara Soueidan's [UNDERSTANDING SVG COORDINATE SYSTEMS AND TRANSFORMATIONS (PART 1) — THE VIEWPORT, VIEWBOX, AND PRESERVEASPECTRATIO](http://sarasoueidan.com/blog/svg-coordinate-systems/).

In the matrix chart I mentioned earlier, the SVG settings are like below:

```raw
var svg = d3.select(opts.selector).append("svg")
              .attr("width", "100%")
              .attr("height", "100%")
              .attr("viewBox", "0 0 " + width + " " + height)
              .attr("preserveAspectRatio", "xMinYMid meet")
              .attr("id", "matrixSVG");
```

One thing needs to point out is that the initial width and height of the viewBox is actually the initial full size of the SVG chart, which means when users switch to 'full screen' mode, the whole chart will smoothly expand to full screen.

The `preserveAspectRatio` attribute works well in some cases. The way I see it is similar to `background-size` attribute in CSS in that `meet` and `slice` in SVG feels like `contain` and `cover`. And if this attribute is set as `none`, the content in the SVG will try to fill the whole space without preserving any ratio, normally that's not what we want.

However, this attribute won't be that useful if the ratio of width and height isn't consistent. That's when JavaScript comes to rescue.

# Javascript Event

As a matter of fact, using `resize` event listener to implement responsive d3 chart is rather straightforward, and is indeed used quite often(e.g. [Responsive Charts With D3 And Pym.js](http://blog.apps.npr.org/2014/05/19/responsive-charts.html) and [Responsive D3 Charting](http://www.brendansudol.com/posts/responsive-d3/)).
Basically, you need to re-render charts once the container size change is detected(if performance is your concern, you can throttle it afterall).

As far as I can tell, using JS to handle responsiveness is handy since the chart itself is written in d3.js. But my major concern is performance, event listeners for resizing aside, what if there are tons of SVG nodes on the chart? What will happen if the data API cannot respond very well to frequent data requests? I would be happy to see more articles addressing these problems in near future.

## Links
- [Options for making D3 Responsive Charts](https://medium.com/@yo_johnnyd/i-feel-like-ive-been-googling-d3-responsive-charts-every-time-i-start-to-do-some-d3-work-and-id-bb652c28967e)
- [Integrating an interactive chart into a responsive web page](http://datatodisplay.com/blog/interactive-data-visualisation/integrating-interactive-chart-responsive-web-page/)
