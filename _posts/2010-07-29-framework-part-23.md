---
layout: post
title: "Let's Make a Framework: Touch Part 1"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- touch
---

Welcome to part 23 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part explores supporting touchscreen devices.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Supporting Touchscreen Devices

Libraries like [jQTouch](http://www.jqtouch.com/) help support devices like the iPhone, iPad and Android phones. A lot of people are interested in this because building web apps is arguably easier than writing native Objective-C or Java code, and in some cases a web app might be more suitable. In fact, I've noticed a lot of companies building jQTouch interfaces as a way of exploring an iPhone version of their site or app before actually building a native app.

jQTouch is a jQuery plugin that provides a whole wealth of features, and also includes graphical components to make building native-looking apps easier. In this tutorial series, I'm going to focus on supporting the more low-level features, like touchscreen events.

### A Simple Example: Orientation

To kick things off, let's look at detecting orientation changes. DailyJS's Turing framework already has support for events. WebKit supports the <code>orientationchange</code> event:

{% highlight javascript %}
turing.events.add($t('body')[0], 'orientationchange', function(e) {
  alert('put me down you oaf!');
});
{% endhighlight %}

### Debugging

Before progressing, let's look at debugging options. Android devices can use <code>logcat</code> -- take a look at [Android Debug Bridge](http://developer.android.com/guide/developing/tools/adb.html) for some help in this area.

Safari for iPhone has a debug console. To enable it, go to Settings, Safari, Developer and enable debugging:

![](/images/posts/iphone_debug_settings.jpg)

Then each page gets a debug bar:

![](/images/posts/safari_console.jpg)

### Orientation Property

There's a property called <code>orientation</code> on <code>window</code> that can be used to detect the current angle of the device. This can be interpreted to figure out the exact orientation of the device:

{% highlight javascript %}
touch.orientation = function() {
  var orientation = window.orientation,
      orientationString = '';
  switch (orientation) {
    case 0:
      orientationString += 'portrait';
    break;

    case -90:
      orientationString += 'landscape right';
    break;

    case 90:
      orientationString += 'landscape left';
    break;

    case 180:
      orientationString += 'portrait upside-down';
    break;
  }
  return [orientation, orientationString];
};
{% endhighlight %}

This code is from the <code>turing.touch</code> object. Now orientation changes can be detected and handled like this:

{% highlight javascript %}
turing.events.add($t('body')[0], 'orientationchange', function(e) {
  alert(turing.touch.orientation());
});
{% endhighlight %}

Remember that <code>$t</code> is the shortcut for <code>turing.dom.get</code> -- this is all in <code>turing.alias.js</code>.

### Conclusion

Although we're only focusing on WebKit here it's interesting that the existing DOM event library we've built can be used for most of this functionality. Other than the proprietary <code>orientation</code> property, there isn't anything here that's too messy.

Next week I'll look at swipe events.
