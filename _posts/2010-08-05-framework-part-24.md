---
layout: post
title: "Let's Make a Framework: Touch Part 2"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- touch
---

Welcome to part 24 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues the discussion on touchscreen devices.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

It might seem like 24 parts is a lot for a series like this. It would seem like a daunting figure if you wanted to read through all of them. Perhaps a professionally edited epub or pdf version would be more appropriate? We'll see!

### Events

Last week I explained how to detect orientation changes. This is actually very simple once you know how to interpret <code>window.orientation</code>. Other events, like multi-touch gestures, take a bit more work. [jQTouch](http://github.com/senchalabs/jQTouch), which is one of the leading frameworks in this area, makes this easier by offering helper events like <code>swipe</code> and <code>tap</code>. The <code>swipe</code> event also makes it easy to detect the direction of the swipe.

The jQTouch source also has a joke about touch events:

{% highlight javascript %}
// Private touch functions (TODO: insert dirty joke)
function touchmove(e) {
{% endhighlight %}

The "real" events, as far as Safari is concerned, are:

-   <code>touchstart</code>
-   <code>touchmove</code>
-   <code>touchend</code>
-   <code>touchcancel</code>

The callback methods are passed event objects with these properties:

-   <code>event.touches</code>: All touches on the page
-   <code>event.targetTouches</code>: Touches for the target element
-   <code>event.changedTouches</code>: Changed touches for this event

The <code>changedTouches</code> property can be used to handle multi-touch events.

### State

The way I handle tap and swipe events is by recording the state at each event.

-   If there's just one event and the position hasn't changed, it's a tap event
-   If <code>touchmove</code> fired, work out the distance of the movement and how long it took for a swipe

I'm fairly sure that only horizontal swipes make sense, seeing as vertical movement scrolls the browser window.

From the developer's perspective, the API should look like this:

{% highlight javascript %}
turing.events.add(element, 'tap', function(e) {
  alert('tap');
});
turing.events.add(element, 'swipe', function(e) {
  alert('swipe');
});
{% endhighlight %}

We can watch for all the touch events inside the library, then fire tap or swipe on the event's target element. The library registers for events like this:

{% highlight javascript %}
turing.events.add(document, 'touchstart', touchStart);
turing.events.add(document, 'touchmove', touchMove);
turing.events.add(document, 'touchend', touchEnd);
{% endhighlight %}

The <code>touchStart</code> and similar methods are our own internal handlers. That's where tap and swipe events are detected. I've actually put these "global" handlers in a method called <code>turing.touch.register</code> because I don't yet have a good way of adding them unless they're needed.

I thought it might be nice if <code>turing.events.add</code> could allow other libraries to extend it, so the touch library could say "hey, if anyone wants events called tap or touch, run register first."

### State and Pythagoras

When <code>touchStart</code> is fired, I store the state of the event:

{% highlight javascript %}
function touchStart(e) {
  state.touches = e.touches;
  state.startTime  = (new Date).getTime();
  state.x = e.changedTouches[0].clientX;
  state.y = e.changedTouches[0].clientY;
  state.startX = state.x;
  state.startY = state.y;
  state.target = e.target;
  state.duration = 0;
}
{% endhighlight %}

Quite a lot of things are recorded here. I got the idea of working out the duration of events from jQTouch -- it makes sense to do things based on time when working with gestures.

Single taps are a simple case:

{% highlight javascript %}
function touchEnd(e) {
  var x = e.changedTouches[0].clientX,
      y = e.changedTouches[0].clientY;

  if (state.x === x && state.y === y && state.touches.length == 1) {
    turing.events.fire(e.target, 'tap');
  }
}
{% endhighlight %}

Moves are a bit more complicated. I use Pythagoras to calculate how far the finger has moved. This probably isn't really required, but I like bringing highschool maths into my tutorials if possible:

{% highlight javascript %}
function touchMove(e) {
  var moved = 0, touch = e.changedTouches[0];
  state.duration = (new Date).getTime() - state.startTime;
  state.x = state.startX - touch.pageX;
  state.y = state.startY - touch.pageY;
  moved = Math.sqrt(Math.pow(Math.abs(state.x), 2) + Math.pow(Math.abs(state.y), 2));

  if (state.duration < 1000 && moved > turing.touch.swipeThreshold) {
    turing.events.fire(e.target, 'swipe');
  }
}
{% endhighlight %}

I calculate <code>turing.touch.swipeThreshold</code> based on screen resolution. I was thinking about scaling up the minimum distance considered a swipe to the iPhone 4's high resolution, but then I found out that it treats the browser as if it was the old iPhone resolution, so this wasn't actually required.

The <code>state</code> object isn't global, it's wrapped up inside a good old closure, like the rest of the class. You can check it all out in [turing.touch.js](http://github.com/alexyoung/turing.js/blob/master/turing.touch.js)
