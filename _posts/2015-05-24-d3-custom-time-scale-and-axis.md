---
title: Custom Time Scale and Axis in D3
categories:
- Frontend
- Web
tags:
- d3
- dataVisualization
description: Customized time scale, axis in D3
date: 2015-05-24
author_profile: true
classes: wide
---

When it comes to time series data, it often requires customization of the time scale and axis. Fortunately, d3.js provides a `d3.time.scale` api, which is an extension of d3.scale.linear that uses JS Date objects as the domain representation.

# Customize Time Format

The default time format used by d3.time.scale is implemented as following:

```js
var format = d3.time.format.multi([
  [".%L", function(d) { return d.getMilliseconds(); }],
  [":%S", function(d) { return d.getSeconds(); }],
  ["%I:%M", function(d) { return d.getMinutes(); }],
  ["%I %p", function(d) { return d.getHours(); }],
  ["%a %d", function(d) { return d.getDay() && d.getDate() != 1; }],
  ["%b %d", function(d) { return d.getDate() != 1; }],
  ["%B", function(d) { return d.getMonth(); }],
  ["%Y", function() { return true; }]
]);
```

So how does this work? For every Date object passed into it, it examine this object by the predict function in each Array, the first predict function returns true will be the determinant of the date format, that is, the specifier.

For Example, the date object `new Date(2015, 3, 1)`, if you getMilliseconds, getSeconds, getMinutes, getHours of this Date object, it return 0, and it will return getDate(), it return 1, but the default setting wants to display only the month name if it is the first day of the month, so d.getMonth() will return 3, thus the specifier `%B`, that is, the full month name. You can modify the predicate function to show the first day of the month. But don't forget to pass the custom format to axis like this:

```js
var xScale = d3.time.scale()
		    .domain([start, end]) // use time scale, the domain needs to be Date Object
		    .range([0, width]);

var xAxis = d3.svg.axis()
		    .scale(xScale)
				.orient("bottom")
         .tickFormat(customTimeFormat)
```

# Customize Time Ticker

I'd say the default time format is applicable for most common case, but there is a time when you need more than that. For example, a time axis with a 9 days gap between ticks. That's when `d3.time.interval` comes to rescue, such as `d3.time.day`, `d3.time.hour`, and `d3.time.sunday` , alias such as `d3.time.hours`, `d3.time.days` etc.

In this case, the automatically generated ticks won't be what we need, therefore we have to generate the ticks by ourselves using `tickValues`, this will override the default tickValues

```js
var start = new Date(2015, 3, 1);
var end = new Date(2015, 4, 1);

var xAxis = d3.svg.axis()
          .scale(xScale)
          .tickValues(d3.time.days(start, end, 25));

// or chain with filter to implement more control
//.tickValues(d3.time.days(start,end,5).filter(function(d,i){
//  return d.getDate() !== 1
//}))
```

**Links:**
1. [Custom Time Format](http://bl.ocks.org/mbostock/4149176)
2. [D3 time scale axis ticks are irregular on year boundaries](http://stackoverflow.com/questions/21296968/d3-time-scale-axis-ticks-are-irregular-on-year-boundaries)
3. [Time Scale Always Forcibly Display First Date of A Month](http://grokbase.com/t/gg/d3-js/1344en8cz8/time-scale-always-forcibly-display-first-date-of-a-month)
4. [all about D3 time scales](http://bl.ocks.org/jebeck/9671241)
5. [ggplot2-Style Axis](http://bl.ocks.org/mbostock/4349486)
