---
title: Basic Transition in D3.js
categories:
- Frontend
- Web
tags:
- d3
- dataVisualization
description: transition in d3.js
date: 2015-05-07
author_profile: true
classes: wide
---

For starters, this [article](http://bost.ocks.org/mike/transition/) by Mike explained transitions very well, if you haven't read it, do it now because it will help you understand what's under the hood of transition.

The way I interpret the key points is:

# Transition Is Another Form of Animation

If you ever write animation in CSS or use jQuery to animate style, you should be familiar with this animation concept.

Take CSS animation for example:

```raw
.element {
  animation: pulse 5s infinite;
}

@keyframes pulse {
  0% {
    background-color: #001F3F;
  }
  50% {
    background-color: #001F5D;
  }
  100% {
    background-color: #FF4136;
  }
}
```

You specify the name, duration, timing, delay properties of the animation. The essence of transition in d3 is key frame animation but with only two key frames - start and end. That means all it requires is a start status and a end status, normally you don't need to specify a start view since d3 will read attribute value by things like `getComputedStyle` or `getAttribute`, however, if the automatically computed value is not what you want, you can specify a start point like this:

```js
d3.select("body")
    .style("color", "red")
  .transition()
    .style("color", "green")
```

But if there is a delay for the transition, the starting value should be set only when the transition starts, which can be implemented by listening for the start event:

```js
d3.select("body").transition()
    .delay(750)
    .each("start", function() {         d3.select(this).style("color", "green");
    })
    .style("color", "red");
```

As you can see, we bind a start listener for each element in this selection(in this case, it is just the body element).

# What happens when transitioning

There is a start event in previous code snippet, you may wonder if there is a correspondent event like 'end', that is correct, so what is the life of transition?

When you write `selection.transition`, you are a)scheduling or creating a transition. If there is a delay specified, then transition only starts after the delay, at that time, b)the start event is dispatched, the transition initializes and begin the animation (That's where the each start callback function comes from). c)While running the transition, an interpolate controls the progress of the animation, which will be addressed later. d)When transition ends, the end event is dispatched.

# The Interpolation

One important thing I didn't mentioned above is how d3 'tween' between start and end. As a matter of fact, you can always override the default tween by using `styleTween`, `attrTween` and `tween`.

What's under the hood of the tweens? The answer is the interpolation, namely, the `d3.interpolate` method. As far as I'm concerned, the awesomeness of `d3.interpolate(a,b)` is that the type of interpolator is based on the type of the end value b, for example, if b is color, then `interpolateRgb` is used, if b is an array, `interpolateArray` is used, and if b is an object, `interpolateObject` is used etc.

Let's take a look at a donut chart update example:

```js
function change(){
	var value = this.value;
	pieLayout.value(function(d){
		return d[value];
	})
	slice = slice.data(pieLayout(data));
  // okay, so we have updated data now

	slice.transition()
					.duration(1000)
					.attrTween("d", arcTween);
};
```

`attrTween` takes two parameters, a name of the attribute, and a tween function, which is arcTween in the example. According to the d3 api, the tween function is invoked when the transition starts on each element with following parameters:

- the current datum: `d`;
- the current index: `i`;
- the current attribute value: `a`;
- `this` as the current DOM element;

And most importantly, the return value of tween must be an interpolator.

```js
function arcTween(d,i,a){
				var i = d3.interpolate(this._current, d);
				this._current = i(1); // update current attribute for next interpolate
				return function(t){
					return arcGenerator(i(t)); //redraw
				}
			}
```

`t` is a value ranging from 0 to 1, basically the returned interpolator maps a `t` value to a color or number value. In this case, the t value gets passed to the interpolate - i, which is set with the start status and the end status, the interpolate will automatically choose the appropriate interpolate for `d`, and the result then gets passed to arcGenerator to actually draw the donut.


# Transition is per-element

One of the fundamental facts of transition is that each element transitions independently. So when you create a transition from a selection, the transition is then set on every element, and the start or end event doesn't intervene with other elements' animation.
