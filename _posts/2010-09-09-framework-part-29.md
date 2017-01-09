---
layout: post
title: "Let's Make a Framework: Chaining Events"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
---

Welcome to part 29 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

[Last week](http://dailyjs.com/2010/09/02/framework-part-28/) I demonstrated a jQuery-like API for working with DOM finders. This week we'll build on that to add event support.

### API Design

We want to be able to do this:

{% highlight javascript %}
turing('#element').bind('click', function(e) {
  alert('Stop clicking me!');
});
{% endhighlight %}

If you haven't read the other chaining tutorials, this might not seem interesting. The reason we're doing this is to get a chainable API for DOM finders, like jQuery. So multiple finders could be called:

{% highlight javascript %}
turing('#element').find('.element li a').bind('click', function(e) {
  alert('Stop clicking me!');
});
{% endhighlight %}

Adding events with Turing is performed with <code>turing.events.add(element, 'event name', callback)</code>. I'll use the method name <code>bind</code> instead of <code>add</code> so it doesn't look confusing next to DOM-manipulation code.

### Test

We need this test to pass (in [test/events\_test.js](http://github.com/alexyoung/turing.js/blob/master/test/events_test.js)):

{% highlight javascript %}
should('bind events using the chained API', function() {
  var clicks = 0;
  turing('#events-test a').bind('click', function() { clicks++; });
  turing.events.fire(element, 'click');
  return clicks;
}).equals(1);
{% endhighlight %}

Running it right now results in an error:

> **should bind events using the chained API**: 1 does not equal: TypeError: Result of expression 'turing('\#events-test a').bind' \[undefined\] is not a function.

### Implementation

It seems like we can just alias <code>bind</code> to <code>add</code> with a bit of currying, but that doesn't fit with our style of keeping each module independent (else <code>turing.dom</code> will rely on <code>turing.events</code>).

However, last week I exposed the object that is returned through the chained DOM calls: <code>turing.domChain</code>. Let's try extending that from the events API if it's available.

In [turing.events.js](http://github.com/alexyoung/turing.js/blob/master/turing.events.js):

{% highlight javascript %}
events.addDOMethods = function() {
  // If there's no domChain then the DOM module hasn't been included
  if (typeof turing.domChain === 'undefined') return;

  // Else it's safe to add the bind method
  turing.domChain.bind = function(type, handler) {
    var element = this.first();
    if (element) {
      turing.events.add(element, type, handler);

      // NOTE: "this" refers to the current domChain object,
      //       which contains the stack of elements
      return this;
    }
  };
};

// It's safe to always run addDOMethods when
// the events module is loaded
events.addDOMethods();
{% endhighlight %}

I've commented each part, but it's fairly straightforward.

### Building

What's interesting about this approach is that <code>events.addDOMethods</code> is not private. People could include scripts in any order, and get the functionality as long as they call <code>turing.events.addDOMethods()</code>.

Obviously load order is important here, so the script that "builds" Turing into a single file ([Jakefile](http://github.com/alexyoung/turing.js/blob/master/Jakefile)) should be aware that [turing.dom.js](http://github.com/alexyoung/turing.js/blob/master/turing.dom.js) should come before [turing.events.js](http://github.com/alexyoung/turing.js/blob/master/turing.events.js):

{% highlight javascript %}
jake.task('concat', function(t) {
  var output = '',
      files = ('turing.core.js turing.oo.js turing.enumerable.js '
              + 'turing.functional.js turing.dom.js turing.events.js turing.alias.js turing.anim.js').split(' ')
{% endhighlight %}

I've written each module name by hand in the correct order, rather than reading the file list from the file system.

### Conclusion

Adding events support to our chainable DOM API was easier than I expected. This could be expanded on with aliases to make it more user-friendly (jQuery makes using events more intuitive through aliases).
