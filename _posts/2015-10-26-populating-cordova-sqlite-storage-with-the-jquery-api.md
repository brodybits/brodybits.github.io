---
layout: post
title: Populating Cordova SQLite storage with the JQuery API
categories: Cordova SQLite API JQuery
---

In a [JQuery JSON Cordova issue on Stack Overflow](http://stackoverflow.com/questions/33240009/jquery-json-cordova-issue) someone had trouble populating a sqlite database when using the JQuery AJAX and JSON APIs due to a subtle misunderstanding of how the Web SQLite API works. This post is written to explain how this can go wrong and how to follow the SQLite API correctly. This post assumes a very basic understanding of how the [Web SQL API](http://www.w3.org/TR/webdatabase/) or [Cordova-sqlite-storage](https://github.com/litehelpers/Cordova-sqlite-storage) API works.

# The problem

In general, once a database is open it should be possible to populate it using the following form:

{% highlight javascript %}
db.transaction(function(tx) {
  tx.executeSql('INSERT INTO MyTable VALUES(?)', my_first_row_array);
  tx.executeSql('INSERT INTO MyTable VALUES(?)', my_second_row_array);
  // ...
} function(e) {
  console.log('Transaction error: ' + e.message);
}, function() {
  console.log('INSERT transaction OK');
});
{% endhighlight %}

## Broken sample

But the following code will NOT work:

{% highlight javascript %}
  // BROKEN SAMPLE:
  var db = window.sqlitePlugin.openDatabase({name: "test.db"});
  db.executeSql("DROP TABLE IF EXISTS tt");
  db.executeSql("CREATE TABLE tt (data)");

  db.transaction(function(tx) {
    $.ajax({
      url: 'https://api.github.com/users/litehelpers/repos',
      dataType: 'json',
      success: function(res) {
        console.log('Got AJAX response: ' + JSON.stringify(res));
        $.each(res, function(i, item) {
          console.log('REPO NAME: ' + item.name);
          tx.executeSql("INSERT INTO tt values (?)", JSON.stringify(item.name));
        });
      }
    });
  }, function(e) {
    console.log('Transaction error: ' + e.message);
  }, function() {
    // Check results:
    db.executeSql('SELECT COUNT(*) FROM tt', [], function(res) {
      console.log('Check SELECT result: ' + JSON.stringify(res.rows.item(0)));
    });
  });
{% endhighlight %}
<!-- (*) -->

Note that `db.executeSql` is sometimes used to execute a sql statement without the overhead of the transaction process.

## Analysis

Since `$.ajax` is an asynchronous call, the database transaction is finished before the `$.ajax` `success` callback is triggered.

# Doing it right

## Create a Cordova project with the plugins

{% highlight bash %}
cordova create AjaxSQLite
cd AjaxSQLite
cordova plugin add cordova-plugin-console
cordova plugin add cordova-plugin-whitelist
cordova plugin add cordova-sqlite-storage
{% endhighlight %}

As a result, you *should* have a Cordova project that has logging enabled, [Cordova-sqlite-storage plugin](https://github.com/litehelpers/Cordova-sqlite-storage) installed, and multi-domain "whitelisting" enabled.

## Fix Content-Security-Policy meta tag

The credit for this section goes to the [accepted answer](http://stackoverflow.com/a/31714491) to [this Stack Overflow question](http://stackoverflow.com/questions/31506957/ajax-call-on-cordova-ios-securityerror-dom-exception-18), at least for iOS.

It took me 2-3 hours of searching, trial, and error last week to discover this one. As of this writing, the Cordova default project template has a limited `Content-Security-Policy` setting in the meta tag. Please fix the `Content-Security-Policy` meta tag in `www/index.html` according to [the accepted answer](http://stackoverflow.com/a/31714491):

<!-- ref: http://html5doctor.com/cite-and-blockquote-reloaded/ -->
<div style="padding-left: 40px; overflow-x: auto"> <!-- blockquote -->
Check your meta tag. By default, it uses:
{% highlight html %}
<meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *">
{% endhighlight %}
use the code below to enable all requests
{% highlight html %}
<!-- Enable all requests, inline styles, and eval() -->
<meta http-equiv="Content-Security-Policy" content="default-src *; style-src 'self' 'unsafe-inline'; script-src: 'self' 'unsafe-inline' 'unsafe-eval'">
{% endhighlight %}
<!-- NOTE: no formal "cite" here due to problems with kramdown -->
</div>
<!-- (*) -->

## Install JQuery

- Download a recent version of jquery.js (such as `1.11.3` as of this writing) from http://jquery.com/download/, either compressed or uncompressed
- Install it in the `www/js` subdirectory
- Add the `<script>` reference to `www/index.html` to load the installed version of `jquery.js` *before* loading `www/js/index.js`

## Add simple AJAX query for JSON data

Add the following code to the `onDeviceReady` function in `www/js/index.js`:

{% highlight javascript %}
    $.ajax({
      url: 'https://api.github.com/users/litehelpers/repos',
      dataType: 'json',
      success: function(res) {
        console.log('Got AJAX response: ' + JSON.stringify(res));
        alert('Got AJAX response OK');
      },
      error: function(e) {
        console.log('Got ERROR: ' + JSON.stringify(e));
        alert('Got ERROR: ' + JSON.stringify(e));
      }
    });
{% endhighlight %}

## Build and run

Use the Cordova CLI commands to add the desired platform, such as Android or iOS, and run your app (on the emulator or on a device). For example:

{% highlight bash %}
cordova platform add android
cordova run android
{% endhighlight %}

**IMPORTANT NOTE:** If you modify `config.xml` or any files in the `www` tree (such as `index.html` or `index.js`) please run the following command to refresh the Android, iOS, and/or Windows project(s) before running again:

{% highlight bash %}
cordova prepare
{% endhighlight %}

## Add SQLite queries

Add the following code before the `$.ajax` call:

{% highlight javascript %}
    var db = window.sqlitePlugin.openDatabase({name: "test.db"});
    db.executeSql("DROP TABLE IF EXISTS tt");
    db.executeSql("CREATE TABLE tt (data)");
{% endhighlight %}

It is recommended to remove (or comment out) the alert from the `$.ajax` `success` callback. Add the following code to the `success` callback:

{% highlight javascript %}
       db.transaction(function(tx) {
         $.each(res, function(i, item) {
           console.log('item: ' + JSON.stringify(item));
           tx.executeSql("INSERT INTO tt values (?)", JSON.stringify(item));
         });
       }, function(e) {
         console.log('Transaction error: ' + e.message);
         alert('Transaction error: ' + e.message);
       }, function() {
         db.executeSql('SELECT COUNT(*) FROM tt', [], function(res) {
           console.log('Check SELECT result: ' + JSON.stringify(res.rows.item(0)));
           alert('Transaction finished, check record count: ' + JSON.stringify(res.rows.item(0)));
         });
       });
{% endhighlight %}
<!-- (*) -->

For review, you should have the following code in the the `onDeviceReady` function in `www/js/index.js`:

{% highlight javascript %}
    var db = window.sqlitePlugin.openDatabase({name: "test.db"});
    db.executeSql("DROP TABLE IF EXISTS tt");
    db.executeSql("CREATE TABLE tt (data)");

    $.ajax({
      url: 'https://api.github.com/users/litehelpers/repos',
      dataType: 'json',
      success: function(res) {
        console.log('Got AJAX response: ' + JSON.stringify(res));
        $.each(res, function(i, item) {
          $.each(data, function(i, item) {
            console.log('item: ' + JSON.stringify(item));
            tx.executeSql("INSERT INTO tt values (?)", JSON.stringify(item));
          });
        }, function(e) {
          console.log('Transaction error: ' + e.message);
          alert('Transaction error: ' + e.message);
        }, function() {
          db.executeSql('SELECT COUNT(*) FROM tt', [], function(res) {
            console.log('Check SELECT result: ' + JSON.stringify(res.rows.item(0)));
            alert('Transaction finished, check record count: ' + JSON.stringify(res.rows.item(0)));
          });
        });
      },
      error: function(e) {
        console.log('Got ERROR: ' + JSON.stringify(e));
        alert('Got ERROR: ' + JSON.stringify(e));
      }
    });

{% endhighlight %}
<!-- (*) -->

## Build and run again

{% highlight bash %}
cordova prepare
{% endhighlight %}

Then use the `cordova run` command, for example:

{% highlight bash %}
cordova run android
{% endhighlight %}

## Sample project

A project with the program working is published at [@brodybits/cordova-sqlite-with-jquery-ajax-api-demo](https://github.com/brodybits/cordova-sqlite-with-jquery-ajax-api-demo).
