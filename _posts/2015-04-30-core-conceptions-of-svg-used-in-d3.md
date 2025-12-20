---
title: Core Conceptions of SVG Used in D3.js
categories:
- Frontend
- Web
tags:
- d3
- dataVisualization
- svg
description: frequently used conceptions of SVG while using d3.js to build charts
date: 2015-04-30
author_profile: true
classes: wide
---

SVG is one of the fundamental parts of d3.js. It is very important to understand the basics of SVG since SVG shapes constitute these d3 charts.

There are a dozen of SVG tutorials out there, but in this post, I will focus on some core conceptions which are commonly used in d3.js.

# Coordinate System

In a normal mathematical coordinate system, the point x=0, y=0 is at the lower left corner of the graph. But in the SVG coordinate system, this (0,0) point is at the top left corner of the 'canvas', it is sort of similar to CSS when you specify the position to absolute/fix and use top and left to control the exact point of the element.

It is essential to keep in mind that as y increases in SVG, the shapes move down.

## Case: Y Axe Range

Let's say we're going to create a scatterplot with each point correspondent to a x value and y value. To scale the value, we need to set the domain and the range like this:

```js
d3.svg.scale()
  .range([0, height])
  .domain([0,max])
```

However, if you just keep the settings like this, the points will be based on the top horizontal edge instead of the bottom horizontal line as what we expected.

The nice thing about d3 is that you can easily change this by a simple adjustment in domain setting:

```js
d3.scale.linear()
  .range([height, 0])
  .domain([0, max])
```

With above code, the zero point of domain is correspondent to the height of the SVG, which is the bottom line of the chart in the viewer's eyes, meanwhile, the max value of the source data will be correspondent to the zero point of the SVG coordinate system, which the max value for viewers.

# The rect Element

`<rect>` represents rectangle, apart from aesthetic properties like stroke and fill, rectangle shall be defined by location and size.

As for the location, it is determined by the `x` and `y` attributes. The location is relative to the rectangle's parent. And if you don't specify the x or y attribute, the default will be 0 relative to the parent element.

After you specify the location, or rather the 'starting point' of the rect, the next thing is to specify the size, which is essential if you want to actually draw sth on the canvas, that is to say, if you don't specify the size attributes or the value is set as 0, you won't see anything on the canvas.

## Case: Bar Chart

Continue with the first scenario, the y axes, but this time, let's try to draw a bar chart.

Assuming the y scale setting is the same, the y axis is properly set as well, the only difference between the scatterplot and this bar chart is that, we need to specify the width and the height, particularly the height. To be more specific, we've already got the 'starting point', the rest is to use things like for the height:

```js
.attr("height", function(d){
  return (height - yScale(d.value))
  })
```


# The svg Element

`<svg>` element is the root element, or the canvas as we are drawing charts on it.

SVG elements can be nested inside each other, and in this way, SVG shapes can be grouped together, meanwhile, all shapes nested inside an `<svg>` element will be positioned relative to its enclosing `<svg>` element.

One thing might need mentioning is that, we can't nest `<rect>` inside another `<rect>`, it won't work.

## Case: Multiple Charts

For example, this [multiple donuts chart](http://bl.ocks.org/mbostock/3888852) is made up by multiple `<svg>` elements, which contains a donut chart respectively. This can be achieved by using the `<g>` element, but in this case where we only want to put donut chart one by one next to each other, `<svg>` is more convenient.

One thing to bear in mind is that we can't use `transform` attribute on `<svg>` nevertheless we can use `x`, `y` for position.

# The g Element

`<g>` element is used to group SVG shapes together, and it is very useful when you want to transform the group of shapes as if it was a single shape whereas you can't use `transform` on `<svg>` element. Besides, the styling of g elements are inherited by its children.

`<g>` element is commonly used in d3 chart, for example, the ticks of the axes, the margin convention, etc.

## Case 1: Margin Conventions

It is convenient to use [margin conventions](http://bl.ocks.org/mbostock/3019563) for d3 chart, because we don't need much effort to keep these margins in line later.

## Case 2: Pie Chart

The other scenario is that [multiple donuts chart](http://bl.ocks.org/mbostock/3888852), if we look closely at each `<svg>` element, we can see that there is a `<g>` element with `transform` attribute. Specifically since the arc generator will draw the pie chart based on the (0,0) point, if we don't transform the `<g>` element, 3/4 of the pie chart will be blocked.


# The text Element

The position of the text is determined by the `x` and `y` attributes (the `y` value determines the horizontal location of the bottom of the text). Some other attributes are commonly used in d3 charts include:

## Text Anchor
This attribute determines which part of the text is positioned at the `x` position.

## Rotate
We can also use `transform` attribute on `<text>` element, which means we can rotate the text, particularly, a single value results in each word rotating at that value while a string of values can be used to assign different rotation value to each letter.

## The `<tspan>` Element
SVG doesn't currently support automatic line breaks or word wrapping, that's when `<tspan>` comes to rescue. `<tspan>` element positions new lines of text in relation to the previous line of text. And by using `dx` or `dy` within each of these spans, we can position the word in relation to the word before it.

## Case: Annotation on Axes

For example, when we want to add an annotation on y Axe:

```js
svg.append("g")
      .attr("class", "y axis")
      .call(yAxis)
    .append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 6)
      .attr("dy", ".71em")
      .style("text-anchor", "end")
      .text("Temperature (ºF)");
```

# The path Element

`<path>` element is used to draw irregular shapes. To draw any shape by `<path>`, the path data is needed as `<path d="[path data]">`.

## Case: Arc and Stacked Chart
For example, when we use arc generator with pie layout, this is one of the results:

```js
<path d="M-239.7935922865823,9.951537484044003A240,240 0 0,1 -25.080788568417415,-238.68588991556737L-18.81059142631306,-179.01441743667553A180,180 0 0,0 -179.84519421493673,7.463653113033002Z" style="fill: rgb(208, 116, 60);"></path>
```
