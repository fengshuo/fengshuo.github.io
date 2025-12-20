---
title: Integrate D3 with React
categories:
- Frontend
- Web
tags:
- react.js
- d3.js
- component
- data visualization
description: Document the react.js, d3.js integration experiment
date: 2016-11-05 20:20
author_profile: true
classes: wide
---

I remember a while ago (when React.js just started to get popular), I was wondering if it's easy to combine the power of d3.js and React.js, I did some searching and realized that it's a bit tricky since both of them will control DOM and track differences. Since I was working on some highly customized data visualization with d3.js, it didn't seem like a good idea at the time to add new layer to the project.

Then things changed(just like every other day in front end world), d3.js released a new major version, Redux became popular, I've been using Redux for a while and it is indeed a huge improvement compared to the time when you need to bind data onto DOM or put it in localStorage. Also, there are some great open source library based on React+D3, I would say if the majority charts you need to implement in a react project are standard, say bar chart or line chart, then these libraries should suffice. But I am still not sure about the right approach to integrate react and d3 for customized data visualization.

To understand more about the differences/similarities between d3.js and react.js, [this post](https://medium.com/@sxywu/on-d3-react-and-a-little-bit-of-flux-88a226f328f3#.qwv5axbef) is a nice start point, the author also has tried three different approaches [here](http://bl.ocks.org/sxywu/61a4bd0cfc373cf08884). The use case I am looking into is customized presentation and potential large amount of data, so approach 3 seems to be the best fit, although that means using react is actually redundant since all the key update/manipulation is done by d3.js. Here is a really simple example I tried based on [d3 in action edition 2 code example](https://github.com/emeeks/d3_in_action_2/blob/master/chapter9/reactd3/src/BarChart.js) and [this post](http://nicolashery.com/integrating-d3js-visualizations-in-a-react-app/), code is [here](https://github.com/fengshuo/react-d3-simple-barChart):


```js
import React, { Component, PropTypes } from 'react';
import { scaleLinear, scaleBand } from 'd3-scale'
import { max } from 'd3-array'
import { select } from 'd3-selection'
import { transition } from 'd3-transition'
import { axisLeft, axisBottom } from 'd3-axis'

class BarChart extends Component {
	static PropTypes = {
		width: PropTypes.number,
		height: PropTypes.number,
		data: PropTypes.array,
	}

	componentDidMount() {
		const node = this.node;
		const margin = this.props.margin;
		const canvasWidth = this.props.width - margin.left - margin.right;
		const canvasHeight = this.props.height - margin.top - margin.bottom;
		select(node).attr("transform", "translate(" + this.props.margin.left + "," + this.props.margin.top + ")")
						.attr("class", "canvas");

		this.renderChart();
	}

	componentDidUpdate() {
		this.renderChart()
	}

	renderChart() {
		const node = this.node;
		const data = this.props.data;
		const canvasSize = this.getCanvasSize()

		// scale and axis
		const xScale = scaleBand().rangeRound([0, canvasSize.width]).padding(0.1)
					.domain(data.map(function(d) { return d.type; }));

		const yScale = scaleLinear().rangeRound([canvasSize.height, 0])
					.domain([0, max(data, function(d) { return d.value; })]);

		select(node)
      	.selectAll(".axis.axis--x")
      	.data([0])
      	.enter()
      	.append("g")
      	.attr("class", "axis axis--x")
      	.attr("transform", "translate(0," + canvasSize.height + ")")
      	.call(axisBottom(xScale));

      	select(node)
	    .selectAll(".axis.axis--x")
	    .transition()
	    .duration(1000)
	    .call(axisBottom(xScale));

  		select(node)
      	.selectAll(".axis.axis--y")
      	.data([0])
      	.enter()
      	.append("g")
      	.attr("class", "axis axis--y")
      	.call(axisLeft(yScale))

      	select(node)
	    .selectAll(".axis.axis--y")
	    .transition()
	    .duration(1000)
	    .call(axisLeft(yScale))

		// rect
		let rects = select(node)
	      .selectAll("rect.bar")
	      .data(this.props.data)

	     rects.enter()
	      .append("rect")
	      .attr("class", "bar")
	      .attr("x", function(d) {
			return xScale(d.type)
			})
		  .attr("y", function(d) {
			return yScale(d.value)
		  })
		  .attr("width", xScale.bandwidth())
		  .attr("height", function(d) {
			return canvasSize.height - yScale(d.value)
		  })
		  .style("opacity", 0)
		  .style("fill", "steelblue")
		  .transition()
		  .duration(1000)
		  .style("opacity", 1)

	      rects.exit()
	      .remove()

	    rects
	      .transition()
	      .duration(1000)
	      .attr("x", function(d) {
			return xScale(d.type)
			})
		  .attr("y", function(d) {
			return yScale(d.value)
		  })
		  .attr("width", xScale.bandwidth())
		  .attr("height", function(d) {
			return canvasSize.height - yScale(d.value)
		  })


	}


	getCanvasSize() {
		const margin = this.props.margin;
		const canvasWidth = this.props.width - margin.left - margin.right;
		const canvasHeight = this.props.height - margin.top - margin.bottom;
		return {
			width: canvasWidth,
			height: canvasHeight
		}
	}

	render() {
		return (
			<svg width={this.props.width} height={this.props.height}>
				<g ref={node => this.node = node} />
			</svg>
		)
	}
}

export default BarChart;
```


The experience I had by using this approach is that using react as some kind of 'container' for all the d3 code is a bit cumbersome, d3.js could've taken control of everything for rendering and updating DOM, but, on the other hand, since react capsuled the whole data visualization as a component, passing properties, data or other parameters from outside make it much easier for the user of the component, which is similar to the closure capsulation method mentioned in [develop a d3.js edge](http://shop.oreilly.com/product/9781939902023.do) book. I might try to compare these react+d3 libraries and see how far they can go in terms of complex data visualization, hopefully to find a better way to integrate these two awesome libraries.

UPDATE: So I tried using this approach in a react+redux structure project, as mentioned earlier, the good thing is that I could easily wrap a chart widget in a component and use state/props to pass down the data, but, one annoying thing is that sometimes you have to copy the data within the component, for instance, if you are using a treemap, and in the chart drawing function, you will need to use things like `treemap(data)`, but this data calculation/manipulation will change the data, and it will be reflected in the redux store, and this will mess up all the store. In this case, you need to create a data copy to avoid affecting data in the store. (but not always, a simple bar chart wouldn't need this).
