---
layout: post
title: "Let's Make a Framework: Events"
author: Alex Young
categories: 
- events
- web
- frameworks
- tutorials
- lmaf
---

Welcome to part 9 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part introduces events.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

This part will look at how events work, event handler implementations in various frameworks, and event handler API designs. I'll select an API design at the end of the article and implement it for next week's lesson.

### Basics

Events and JavaScript are closely related -- can you imagine writing scripts for web pages that don't respond to user interaction? That means that as soon as JavaScript appeared, events did. Early event handlers were written inline, like this:

{% highlight javascript %}
<a href="/" onclick="alert('Hello World!')">
{% endhighlight %}

You've probably seen this type of inline event handler before. This came from Netscape originally, and due to the popularity of early versions of Netscape, Microsoft implemented a compatible system.

This can be written in JavaScript like this:

{% highlight javascript %}
// assume 'element' is the previous link minus the onclick
element.onclick = function() { alert('Hello World!'); };
{% endhighlight %}

### Accessing the Event

An event handler can access an event like this:

{% highlight javascript %}
function handler(event) {
	if (!event) var event = window.event;
}
{% endhighlight %}

<code>window.event</code> is a Microsoft property for the last event. I always felt like this was dangerous and could lead to to a clobbered value, but as JavaScript is single-threaded it's safe enough that most frameworks depend on it.

jQuery does it like this:

{% highlight javascript %}
handle: function( event ) {
  var all, handlers, namespaces, namespace_sort = [], namespace_re, events, args = jQuery.makeArray( arguments );
  event = args[0] = jQuery.event.fix( event || window.event );
{% endhighlight %}

### Stopping Events

I used to get default actions and bubbling confused, because I thought stopping an event just meant everything stopped. These two things are different.

#### Default Action

Returning <code>false</code> from an event handler prevents the default action:

{% highlight javascript %}
element.onclick = function() { alert('Hello World!'); return false; };
{% endhighlight %}

Now the link won't be followed.

#### Capturing and Bubbling

Attaching an <code>onClick</code> to two elements, where one is the ancestor of the other, makes it difficult to tell which element should take precedence. Different browsers do it different ways. However, fortunately it's very unusual to actually care about this -- in most cases we just need to stop the event.

{% highlight javascript %}
function handler(event) {
	if (!event) var event = window.event;
	event.cancelBubble = true;
	if (event.stopPropagation) event.stopPropagation();
}
{% endhighlight %}

As far as I know, only IE uses <code>cancelBubble</code>, but it's safe to set it in browsers that don't use it. <code>stopPropagation</code> is used by most browsers.

jQuery's implementation in [event.js](http://github.com/jquery/jquery/blob/master/src/event.js) is similar to the above, and most frameworks are broadly similar.

### Multiple Handlers

If events were this simple we'd barely need frameworks to help us. One of the things that makes things less simple is attaching multiple events:

{% highlight javascript %}
element.onclick = function() { alert('Hello World!'); return false; };
element.onclick = function() { alert('This was the best example I could think of'; return false; };
{% endhighlight %}

This example overwrites the <code>onClick</code> handler, rather than appending another one.

The solution isn't as simple as wrapping functions within functions, because people might want to remove event handlers in the future.

### Framework APIs

The job of an event framework is to make all of these things easy and cross-browser. Most do the following:

-   Normalise event names, so <code>onClick</code> becomes <code>click</code>
-   Easy event registration and removal
-   Simple cross-browser access to the event object in handlers
-   Mask the complexities of event bubbling
-   Provide cross-browser access to keyboard and mouse interaction
-   Patch browser incompetency like IE memory leaks

#### jQuery

Interestingly, [jQuery's event handling](http://api.jquery.com/category/events/event-object/) approach is to make the event object behave like the W3C standards. The reason for this is that Microsoft failed to implement a compatible API.

jQuery wraps methods into its internal DOM list objects, so the API feels very easy to use:

{% highlight javascript %}
$('a').click(function(event) {
  alert('A link has been clicked');
});
{% endhighlight %}

This adds the <code>click</code> handler to every link.

-   The Event's element is in <code>event.target</code>
-   The current event within the bubbling phase is in <code>event.currentTarget</code> -- also found in the <code>this</code> object in the function
-   The default action can be prevented with <code>event.preventDefault()</code>
-   Bubbling can be stopped with <code>event.stopPropagation();</code>
-   Events can be removed with <code>$('a').unbind('click');</code>
-   Events can be fired with <code>$('a').trigger('click');</code>

#### Prototype

[Prototype's event handling](http://api.prototypejs.org/dom/event/) is closely linked to its core DOM <code>Element</code> class. Events are registered like this:

{% highlight javascript %}
$('id').observe('click', function(event) {
  var element = Event.element(event);
});
{% endhighlight %}

-   The Event's element can be accessed with <code>Event.element(event)</code> or <code>event.element();</code>
-   Events are stopped with <code>Event.stop()</code>
-   Events can be removed with <code>Event.stopObserving(element, eventName, handler);</code>
-   Events can be fired with <code>Event.fire(element)</code>

#### Glow

Glow's event handling is found in [glow.events](http://www.bbc.co.uk/glow/docs/1.7/api/glow.events.shtml). Events are registered with <code>addListener</code>:

{% highlight javascript %}
glow.events.addListener('a', 'click', function(event) {
  alert('Hello World!');
});
{% endhighlight %}

-   The Event's element is in <code>event.source</code>
-   The default action can be prevented by returning <code>false</code>
-   Events can be removed with <code>glow.events.removeListener(handler)</code> where <code>handler</code> is the returned value from <code>glow.events.addListener</code>
-   Events can be fired with <code>glow.events.fire()</code>

Glow's jQuery influence is evident here.

#### Dojo

Rather than ironing out the problems in browser W3C event handling supporting, dojo uses a class-method-based system like Prototype, but with a different approach. Dojo's API is based around *connections* between functions. Registering a handler looks like this:

{% highlight javascript %}
dojo.connect(dojo.byId('a#hello'), 'onclick', function(event) {
  alert('Hello World!');
});
{% endhighlight %}

Notice that the event names aren't mapped. Like the other frameworks, the <code>event</code> object is normalised.

-   The Event's element is in <code>event.target</code>
-   The current event within the bubbling phase is in <code>event.currentTarget</code> -- this is also <code>this</code> in the function
-   The default action can be prevented with <code>event.preventDefault()</code>
-   Bubbling can be stopped with <code>event.stopPropagation();</code>
-   Events can be removed with <code>dojo.disconnect()</code>

### Conclusions

Out of all these frameworks, jQuery's event handling is the most fascinating. It makes a familiar W3C event system available to all browsers, carefully namespaced, and completely takes over event bubbling to achieve this. This is partially because Microsoft's API has major problems with bubbling.

Building something in between jQuery and Glow is suitable for Turing -- I don't want to worry about bubbling too much, but we will need a system that will cope with multiple handlers on the same event and the removal and firing of handlers.

Next week I'll start building the event handling code, building with the following goals in mind:

-   Normalise event names, so <code>onClick</code> becomes <code>click</code>
-   Easy event registration and removal
-   Simple cross-browser access to the event object in handlers
-   Mask the complexities of event bubbling
-   Provide cross-browser access to keyboard and mouse interaction
-   Patch browser incompetency like IE memory leaks

### Links

If you want to read more about events, browser wars, and Microsoft's frustrating inability to make life easy, [Introduction to Events](http://www.quirksmode.org/js/introevents.html) on QuirksMode covers the history and basics, and branches out into an entire series.
