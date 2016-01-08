---
layout: post
title: Sample Cordova plugins with web workers
categories: Cordova SQLite WebWorker
---

The existing Cordova Javascript-native bridge mechanism was not designed to support web workers. Here is a description of how I rebuilt the Javascript-native bridge mechanism to support web workers in a sample project (with permissive licensing) and apply it to a version of [litehelpers / Cordova-sqlite-enterprise-free](https://github.com/litehelpers/Cordova-sqlite-enterprise-free) (available under GPL or commercial licensing schemes).

## Possible replacement for Cordova with web worker support

In December 2012 I described a [JSONBus idea](http://brodyspark.blogspot.com/2012/12/starting-jsonbus-towards-replacement.html) that could eventually replace [Cordova](http://cordova.apache.org/) for Android and iOS. I revisited this idea as a first step to rebuild the Cordova Javascript-native interworking to support web workers.

I posted the sample projects, which are public domain, at:

- [brodybits / aq-query-test-android](https://github.com/brodybits/aq-query-test-android)
- [brodybits / aqsa-query-test-ios](https://github.com/brodybits/aqsa-query-test-ios)

These samples show how a web worker may request native functionality and receive a callback. There are Javascript setup functions that need to be called from both the main thread and the web worker. Here are some important highlights:

- In both Android (Java) and iOS (Objective-C) versions, there is an `AQManager` class which can handle multiple implementations of the `AQHandler` interface.
- A small fix was made to the Android version to avoid ugly error messages in the Chrome debug console view. A similar fix is currently needed in the iOS version.
- The iOS version also contains test code for the Javascript to query the Objective-C code with a couple of string parameters and receive a direct result. This can be useful to get the bridge security code, for example (see below). It would be relatively trivial to add a similar test to the Android version using the `addJavascriptInterface` function on the `WebView` object.

## Sample Cordova plugins

I posted sample Cordova plugins with web workers supported (under BSD 3-clause license) at:

- [brodybits / cordova-aqs-android](https://github.com/brodybits/cordova-aqs-android)
- [brodybits / cordova-aqs-ios](https://github.com/brodybits/cordova-aqs-ios)

There is also a test project at: [brodybits / cordova-aqs-test](https://github.com/brodybits/cordova-aqs-test)

Note that the sample plugins do *not* provide any of the necessary Javascript interface code.

Major TODOs:

- Integrate `AQHandler` interface and `AQManager` class
- iOS version may have ugly error messages in the Safari debug console view. (The Android version has this fixed.)
- [brodybits / cordova-aqs-test](https://github.com/brodybits/cordova-aqs-test) sends the request from a web worker and gets the callback in the main thread. The Javascript code from [brodybits / aq-query-test-android](https://github.com/brodybits/aq-query-test-android) or [brodybits / aqsa-query-test-ios](https://github.com/brodybits/aqsa-query-test-ios) should be integrated to receive the callbacks in the sample worker.

## Cordova sqlite plugin (free enterprise version)

The Javascript and native code in the sample project above was used in [litehelpers / cordova-sqlite-workers-evfree](https://github.com/litehelpers/cordova-sqlite-workers-evfree) (available under GPL or commercial licensing options) to support the sqlite functionality in both the main thread and web workers. Here are some important highlights:

- In this version, `SQLitePlugin.js` is included in the application code and *not* installed automatically. This version of the plugin adds a `sqlitePluginHelper` object to the root object with an `exec` function.
- Both the Android and iOS versions are fixed to avoid ugly "not found" errors in the debug console view.

## Possible security issue(s)

- XHR URI request mechanism needs a secret code mechanism, like they have in the Cordova framework

NOTE: This is only an issue if app loads external content.
