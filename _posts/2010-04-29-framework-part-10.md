---
layout: post
title: "Let's Make a Framework: Events Part 2"
author: Alex Young
categories: 
- events
- web
- frameworks
- tutorials
- lmaf
---

Welcome to part 10 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at events.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

This part explores an approach to event handling used by most frameworks. The implementation is a basic one that will be expanded next week.

### Event Registration

Last week I explained how event registration works when using attributes and properties like <code>onclick</code>. This is a simple way to manage cross-browser events, but isn't quite suitable for a framework.

For one thing, managing multiple events for the same type can become confusing:

{% highlight javascript %}
element.onclick = function() { alert('Hello World!'); return false; };
element.onclick = function() { alert('This was the best example I could think of'; return false; };
// Guess what happens when the element is clicked?
{% endhighlight %}

To get around this, some developers use a registry or cache to track and remove events. For a discussion and examples of this, take a look at [addEvent recording contest](http://www.quirksmode.org/blog/archives/2005/09/addevent_recodi.html) on QuirksBlog.

### W3C and Microsoft

The [Level 2 Events Specification](http://www.w3.org/TR/2000/REC-DOM-Level-2-Events-20001113/) tried to encourage browser developers to provide a better API, but in the end wasn't supported by Microsoft. From my experiences of framework-less JavaScript I can tell you that this model is pretty sound.

The quest for browser support using this API has resulted in lots of similar event handling implementations. Most conditionally use the W3C or Microsoft APIs.

### W3C Event Handling

Adding an event handler to an element looks like this:

{% highlight javascript %}
element.addEventListener('click', function() { }, false);
{% endhighlight %}

Type is W3C's terminology for event names, like 'click'. The third parameter determines if capturing should be initiated. We're not going to worry about capturing here. The event handler is passed an <code>event</code> object, which has a <code>target</code> property.

Events can be removed with <code>removeEventListener()</code>, with the same parameters. It's important that the callback matches the one the event was registered with.

Events can be fired programatically with <code>dispatchEvent()</code>.

### Microsoft

Before getting disgruntled about Microsoft breaking the universe again, their implementation almost stands up:

{% highlight javascript %}
var handler = function() { };
element.attachEvent('onclick', handler);
element.detachEvent('onclick', handler);

// Firing events
event = document.createEventObject();
return element.fireEvent('on' + type, event)
{% endhighlight %}

This is mostly similar to W3C's recommended API, except 'on' is used for event type names.

The two main issues with Microsoft's implementation are [memory leaks](http://msdn.microsoft.com/en-us/library/bb250448%28VS.85%29.aspx) and the lack of a <code>target</code> parameter. Most frameworks handle memory leaks by caching events and using <code>onunload</code> to register a handler which clears events.

As for <code>target</code>, jQuery's <code>fix</code> method maps IE's proprietary [srcElement property](http://msdn.microsoft.com/en-us/library/ms534638%28VS.85%29.aspx), and Prototype does something similar when it extends the <code>event</code> object:

{% highlight javascript %}
Object.extend(event, {
  target: event.srcElement || element,
  relatedTarget: _relatedTarget(event),
  pageX: pointer.x,
  pageY: pointer.y
});
{% endhighlight %}

### Capabilities and Callbacks

Our event handling code would be a lot simpler without Microsoft's API, but it's not a huge problem for the most part. Capability detection is simple in this case because there's no middle-ground -- browsers either use W3C's implementation or they're IE. Here's an example:

{% highlight javascript %}
if (element.addEventListener) {
  element.addEventListener(type, responder, false);
} else if (element.attachEvent) {
  element.attachEvent('on' + type, responder);
}
{% endhighlight %}

The same approach can be repeated for removing and firing events. The messy part is that the event handlers need to be wrapped in an anonymous function so the framework can correct the <code>target</code> property. The process looks like this:

1.  The event is set up by the framework: <code>turing.events.add(element, type, handler)</code>
2.  The handler is wrapped in a callback that can fix browser differences
3.  The event object is passed to the original event handler
4.  When an event is removed, the framework matches the passed in event handler with ones in a "registry", then pulls out the wrapped handler

Both jQuery and Prototype use a cache to resolve IE's memory leaks. This cache can also be used to help remove events.

### Valid Elements

Before adding an event, it's useful to check that the supplied element is valid. This might sound strange, but it makes sense in projects where events are dynamically generated (and also with weird browser behaviour).

The <code>nodeType</code> is checked to make sure it's not a text node or comment node:

{% highlight javascript %}
function isValidElement(element) {
  return element.nodeType !== 3 && element.nodeType !== 8;
}
{% endhighlight %}

I got this idea from [jQuery's source](http://github.com/jquery/jquery/blob/master/src/event.js).

### API Design

At the moment the Turing API reflects W3C's events:

-   Add an event: <code>turing.events.add(element, type, callback)</code>
-   Remove an event: <code>turing.events.remove(element, type, callback)</code>
-   Fire: <code>turing.events.fire(element, type)</code>
-   An <code>event</code> object is passed to your callback with the <code>target</code> property fixed

### Tests

Like previous parts of this project, I started out with tests. I'm still not quite happy with the tests though, but they run in IE6, 7, 8, Chrome, Firefox and Safari.

{% highlight javascript %}
var element = turing.dom.get('#events-test a')[0],
    check = 0,
    callback = function(e) { check++; return false; };

should('add onclick', function() {
  check = 0;
  turing.events.add(element, 'click', callback);
  turing.events.fire(element, 'click');
  turing.events.remove(element, 'click', callback);
  return check;
}).equals(1);
{% endhighlight %}

I use a locally-bound variable called <code>check</code> which counts how many time an event has fired using the event's handler.

The tests also ensure that attaching global handlers work (on <code>document</code>).

### Conclusion

The code is in the [GitHub repo](http://github.com/alexyoung/turing.js) for you to experiment with (and supply fixes!)

Although there's still more to do, the W3C/Microsoft approach works well. I actually experimented with implementations that manage lists of traditional event handlers rather than using the more modern API, and I spent several hours looking at Prototype and jQuery's APIs before arriving at this point.

You'll notice production frameworks contain a lot more code than Turing -- jQuery actually implements its own bubbling system, and both frameworks have to deal with inconsistencies for certain types of events. I've left those browser behaviour patches out for now in the interest of brevity.

If you'd like to expand your JavaScript knowledge on this subject, here are the references I used:

-   [Document Object Model Events](http://www.w3.org/TR/2000/REC-DOM-Level-2-Events-20001113/events.html)
-   [Node.nodeType](https://developer.mozilla.org/en/nodeType)
-   [Understanding and Solving Internet Explorer Leak Patterns](http://msdn.microsoft.com/en-us/library/bb250448%28VS.85%29.aspx)
-   [srcElement Property](http://msdn.microsoft.com/en-us/library/ms534638%28VS.85%29.aspx)
-   [createEventObject Method](http://msdn.microsoft.com/en-us/library/ms536390%28VS.85%29.aspx)
-   [QuirksMode on registration models](http://www.quirksmode.org/js/events_advanced.html)
-   [addEvent() recoding contest](http://www.quirksmode.org/blog/archives/2005/09/addevent_recodi.html)
-   [Prototype's event.js](http://github.com/sstephenson/prototype/blob/master/src/dom/event.js)
-   [jquery's event.js](http://github.com/jquery/jquery/blob/master/src/event.js)
