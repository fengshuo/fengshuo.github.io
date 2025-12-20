---
title: What I learned from building a React Native iOS app - Part 3
categories:
- Frontend
- Web
tags:
- react
- react native
- iOS
- mobile app
- redux
- redux-saga
- google
description: Things I learned from building a react native iOS app
date: 2017-10-31 19:20
author_profile: true
classes: wide
---

[Part 1](https://shuofeng.dev/what-i-learned-from-building-react-native-ios-app-part-1/) is mainly about setup and all the prep. [Part 2](https://shuofeng.dev/what-i-learned-from-building-react-native-ios-app-part-2/) will be notes about problems I encountered while implementing the app.

# Using the Camera
AWS has a similar image scene detector api, but I still prefer [google's image analysis api](https://cloud.google.com/vision/), it has many cool features like label detection, face detection, and OCR, for the app, my intention is to use the google api to identify items such as this is a can, that is a bottle etc. [The usage of the api](https://cloud.google.com/vision/docs/request) is pretty simple and straightforward, a sample request is like this:

```js
{
  "requests":[
    {
      "image":{
        "content":"/9j/7QBEUGhvdG9...image contents...eYxxxzj/Coa6Bax//Z"
        // the content of the image has to be one of these three:
        // base64-encoded image string
        // Google Cloud Storage URI
        // publicly-accessible HTTP or HTTPS URL
      },
      "features":[
        {
          "type":"LABEL_DETECTION",
          "maxResults":1
        }
      ]
    }
  ]
}
```

I am gonna have to use the base64-encoded image string to request the api.

I remember I spent a lot of time to make the camera flow work, but I encountered some native app specific issues. I used [react-native-camera](https://github.com/lwansbrough/react-native-camera), not only you have to follow the doc to add the usage description key in xcode info.plist, I also had to make some change to link cameraroll in xcode(see this [doc](https://facebook.github.io/react-native/docs/linking-libraries-ios.html)), and you can't test if the camera is working with simulator, you have to [connect to a real device](https://facebook.github.io/react-native/docs/running-on-device.html) 😅, I believe I spent more time on the configuration than actually coding for the camera flow, just another typical dev day.

The plan to use the api is: 1) take the picture 2) get the uri of the pic in photo library 3) resize the image to the minimum size to get both quality and speed 4) update the label data in redux.

This seems to be okay, but later when I got to work on the navigation, I realized that since the keyword search and image search are sharing the same result page, this page assumes the data is already there when user get to the result page. So I need the user to stay on the camera page with a loading spinner, and only navigate them to the result page when it gets the google api response, the detail of this will be discussed in another post.

Go back to the plan, I remember it felt easy to use the `yield` in redux-saga, because all these file reading resizing processes are asynchronous, it just looks so much cleaner with the yield generator.

Another very native app issue is the permissions, I didn't realize the necessity of handling logic when user denies the camera or photo library access until I was about to make the ipa file.

# Find the result

The open data api returns JSON data(GET method) it doesn't support using parameter query to get result for a keyword, that means I have to do all the searching, since I also want to support auto suggestion in the keyword search box, just using lodash doesn't seem to be enough, luckily some smart people work on a JavaScript fuzzy search so I can spend more time on other things, it's not perfect but much better than using just lodash, I used [fuse.js](http://fusejs.io/) to handle the fuzzy search in the waste wizard data, [however there is an issue using Fuse with RN project](https://github.com/krisk/Fuse/issues/171), to use Fuse I had to use the workaround - `copy the the src dir from node_modules/fuse.js/src and pasted it into my ./app/ dir but renamed it to fuse` then use it like this `const Fuse = require('../libs/fuse');`.

The search for one key word is easy but it's kind of hard to decide the threshold parameter, because the google image api returns multiple labels, if the threshold is high the result will be very long, but if the threshold is too low, it won't return good result, for instance, if the threshold is 0, the label from google api is `eyecare` it doesn't return `eyeware` stuff like this.

# Navigation(Routing)

The biggest lesson I learned from Navigation is you really need to have a design, it doesn't have to be fancy, but it should at least have all the user flow so that you have a plan when you implement the routing.

I used [react-native-navigation](https://facebook.github.io/react-native/docs/navigation.html) as the official RN docs said, BUT I saw a long thread discussing things to improve for this lib, and I did have some issues with it, I would say if you are familiar with other react routing libs, maybe compare the pros and cons before choosing which routing/navigation lib to use.

I mentioned earlier that after taking a picture I need to get the result first then navigate user to the result screen from camera screen, that means I need to call the navigation in saga instead of inside the component, unfortunately there is no api to use, I had to use the workaround in this [thread](https://github.com/react-community/react-navigation/issues/1439).

The other thing I noticed is that the navigation transition sometime is slow, I am not sure if a customized transition can fix this or it's just a performance issue of the lib.
