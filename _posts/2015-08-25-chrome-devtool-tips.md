---
title: Learn Chrome DevTool by questions
categories:
- Frontend
- Web
tags:
- chrome
- DevTool
- frontend
description: tips and tricks of chrome devtool
date: 2015-08-25 22:48
author_profile: true
classes: wide
---

# Element Panel

1) Is there any faster way to inspect an element?

Normally people would right click element and select *Inspect Element*, or use the *Inspect Element Button* to go into Inspect Element Mode. But for developers, clicking things here and there with mouses is not efficient, so you can use shortcut `cmd+shift+c` to directly go into inspect element mode. Additionally, if it is not easy to find one specific element on a complex page, you can use `inspect()` in Console.

2) Is it possible to update element's box model parameters?

At the bottom of Styles panel, there is a visualization of element's box model, and the parameters(especially the content box size) is editable.

3) How to update CSS property value faster?

The chances are high that you need to manually update CSS property values in dev tool to see how the style looks like, instead of randomly guess some number, you can use several shortcut to update the value faster:

- `Up/Down`: increment or decrement by 1(or .1 if current value is between -1 and 1)
- `Alt + Up/Down`: increment or decrement by 0.1
- `Shift + Up/Down`: increment or decrement by 10
- `Shift + PageUp/PageDown`: (On Mac, PageUp/PageDown = Fn + Up/Down), increment or decrement by 100

4) If I am editing DOM attributes frequently, how can I switch attribute values faster?

Once the edit mode is active(by double-click on the attribute name or value, or press enter), you can cycle through attribute values by pressing `Tab`.

5) DOM breakpoints use cases?

DOM breakpoints can be very helpful while debugging your JavaScript, it includes subtree change, attribute change, node removal, which will go into breakpoint if any of those change happens. All DOM breakpoints are logged under the DOM breakpoints panel.


6) Useful keyboard shortcuts in Element Panel?

- Expand/Collapse Node: `Right Arrow / Left Arrow`;
- Edit DOM node as HTML:  `Fn + F2`;
- Expand/Collapse node and all its children: `opt + click on arrow icon`;
- Hide Element: press `H`;
- Redo Change: `cmd + shift + z`;

7) Any other tips for Style Panel?

- Color Picker: when selecting a color in style editor and the color picker is opened, if you hover over your page, your mouse pointer will transform into a magnifying glass for selecting colors.
- Change color definition: `shift + click on color picker box`;
- Go to line of property in source: `cmd + click on property value`;


# Source Panel

1) What kinds of breakpoints are available in dev tool?

In addition to DOM breakpoints mentioned above, in chrome dev tool, you can also add JavaScript breakpoint, XHR breakpoints, event listener breakpoint.

2) How to make the most out of breakpoints?

Breakpoints are essential when it comes to debugging. By stepping through the code, you can observe changes and understand what is going on, besides you can modify values and even the scripts while debugging.

- Resume: Resumes execution up to the next breakpoint;
- Long Resume: Resumes execution with breakpoints disabled for 500ms, this can be handy if the breakpoint is inside a loop;
- Step Over: Executes whatever happens on the next line and jumps to the next line;
- Step Into: If the next line contains a function call, it will jump to and pause that function at its first line;
- Step Out: Executes the remainder of the current function and then pauses at the next statement after the function call;

It is also feasible to change value in console, or change scripts in the source file, which is really helpful to experiment different debug solutions.

3) Work Spaces

Work spaces is similar to a chrome plugin called 'live style', which should be used with Sublime, basically anything you changed in dev tool editor is instantly saved in local files. Details can be found in this [talk](http://www.html5rocks.com/en/tutorials/developertools/revolutions2013/) and [official docs](https://developers.google.com/web/tools/setup/workspace/setup-workflow).

4) Run Snippets

Running custom snippets in dev tool is available in source panel >  snippets, you can add some frequently used scripts here, there is a list of [devtool-snippets](https://github.com/bgrins/devtools-snippets) for reference.

5) Useful Keyboard Shortcuts

Since you are going to step through code, it involves lots of clicking, using shortcuts is more convenient. The complete shortcuts list is [here](https://developers.google.com/web/tools/iterate/inspect-styles/shortcuts?hl=en), but I am just gonna put a few common ones (personal preference) down here:

- pause/resume script execution: `cmd+\`;
- step over: `cmd+'`;
- step into: `cmd+;`;
- step out: `cmd+shift+;`;
- toggle breakpoint condition: `cmd+b`;
- go to line: `ctrl+g`;
- search by filename: `cmd+o`;
- go to member: `cmd+shift+o`;
- close active tab: `opt+w`;
- search within source code: `cmd + opt + f`;
- multiple carets: `cmd+click`;
- select next occurrence of the current word: `cmd+d`;


# Console Panel

1) There seems to be multiple ways to activate console panel?

Console is frequently used in dev tool, I suppose that's why you can invoke the console panel in many ways (besides: chrome canary just changed the position of console panel this month to second, yay).

- the fastest: `cmd+option+j`;
- activate console drawer: `cmd+option+i` to invoke dev tool, and `esc` to open console drawer;
- `cmd+option+i` to invoke dev tool, `cmd+[` or `cmd+]` to switch panels;

2) Can I keep logs between pages and page refreshes in console?

It's pretty simple, just check the `preserve log` in console panel, then console will persist the log history.

If there are too many log info in the console, use `cmd+k` to clear the logs.

3) Using `console.log()` to diagnose is tedious, can I group or highlight log info in console?

- you can group logs by using `console.group("group msg start");` and `console.groupEnd();`
- when using group heavily, you can print automatically collapse groups by calling `console.groupCollapsed()`;
- in addition to using `console.error()` and `console.warn()` to distinguish information, it is also possible to style console output with css: `console.log("special log", "color: blue; font-size: 30px");`;

Lastly, you can add `console.assert()` to conditionally display an error string:
`console.assert(list.childNodes.length < 500, "Node count is > 500");`.

4) I tried to print a data object, but I have to expand them to see the actual data, is there a preview of the data object in console?

When diagnose data object, `console.table()` is a life saver, not only it will display object data in a table, but also it provides a feature of logging specific properties by the second parameter, such as `console.table(family, ["firstName", "lastName", "age"])`.

However, as far as I can tell, if the value of one key is an array, the array won't be expanded whatsoever.

5) I understand that I can write some JavaScript code to calculate time lapse of code execution, is there any quicker way to achieve this?

```raw
console.time("Array initialize");
var array= new Array(1000000);
for (var i = array.length - 1; i >= 0; i--) {
    array[i] = new Object();
};
console.timeEnd("Array initialize");
```

Beyond that, if a timeline recording is in progress during a `time()` operation, the result will also be available in timeline graph.

6) I would like to distinguish each log the code prints, can I add styles on logs?

You can use the format specifier to colorize output strings:

`console.log('%cheight'+$(".header").height(),'color:orange')`


The complete console api list is [here](https://developers.google.com/web/tools/javascript/console/console-reference?hl=en).


There are other sections in dev tool, such as Timeline, Network, Profile, these will be addressed in another blog(hopefully).


# Links

- [20+ Questions to learn Chrome Developer Tools — Moderate Level](https://medium.com/@cihadturhan/20-questions-to-learn-chrome-developer-tools-moderate-level-faff2abb1316)
- [15 Must-Know Chrome DevTools Tips and Tricks](http://tutorialzine.com/2015/03/15-must-know-chrome-devtools-tips-tricks/)
- [Keyboard Shortcuts](https://developers.google.com/web/tools/iterate/inspect-styles/shortcuts?hl=en)
