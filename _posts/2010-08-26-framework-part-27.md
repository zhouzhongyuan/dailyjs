---
layout: post
title: "Let's Make a Framework: Chaining"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
---

Welcome to part 27 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

### Namespaces and Chaining

Throughout this series I've referenced techniques used by widely-used frameworks like jQuery and Prototype. Prototype packs a lot of functionality and extends global JavaScript objects to do this.

jQuery takes a different approach. It uses large module-like chunks of functionality wrapped in closures, then specific parts are exposed through the <code>jQuery</code> object (we usually write <code>$()</code> instead).

Turing has been designed in a similar way to jQuery -- to carefully keep implementation details private and make functionality available without polluting global objects.

One drawback of our current implementation is everything takes a lot of typing. Disregarding the alias we created, code looks like this:

{% highlight javascript %}
var element = turing.dom.get('#events-test a')[0];
turing.events.add(element, 'click', callback);

// Or...
turing.events.add(turing.dom.get('#events-test a')[0], 'click', callback);
{% endhighlight %}

We'd do this in jQuery:

{% highlight javascript %}
$('#events-test').click(function() {
  // Handler
});
{% endhighlight %}

In this case, <code>click</code> is a shortcut, so the following is equivalent:

{% highlight javascript %}
$('#events-test').bind('click', (function() {
  // Handler
});
{% endhighlight %}

This chaining can go on as long as you want. jQuery even provides tools for popping up to different points in a chained result stack, like <code>end()</code>:

{% highlight javascript %}
$('ul.first').find('.selected')
  .css('background-color', 'red')
.end().find('.hover')
  .css('background-color', 'green')
.end();
{% endhighlight %}

This works particularly well when working with DOM traversal.

What this style of API gives us is the safety of namespaced code with the power and succinctness of prototype hacking, without actually modifying objects that don't belong to us.

### API

The way this works in jQuery is <code>jQuery()</code> accepts a selector and returns an array-like **jQuery object**. The returned object has a <code>length</code> property, and each element can be accessed with square brackets. It's not a true JavaScript <code>Array</code>, just something similar enough.

Each call in the chain is operating on a <code>jQuery</code> object, which means all of the appropriate methods are available.

### Previously...

We've already seen a combination of aliasing and currying to create a chainable API in Turing -- check out *turing.enumerable.js* and *turing.anim.js*. In these cases, API calls were chained based on the first parameter -- the first parameter for functions in these classes was always a certain type, so we could shortcut this and create a chain.

This is really a case of [currying](http://en.wikipedia.org/wiki/Currying), and is one of those fine examples of a nice bit of functional programming in JavaScript.

### fakeQuery

jQuery's chaining is based around the DOM, so the previous examples don't really help. Rather than jumping straight into Turing code, I've created a little class you can play with called <code>fakeQuery</code>. This will illustrate what underpins jQuery.

It uses a mock up of the DOM so it has something to query:

{% highlight javascript %}
var dom = [
  { tag: 'p', innerHTML: 'Test 1', color: 'green' },
  { tag: 'p', innerHTML: 'Test 2', color: 'black' },
  { tag: 'p', innerHTML: 'Test 3', color: 'red' },
  { tag: 'div', innerHTML: 'Name: Bob' }
];
{% endhighlight %}

It's not a particularly accurate representation of the DOM, but it's readable.

This is the core function:

{% highlight javascript %}
function fakeQuery(selector) {
  return new fakeQuery.fn.init(selector);
}
{% endhighlight %}

It returns a new object based on an <code>init</code> method. The <code>init</code> method builds an object which can carry around the current selector and related elements:

{% highlight javascript %}
fakeQuery.fn = fakeQuery.prototype = {
  init: function(selector) {
    this.selector = selector;
    this.length = 0;
    this.prevObject = null;

    if (!selector) {
      return this;
    } else {
      return this.find(selector);
    }
  },

  find: function(selector) {
    // Finds elements
    // Returns a new fakeQuery
  },

  color: function(value) {
    // Creates a copy of the current elements
    // Changes them
    // Returns a fakeQuery object with these elements
  }
};

fakeQuery.fn.init.prototype = fakeQuery.fn;
{% endhighlight %}

The <code>prevObject</code> property could be used to implement <code>end()</code> (mentioned above). The full code is in a gist: [fakeQuery](http://gist.github.com/551836). This code uses Node, but you could delete the Node-related parts if you want to run it with Rhino.

Running this code with something like <code>fakeQuery('p').color('red').elements</code> will produce:

{% highlight javascript %}
[ { tag: 'p', innerHTML: 'Test 1', color: 'red' }
, { tag: 'p', innerHTML: 'Test 2', color: 'red' }
, { tag: 'p', innerHTML: 'Test 3', color: 'red' }
]
{% endhighlight %}

### Overall Pattern

The overall architecture of jQuery is deceptively simple:

-   A "container function" is used to create new objects without having to type <code>new</code>
-   It returns objects based on a CSS selector
-   A class is created and copied so usage of methods like <code>find</code> can be called in a chain

The key to the last part is <code>fakeQuery.fn.init.prototype = fakeQuery.fn;</code>. This line is what allows the <code>init</code> method reference <code>fakeQuery.prototype</code>. You can try running the code without this if you want to see what happens.

### Conclusion

jQuery's design offers an efficient way of traversing and modifying the DOM. This is attractive to us because we're building a framework without modifying global objects in the way [Prototype](http://prototypejs.org/) does.

Next week I'll look at building this into Turing. The interesting challenge here is that as it stands there are no dependencies between Turing's components.
