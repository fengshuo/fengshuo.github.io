---
title: Uncontrolled React Input Component
categories:
- Frontend
- Web
tags:
- react
- component
- uncontrolled
- input
description: Things to note when making an uncontrolled react input component.
date: 2017-03-31 19:20
author_profile: true
classes: wide
---

The other day I was working on a react component which is basically a input box with a validation rule, nothing complicated here. So the validation is to allow user type only numbers in the box, I had something like this:

```js
// pseudo code for the component
constructor(props) {
	super(props);
	this.state = {
		value: someRelatedStuff || '', //someRelatedStuff can be null
	};
	this.handleInputValue = this.handleInputValue.bind(this);
}

handleInput(e) {
	if (/^[0-9]+$/.test(e.target.value) || e.target.value === '') {
		this.setState({
			value: e.target.value || null,
		});
	}
}

render() {
	return (
		<div>
				<Input
					ref="someStuff"
					type="text"
					placeholder={'type something here'}
					value={this.state.value}
					onChange={this.handleInputValue}
				/>
			</div>
			<div>
				<Button />
			</div>
		</div>
	);
}
```

It works as I expected, user can only type numbers into the box. However, while I was working on other part of the code, I kind of want to make the default value of the input box is `null`, so if user doesn't type anything in the input box and he/she submit the value, I should be able to get a value of `null` in other parts of the code, so I changed the initial state to:

```js
constructor(props) {
	super(props);
	this.state = {
		value: someRelatedStuff || null,
    // my initial thought is to make sure the value will always be able to fall back to null
	};
	this.handleInputValue = this.handleInputValue.bind(this);
}
```

And I didn't change other parts of the code, the input box didn't work as before: If I type letters, the input box will take all the letters I typed as if the `value={this.state.value}` in the `<Input />` is not working anymore, and the state of this component is not being updated whatsoever. After some googling around, it turns out that [React interpret](https://github.com/facebook/react/issues/4085) `<input value={null}>` as **an uncontrolled component**, what it means is that the data is handled by the DOM itself just like the old vanilla JS days. That's why the state is not updating and you can type letters into the box. If I clear all the letters I just typed, based on the logic in `handleInput(e)`, the value in the state will be set as an empty string, then it became a controlled form component, and the validation is working again. I wish this thing is mentioned in the [React uncontrolled component documentation](https://facebook.github.io/react/docs/uncontrolled-components.html) instead of explaining it in [Github issues](https://github.com/facebook/react/issues/6996), so that it is [less confusing](https://gist.github.com/markerikson/d71cfc81687f11609d2559e8daee10cc).
