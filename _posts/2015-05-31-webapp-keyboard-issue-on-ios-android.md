---
title: Keyboard Issue of Hybrid Web App on Android and iOS
categories:
- Frontend
- Web
tags:
- android
- iOS
- hybrid
- mobile
description: keyboard issue on hybrid web app
date: 2015-05-31
author_profile: true
classes: wide
---

Hybrid Apps has many advantages, but it also brings some headaches when it comes to integratation with native components, such as camera, keyboard, form etc. I encountered a typical issue where the form is blocked by the activated keyboard. Apart from solutions from libraries or frameworks like [ionic](http://blog.ionic.io/ionic-keyboard/), other options including iScroll, css position, and of course tweaks in native android code. Below is a snippet works fine on iOS (you can scroll the form at least), but it just couldn't work on Android, what's more weird is that it actually could work occasionally, which made the process of debug not easy.

```raw
if(parseInt(Device.os.version)>6 || Device.os.android)
// device.js plugin
{

    var wh=$(window).height();
    // resize event cannot be triggered as you expected on Android
    $(window).on('resize',function(){
        var sh=$(window).height();
        if(wh==sh)
        {
            $(window).scrollTop(0);
        }
    });
};
```

Fortunately, this can be solved by updating Android settings:

1. In AndroidManifest.xml file, add `android:windowSoftInputMode="adjustResize"` right after `android:screenOrientation="portrait"` for each view;
2. Add meta tag `"height=device-height"` in your html file, in my case, it is `<meta name="viewport" content="width=device-width, maximum-scale=1.0, user-scalable=0, initial-scale=1.0, height=device-height"/>`.

*Update:*

I just ran into a similar issue on iOS, specifically iOS 8.3. The odd thing is that `next.focus()` works on older version such as iOS 8.1. As always, I tried to solve the 'auto focus' effect issue within the domain of web area. However, 'focusin' event is more bizzare on iOS as described [here](http://stackoverflow.com/questions/6287478/mobile-safari-autofocus-text-field/7332160#7332160). As far as I can tell, at this point, the best solution is to update the iOS configure - [keyboardDisplayRequiresUserAction](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIWebView_Class/index.html#//apple_ref/occ/instp/UIWebView/keyboardDisplayRequiresUserAction). Per the API document: 'When set to NO, a focus event on an element causes the input view to be displayed and associated with that element automatically'.

## Links

[1](http://stackoverflow.com/questions/24328156/how-to-auto-scroll-to-input-field-in-android-when-the-soft-keyboard-overlaps-the)
[2](http://stackoverflow.com/questions/7026854/textbox-hidden-below-keyboard-in-android-webview)
[3](http://stackoverflow.com/questions/2559273/android-adjust-screen-when-keyboard-pops-up)
