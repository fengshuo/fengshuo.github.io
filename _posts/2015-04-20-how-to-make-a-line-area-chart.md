---
title: How to make a chart(line chart + area chart)
categories:
- Frontend
- Web
tags:
- d3
- chart
- dataVisualization
description: tutorial for implementing basic charts with d3.js
date: 2015-04-20
author_profile: true
classes: wide
---

Here is the [example](http://bl.ocks.org/mbostock/3883195) on d3.js site, and [this is my customized version](http://bl.ocks.org/fengshuo/37c0c3a81ba89fcbe97b).

The components of this chart include:

- a area chart
- a line chart
- an indicator group which contains a horizontal line, a vertical line, and a tooltip

# Data

The data is public [here](http://www.nasdaq.com/symbol/bidu/historical), the CSB file contains stock price & volumes of Baidu in the last 12 months.

Let's say what we want to present in this chart is the close value trend in the past year.

# Area Chart

Since the chart is originated from the plain area chart, let's start by implement an area chart first.

## Margin Convention
The [margin convention](http://bl.ocks.org/mbostock/3019563) wrote by Mike is very clear, admittedly this is not the only way to handle margins, but most of the time, following this convention can save you lots of troubles when the lines or areas are rendered.

## Scales and Axes

Apparently we need to make an x axe presenting time(or date in this case). If you've only seen a few d3.js tutorials, the chances are high that you don't know there is a dedicated api for time scale(`d3.time.scale`).

The date in our raw data is in the format of `2014-09-01`, let's create a range for x scale:

```js
var xScale = d3.time.scale()
              .range([0, width]);
```

However, we can only add the flexible domain to xScale after the data is loaded.

## Handle Raw Data

There are six columns in the origin data file, but for this chart, assume we only need to see the close value change over time. Are you thinking about open up the CSV file and edit it in Excel? No need, there is an accessor function in `d3.csv` which is very convenient for us to tweak origin data. In this case, we only need two columns: date and close value for that date.

```js
d3.csv("../data/BaiduNasdaq.csv",function(d){
    return {
      date:dateParser(d.date),
      close:+d.close
    }
  },function(err,data){
    // use this data
    })
```

After processing with accessor function, the data is ready to use.

## Date Parser

As seen in the accessor function, there is a dateParser function to process the date *string*, so what is this function for?  `var dateParser = d3.time.format("%Y/%m/%d").parse;`

You must remember that we use `d3.time.scale` for xScale in the previous step. For time scale, in fact, domain values are coerced to dates rather than numbers. Thus, we need to format the string in the csv.

One interesting thing about time format is that what `var format = d3.time.format("%Y-%m-%d")` returns is both an object and a function:

```js
format.parse("2011-01-01") // returns a Date
format(new Date(2011,0,1)) // returns a string
```

we will use both of them in this chart.

## Area Generator

After setting scales and axis, the most import step to draw the area chart is to create an area generator.

```js
var areaGenerator = d3.svg.area()
                    .x(function(d){
                      return xScale(d.date);
                      // remember to add Scale!
                    })
                    .y0(height)
                    .y1(function(d){
                      return yScale(d.close);
                      // remember to add Scale!
                    });
```

### Draw Area

The last step for rendering the area chart is to use the area generator. One thing to note is that, in this chart, there is only one area (and one line later), so we actually don't need use `d3.data()`. Let's use datum instead.

```js
svg.append("path")
  .datum(data) //use data for static, one time data bound
  .attr("d",areaGenerator)
  .attr("class","area");
```

## Tick Format

The date of xAxis may not be what you want, luckily it can be easily converted by time format:

```js
var xAxis = d3.svg.axis()
            .scale(xScale)
            .orient("bottom")
            .tickFormat(d3.time.format("%Y-%m"));
            // use time format for ticks
```

# Line Chart

As mentioned in the official document, area chart is can be used with line chart if we want to 'highlight' the edge of the area chart. The steps are similar with one difference, instead of using area generator, we need a line generator looks like this:

```js
var lineGenerator = d3.svg.line()
                    .x(function(d){
                            return xScale(d.date);
                    })
                    .y(function(d){
                            return yScale(d.close);
                    });
```

# Indicators

## Scatterplot

What we want to achieve by using indicators is that when users hovering on the data points, they will be able to see the tooltip with relevant text and also one horizontal line and one vertical line to indicate the value on x and y axe.

Note that we use `selection.datum` for line and area, if you look at the console, you'll see that there is two path element, one for line, the other for area. But if we want to achieve the interaction mentioned above, we need to sort of generate a scatterplot, which means this time, we need `selection.data`.


## Indicator lines

As for the indicator lines, I attempted to use `rect` initially, but it was not that applicable in this case since the very nature of rect is to grow from the top-left corner. So I turned into `lines`.

Personally I think it's better to group those annotation together in one place, that's why I created an `indicatorGroup` to save the tooltip, the lines. Another benefit of this is that when you add new line elements, you can nest selections within the indicator group. For example:

```js
d3.select(this).selectAll("line")
```

The selection is within the indicator group so that it won't affect other line elements on the chart.


## Tooltip

As usual, we use class to control the tooltip, and we can use time format here again for any time format output you prefer.

Although the position of the tooltip is not very flexible yet.
