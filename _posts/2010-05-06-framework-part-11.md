---
layout: post
title: "Let's Make a Framework: Events Part 3"
author: Alex Young
categories: 
- events
- web
- frameworks
- tutorials
- lmaf
---

Welcome to part 11 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at events.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Stopping Events

Once an event has been triggered it can propagate to other elements -- this is known as *event bubbling*. To understand this, try to think about what happens if you have a container element and attach an event to an element inside it. When the event is triggered, which elements should receive the event?

We often don't want to propagate events at all. In addition, some elements have *default actions* -- a good example is how a link tag's default action makes the browser follow the link.

[Prototype's Event.stop() method](http://api.prototypejs.org/dom/event/stop/) simplifies event management by cancelling event propagation and default actions. We generally want to do both at the same time.

jQuery models the [W3C's Document Object Model Events spec](http://www.w3.org/TR/2000/REC-DOM-Level-2-Events-20001113/events.html#Events-flow-cancelation), providing lots of methods on the event object itself:

-   <code>event.preventDefault()</code>: Stop the default action of the event from being triggered
-   <code>event.stopPropagation()</code>: Prevents the event from bubbling up the DOM tree, preventing any parent handlers from being notified of the event
-   <code>event.stopImmediatePropagation()</code>: Keeps the rest of the handlers from being executed and prevents the event from bubbling up the DOM tree

### Our Stop API

I've modelled Turing's API on jQuery, with the addition of <code>stop()</code>. The reason I like jQuery's approach is it creates a cross-browser W3C API, which may future-proof the library.

Event objects are extended with:

-   <code>event.stop()</code> - Prevents the default handler and bubbling
-   <code>event.preventDefault()</code> - Prevents default handler
-   <code>event.stopPropagation()</code> - Stops the event propagating

Usage is best illustrated with a test from [test/events\_test.js](http://github.com/alexyoung/turing.js/blob/master/test/events_test.js)

{% highlight javascript %}
should('stop', function() {
  var callback = function(event) { event.stop(); };
  turing.events.add(turing.dom.get('#link2')[0], 'click', callback);
  // ...
{% endhighlight %}

### The Implementation

I've created a private function to extend and fix event objects. This essentially patches IE and adds <code>stop()</code>:

{% highlight javascript %}
function stop(event) {
  event.preventDefault(event);
  event.stopPropagation(event);
}

function fix(event, element) {
  if (!event) var event = window.event;

  event.stop = function() { stop(event); };

  if (typeof event.target === 'undefined')
    event.target = event.srcElement || element;

  if (!event.preventDefault)
    event.preventDefault = function() { event.returnValue = false; };

  if (!event.stopPropagation)
    event.stopPropagation = function() { event.cancelBubble = true; };

  return event;
}
{% endhighlight %}

### Other Browser Fixes

Most frameworks patch other browser inconsistencies as well. Keyboard and mouse handling in particular are problematic.

jQuery corrects the following:

-   Safari's handling of text nodes
-   Missing values for <code>event.pageX/Y</code>
-   Key events get <code>event.which</code> and <code>event.metaKey</code> is corrected
-   <code>event.which</code> is also added for mouse button index

Prototype also has similar corrections:

{% highlight javascript %}
var _isButton;
if (Prototype.Browser.IE) {
  // IE doesn't map left/right/middle the same way.
  var buttonMap = { 0: 1, 1: 4, 2: 2 };
  _isButton = function(event, code) {
    return event.button === buttonMap[code];
  };
} else if (Prototype.Browser.WebKit) {
  // In Safari we have to account for when the user holds down
  // the "meta" key.
  _isButton = function(event, code) {
    switch (code) {
      case 0: return event.which == 1 && !event.metaKey;
      case 1: return event.which == 1 && event.metaKey;
      default: return false;
    }
  };
} else {
  _isButton = function(event, code) {
    return event.which ? (event.which === code + 1) : (event.button === code);
  };
}
{% endhighlight %}

You can also find similar patching in MooTools:

{% highlight javascript %}
if (type.test(/key/)){
  var code = event.which || event.keyCode;
  var key = Event.Keys.keyOf(code);
  if (type == 'keydown'){
    var fKey = code - 111;
    if (fKey > 0 && fKey < 13) key = 'f' + fKey;
  }
  key = key || String.fromCharCode(code).toLowerCase();
} else if (type.match(/(click|mouse|menu)/i)){
  doc = (!doc.compatMode || doc.compatMode == 'CSS1Compat') ? doc.html : doc.body;
  var page = {
    x: event.pageX || event.clientX + doc.scrollLeft,
    y: event.pageY || event.clientY + doc.scrollTop
  };
  var client = {
    x: (event.pageX) ? event.pageX - win.pageXOffset : event.clientX,
    y: (event.pageY) ? event.pageY - win.pageYOffset : event.clientY
  };
  if (type.match(/DOMMouseScroll|mousewheel/)){
    var wheel = (event.wheelDelta) ? event.wheelDelta / 120 : -(event.detail || 0) / 3;
  }
{% endhighlight %}

### Conclusions

Over the last 3 weeks I've introduced event handling, explained how to use the W3C and Microsoft APIs, and built an event handling framework. The life cycle of event handling has been explained an implemented, from creating events to stopping and removing them. I've also demonstrated how differences between browsers are dealt with.

You can find the current version of Turing on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).
