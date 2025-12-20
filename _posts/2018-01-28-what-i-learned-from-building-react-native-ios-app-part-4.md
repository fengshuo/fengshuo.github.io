---
title: What I learned from building a React Native iOS app - Part 4
categories:
- Frontend
- Web
tags:
- xcode
- react native
- iOS
- mobile app
description: Things I learned from building a react native iOS app
date: 2018-01-28
author_profile: true
classes: wide
---

I received an email from a user (yay!) but he said that basic search doesn't work (oh no!), so I started to debug, it turns out it was caused by data source change, the url and response shape is different, I expected it should take less than an hour to release a new version.

I was wrong.

The logic change only took a few min, but I was stuck for hours just because of a build issue. I couldn't figure out why, I posted a question on [StackOverflow](https://stackoverflow.com/questions/48482984/react-native-build-error-could-not-read-optimization-profile-file-even-after-c) but I wanted to release the fix version as soon as I can, so I used the approach listed in the answer I posted...It worked, but I did spend some time to configure what I've already configured before, such as `react native link` for all the plugins that are using native component. Hopefully I won't need to do all these steps again in the future, but I'll write it down just in case:

### undefined is not an object (evaluating 'regeneratorRuntime.mark')
How to solve it? I actually [posted the solution before](https://github.com/facebook/react-native/issues/14838#issuecomment-327343053) 😅

### react-native-camera

Before using it on connected device, you need to add camera permission in `info.plist`: `Privacy - Camera Usage Description` and `Privacy - Photo Library Usage Description`

### react-native-permissions

Add all the App Store submission disclaimer if you want to submit it app store.

### app icons and launch images

Luckily all the app icons and launch image source are still in the xcode, but make sure the launch screen file field is empty:

![app icons](/assets/img/posts/2018-01-28-what-i-learned-from-building-react-native-ios-app-part-4/xcode-image-resource.png)
