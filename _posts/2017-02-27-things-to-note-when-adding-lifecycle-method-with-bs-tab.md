---
title: Things to note when adding lifecycle methods with react-bootstrap tab
categories:
- Frontend
- Web
tags:
- react.js
- lifecycle
- component
- react-bootstrap
description: Issues I encountered when using react-bootstrap tab
date: 2017-02-27
author_profile: true
classes: wide
---

# Context and Issues

At work we use `react-bootstrap(0.29.3)` to put different things in [tabs](https://react-bootstrap.github.io/components.html#tabs), there was no issue using this handy tool until I started to add another tab with Highcharts components(`react-highcharts`, 2.1.0, a bit out of date, but irrelevant here). Everything looks good except when you switch from tab to tab, the line chart will execute plotting animation no matter if you are loading new data or just switch from different tab(no data updating in this case), to keep consistent with legacy products, the animation can't be turned off, so I looked at Highcharts API doc, it says the animation _only applies to the initial animation of the series itself._, which means when the data is already loaded and you switch back from other tabs, the component gets updated too. But what I want is to only redraw(the plotting animation) when there is an actual update.

The other issue is that, in the Highcharts tab, there is a dropdown menu, and it will trigger an action dispatch in redux, this action will change props for a sub-component of this tab, but no props will change for this particular tab panel. Whenever I change the dropdown active option, the page scrolls up a bit but just for the first time.

# How to fix them and trying to figure out why

For the first part of the problem, the solution is to update the component only when there is an actual update, since we are using Redux, we just need to keep an eye on the `props` change, if there is no props change, don't update the component(so that the plotting animation won't be triggered), the lifecycle function I used for this is:

```js
shouldComponentUpdate(nextProps) {
		return !_.isEqual(nextProps, this.props);
}
```

This will do the trick, but I had a hard time to understand what was going on with the scrolling.

I checked every updating related lifecycle in this Chart Tab Panel component:

```js
componentWillMount() (not really related to updateing)
componentDidMount()
componentWillReceiveProps()
shouldComponentUpdate()
componentWillUpdate()
componentDidUpdate()
```

There is no code related to scrolling up in this panel, I couldn't understand why the scrolling is happening. After I got a second pair of eyes, I then realized that there is a function in other tabs like this:

```js
componentDidUpdate(prevProps) {
	this.scrollTop();
}

scrollTop() {
	const node = ReactDOM.findDOMNode(this);
	// scroll the page to the top
}
```

and the `componentDidUpdate` function in OTHER tabs get called even though I am only changing a sub-component in the chart panel tab. I then added all the updating related lifecycle function to one of the other tabs, `componentWillReceiveProps` and `componentDidUpdate` are not being called in this scenario, but `componentDidUpdate` gets called. I understand that when user switches tabs, the whole tab tree is being [re-rendered](https://github.com/react-bootstrap/react-bootstrap/issues/2062), but what I did is to update sub-component after the tab switching, I still couldn't wrap my head around why the `componentDidUpdate` gets called even though there is no change in props or state.

After figuring out why the scrolling up is happening, what we did here is similar like the solution to question one, we used `shouldComponentUpdate` function to check if the scrolling up needs to happen.

But, as the [official react doc](https://facebook.github.io/react/docs/react-component.html#shouldcomponentupdate) says:

> Use shouldComponentUpdate() to let React know if a component's output is not affected by the current change in state or props. The default behaviour is to re-render on every state change, and in the vast majority of cases you should rely on the default behaviour.

Using `shouldComponentUpdate` might not be the ideal solution, but this works because I don't know what is triggering the change in the parent's sibling components, it could be a library issue since we are not using the latest version, they might have fixed it.
