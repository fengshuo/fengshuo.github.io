---
title: Immutable state to trigger component re-render in react-redux
categories:
- Frontend
- Web
tags:
- react
- redux
- immutable
- render
- component
description: Don't forget to use immutable state in react-redux to trigger component
  re-render.
date: 2017-03-14 20:20
author_profile: true
classes: wide
---

I was working on a feature that requires manipulating an array, the logic is that when user select something, push the id into the array, if user select this thing again, remove the id from the array, a pretty simple thing with the array.

After I connected the React component with the array in the store, I could see that the store is updated correctly, but the component is not re-rendering. It turns out that the state [needs to be immutable](https://github.com/reactjs/react-redux/issues/87) so that re-render can be triggered. I was doing all those `indexOf`, `splice` on the original array, so I was mutating the state in the reducer, to create a new array instead of changing the original array in the store, just use `someArray.slice(0);` because `slice` will [return a shallow copy portion of an array into a new array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) of a  , in another word, the slice operation clones the array and returns the reference to the new array, and use `0` will be [faster in some cases](http://stackoverflow.com/questions/3978492/javascript-fastest-way-to-duplicate-an-array-slice-vs-for-loop).

This is not something bizarre, but I want to take a note here, because I tend to forget that arrays are references, and should be treated differently than primitive types.
