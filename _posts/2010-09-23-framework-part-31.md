---
layout: post
title: "Let's Make a Framework: Event Delegation 2"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- dom
- events
---

Welcome to part 31 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

[Last week](http://dailyjs.com/2010/09/16/framework-part-30/) I wrote about event delegation in popular frameworks. This week I'll demonstrate a simple implementation for Turing.

### dom.findElement

This method will make dealing with event delegation much easier. Recall last week's tutorial:

{% highlight javascript %}
$('navigation').observe('click', function(event) {
  var element = event.findElement('a');
  if (element) {
    // Handler
  }
});
{% endhighlight %}

This [Prototype](http://prototypejs.org/) code simply uses the event's target element to see if it matches a selector. Turing's delegation API could wrap this up a method with a signature like this:

{% highlight javascript %}
turing.events.delegate(document, selector, 'click', function(e) {
  // Handler
});
{% endhighlight %}

The body of <code>delegate</code> will look a bit like the Prototype example above.

There's a few obstacles to building <code>findElement</code> with Turing's event library as it stands though. A few months ago we built a class called <code>Searcher</code> that can recurse through the DOM to match tokenized CSS selectors.

The <code>Searcher</code> class could be reused to implement <code>findElement</code>, the <code>matchesAllRules</code> method in particular is of interest.

### Tests

I'd like the following tests to pass ([events.js](http://github.com/alexyoung/turing.js/blob/master/test/events_test.js)):

{% highlight javascript %}
given('a delegate handler', function() {
  var clicks = 0;
  turing.events.delegate(document, '#events-test a', 'click', function(e) {
    clicks++;
  });

  should('run the handler when the right selector is matched', function() {
    turing.events.fire(turing.dom.get('#events-test a')[0], 'click');
    return clicks;
  }).equals(1);

  should('only run when expected', function() {
    turing.events.fire(turing.dom.get('p')[0], 'click');
    return clicks;
  }).equals(1);
});

{% endhighlight %}

To get there, we need <code>findElement</code> and corresponding tests ([dom.js](http://github.com/alexyoung/turing.js/blob/master/test/dom_test.js)):

{% highlight javascript %}
given('a nested element', function() {
  var element = turing.dom.get('#dom-test a.link')[0];
  should('find elements with the right selector', function() {
    return turing.dom.findElement(element, '#dom-test a.link', document);
  }).equals(element);

  should('not find elements with the wrong selector',function() {
    return turing.dom.findElement(turing.dom.get('#dom-test .example1 p')[0], 'a.link', document);
  }).equals(undefined);
});
{% endhighlight %}

This test depends on the markup in [dom\_test.html](http://github.com/alexyoung/turing.js/blob/master/test/dom_test.html).

### Adapting the Searcher Class

After I looked at <code>matchesAllRules</code>, in [turing.dom.js](http://github.com/alexyoung/turing.js/blob/master/turing.dom.js) I realised it shouldn't be too hard to make it more generic. It previously took an element and searched its ancestors, but we need to include the current element in the search.

To understand why, consider how <code>findElement</code> should work (this is simplified code):

{% highlight javascript %}
dom.findElement = function(element, selector, root) {
  while (element) {
    if (matchesAllRules(selector, element)) {
      // We've found it!
      return element;
    }
    // Else try again with the parent
    element = element.parentNode;
  }
};
{% endhighlight %}

All I had to do was refactor code that relies on <code>matchesAllRules</code> to pass an element's <code>parentNode</code> instead of the element itself.

The start of the <code>matchesAllRules</code> method now looks slightly different:

{% highlight javascript %}
Searcher.prototype.matchesAllRules = function(element) {
  var tokens = this.tokens.slice(), token = tokens.pop(),
      matchFound = false;
{% endhighlight %}

The code that refers to the <code>ancestor</code> element has been removed and the <code>element</code> argument is used instead.

### The Event Delegation Method

We need to wrap the user's event handler with one that checks the element is one we're interested in, and other than that it's standard Turing event handling:

{% highlight javascript %}
if (turing.dom !== 'undefined') {
  events.delegate = function(element, selector, type, handler) {
    return events.add(element, type, function(event) {
      var matches = turing.dom.findElement(event.target, selector, event.currentTarget);
      if (matches) {
        handler(event);
      }
    });
  };
}
{% endhighlight %}

This code checks to see if <code>dom</code> is available because we don't want interdependence between the modules. Then it sets up a standard event handler and uses <code>findElement</code> to ensure the event is one we're interested in.

### Conclusion

This implementation is very naive and there's no easy delegate event removal, but it illustrates the fundamentals: event delegation depends on existing DOM methods and event handling to be developed in a reusable style.
