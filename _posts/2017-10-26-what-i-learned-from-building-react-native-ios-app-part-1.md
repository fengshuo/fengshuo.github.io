---
title: What I learned from building a React Native iOS app - Part 1
categories:
- Frontend
- Web
tags:
- react
- react native
- iOS
- mobile app
description: Things I learned from building a react native iOS app
date: 2017-10-26 19:20
author_profile: true
classes: wide
---

![Recycle Wizard in App Store](/assets/img/posts/2017-10-26-what-i-learned-from-building-react-native-ios-app-part-1/appInStore.png)

Today the React Native iOS app I've been building is available in App Store, I am very excited and I want to write down all the lessons I‘ve learned.

# The idea
A quick intro on why I want to build this app: Recycling right is hard, and it's more difficult for me, because I come from a place doesn't really do recycle that much, when I try to follow the City of Toronto guide, I have to open the browser, find the waste wizard link and type in the keyword, but the problem is, sometime I am not even sure about which keyword to use, or there are just too many items to get rid of like when I was moving to another place. Then I thought maybe I can use image recognition to help me identify items and use the Waste Wizard data to find a match to tell me what to do with them, I know google has pretty good image recognition api, and I can use the phone camera to take photos, and I know I can use React Native to build an iOS app.

# The Hello world

At first I used the quick start [create react native app](https://github.com/react-community/create-react-native-app), it requires minimal setup and you can use the Expo app to see the hello world example in a few minutes. But then I switched to using the other setup approach because I was concerned that it's very possible that I need to use the xcode simulator to test the app on different devices and since I have to deal with xcode evetually when I submit the app, I should probably just use the non quick start initialization.

If you want to release your app to the app store(not sure about Google Play, but at least for iOS app store), personally I think it's better to use the Building Projects with Native Code approach to set up your project, but if it's for a quick demo, then use the create react native app approach is handy.

# Debugging is different from web

I have to say debugging on web is so much easier compared with debugging a RN App. On browser(say Chrome), the dev tool is easy to use and it's very mature, but for RN apps(or maybe for hybrid app, or those built with Ionic) the official debugging tool, which is just a URL, is not that helpful for me, you can console log things in the console and see network, BUT you can't see react component because the app is either in the simulator or your testing device, and since I've been using redux, I use the redux dev tool all the time on browser. So I have to find a dev tool to debug react, redux, and the regular network and console, it turned out there are many options in the open source community, I tried a few such as Reactotron, but it's not what I needed, then I found [React Native Debugger](https://github.com/jhen0409/react-native-debugger), which provides all the debugging functionalities I need. If you are new to RN dev, think about what you need and find a dev tool suits your need first.

Once you have the external debug tool set up, you need to enable the connection in your RN app(sometimes I forgot this step and then got confused why the debug tool doesn't work). If you are using a Xcode simulator you can use `cmd + d` to invoke the menu, you should be able to see options like `Enable Remote Debug`, `Enable Live Reload`, `Show Inspector` etc, you have to at least enable remote debug and live reload/hot reloading is also definitely helpful(however for some test cases you'll probably need a hard refresh, I'll talk about this later). If you are using a testing device like [this](https://facebook.github.io/react-native/docs/running-on-device.html), you'll need to shake your device to invoke the debug menu. I don't use the Inspector (on the device) that much because for me it's a bit hard to find a small component on a small screen and the information from the inspector is a bit hard to see, so if I want to inspect a component, I just search the component in the react dev part of the debugging tool, and change styles with hot reloading, but there might be other scenarios where inspectors are more helpful.

If you see some error message like `No bundle url present`, don't worry, so many people encountered this issue, and there are more than one or two solutions in this [thread](https://github.com/facebook/react-native/issues/12754), you'll find one that works for you.

# Design Mockup

The idea is pretty straightforward so I didn't think I need to spend too much time on it, but I should've done my
design prep in hindsight. I only drew some drafts with a pen, only wanted to have a search bar for keyword matching, a camera page to take photos, a search result page and a footer nav bar to navigate. But I wasn't in the mobile app development mindset, I completely forgot about the navigation of the other parts. I would say that you still need planning and designing even if it's just a simple app, especially if you are not that familiar with mobile app dev. For instance, I realized I didn't consider the UX of asking for camera access and photo library permission, I also forgot the offline mode, if I am developing for web, I don't really need to consider these little things.


# Getting Started

I wanted to use RN with redux as I am used to, also I wanted to try to use `redux-saga`, the api requests aren't that complicated, I only need: 1) send a GET request to get the City of Toronto [Waste Wizard JSON data](https://www1.toronto.ca/wps/portal/contentonly?vgnextoid=859d258b2262e410VgnVCM10000071d60f89RCRD&vgnextchannel=bcb6e03bb8d1e310VgnVCM10000071d60f89RCRD) 2) send a POST request to the Google image recognition api with the picture I take to get the labels. Redux saga is a bit overkill in this case but I want to try it out since I am using/learning it at work.

After I finish the very basic mockup, I checked the RN official doc again to see how "native" the RN component can be, some basic ones are pretty good, like `TextInput`, it has all the handy and native props, such as `returnKeyType`, `clearButtonMode`, But I also want to have some open source mature components, just like the react material UI lib, I found [Native Base](https://docs.nativebase.io/), it provides many common iOS components, like swipe card, drawer, etc.

# Rename the project (unnecessary work)

I am not good at thinking creative names for a product, so initially I started the project as 'Recycle Jon' but I didn't realize I want to use a different name until I already worked on it for a while, so I decided to change the project name to Recycle Wizard. So to change the project name, I need to also change some configuration in XCode, relinking and whatnot, but now I think I don't need to do this, because there is a setting in xcode to change the display name, it's in `info.plist - bundle display name`, this is the place to change the name to be displayed on the phone.
