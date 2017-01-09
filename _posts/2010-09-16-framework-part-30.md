---
layout: post
title: "Let's Make a Framework: Event Delegation"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- dom
- events
---

Welcome to part 30 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

[Last week](http://dailyjs.com/2010/09/09/framework-part-29/) I wrote about mixing event handlers into the DOM finder chainable API. This week I'll cover event delegation.

[Ryan Cannon](http://ryancannon.com/) commented on last week's post to ask where event delegation was. I was planning on leaving this out until later, but Ryan reminded me how important event delegation is to modern browser-based JavaScript.

### Event Delegation

*Event delegation* is where an event is attached to a parent element (or the whole document), then selectors are used to determine if a handler should run.

Consider the following markup:

{% highlight html %}
<ul id="navigation">
  <li><a href="#page_1">Page 1</a></li>
  <li><a href="#page_2">Page 2</a></li>
  <li><a href="#page_3">Page 3</a></li>
</ul>
{% endhighlight %}

An event handler could be attached to <code>\#navigation</code> and watch for clicks on the links. The advantage of this is new links can be added (perhaps through Ajax) and the event handler will still work. It also uses less events when compared to attaching events to each link.

This approach was a revelation 5 or 6 years ago, but it's now considered best practice for a wide range of applications.

### Implementations

jQuery offers two methods for dealing with this:

{% highlight javascript %}
$('#navigation a').live('click', function() {
  // Handler
});

$('#navigation').delegate('a', 'click', function() {
  // Handler
});
{% endhighlight %}

The [live](http://api.jquery.com/live/) and [delegate](http://api.jquery.com/delegate/) methods are very similar and share the same code underneath.

The way this used to work in Prototype (and other Prototype-influenced frameworks) was like this:

{% highlight javascript %}
$('navigation').observe('click', function(event) {
  var element = event.findElement('a');
  if (element) {
    // Handler
  }
});
{% endhighlight %}

As you can see, jQuery's API saves a little bit of boilerplate code. Prototype 1.7 introduced <code>on</code> which can be used like this:

{% highlight javascript %}
$('navigation').on('click', 'a', function(event, element) {
  // Handler
});
{% endhighlight %}

This is more like jQuery's <code>delegate</code> method.

### Underneath

jQuery relies on several things to handle delegation. The main method is <code>liveHandler</code>, in [event.js](http://github.com/jquery/jquery/blob/master/src/event.js):

1.  <code>jQuery.data</code> is used to track event handler objects in the current context
2.  <code>closest</code> is used to find elements based on the event's <code>target</code> and <code>currentTarget</code>, and the selectors found in the previous step
3.  Each matching element is compared against the selectors to generate a list of matching elements
4.  This element list is then used to run the user-supplied event handlers
5.  If the handler returns false or propagation has been stopped, false will be returned

The Prototype approach is simpler -- it loses some of the flexibility of the jQuery approach -- but might be a better candidate to base our framework code on. Let's target the original Prototype style (that used <code>findElement</code>) and make it generic.

{% highlight javascript %}
function findElement(event, expression) {
  var element = Event.element(event);
  if (!expression) return element;
  while (element) {
    if (Prototype.Selector.match(element, expression)) {
      return Element.extend(element);
    }
    element = element.parentNode;
  }
}
{% endhighlight %}

This code loops through each element, going up the DOM, until an element is found that matches the selector. Turing already has some internal DOM methods we can use to implement this feature.

### Next Week

In next week's tutorial I'll explain how to implement <code>findElement</code> and use it to create a simple event delegation API.
