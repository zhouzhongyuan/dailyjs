---
layout: post
title: "Let's Make a Framework: More on Chained Events"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- events
- dom
---

Welcome to part 35 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Last week I talked about <code>NodeList</code> and converting it into an <code>Array</code>. This week I'm going to improve the chained API for events.

### Event Handler Shortcuts and Loop Scoping

The only DOM event handler I added to our chained API was <code>click</code>. Let's add more of the events named after the HTML 'on' attributes. I used the list on [MDC's Event Handlers](https://developer.mozilla.org/en/DOM/element#Event_Handlers) page as a reference, and set up aliases like this:

{% highlight javascript %}
events.addDOMethods = function() {
  if (typeof turing.domChain === 'undefined') return;

  turing.domChain.bind = function(type, handler) {
    var element = this.first();
    if (element) {
      turing.events.add(element, type, handler);
      return this;
    }
  };

  var chainedAliases = ('click dblclick mouseover mouseout mousemove ' +
                        'mousedown mouseup blur focus change keydown ' +
                        'keypress keyup resize scroll').split(' ');

  for (var i = 0; i < chainedAliases.length; i++) {
    (function(name) {
      turing.domChain[name] = function(handler) {
        return this.bind(name, handler);
      };
    })(chainedAliases[i]);
  }
};
{% endhighlight %}

The technique I used to create an array from a string is found throughout jQuery. It's handy because it has less syntax than lots of commas and quotes. I use the anonymous function to capture the name parameter for each alias, doing it with <code>var name = chainedAliases\[i\]</code> would bind <code>name</code> to the last value executed, which isn't what we want.

In jQuery's code they use <code>jQuery.each</code> to iterate over the event names, which actually reads better. I put our iterators in [turing.enumerable.js](http://github.com/alexyoung/turing.js/blob/master/turing.enumerable.js) and have been avoiding interdependence between modules, so I'm doing it the old fashioned way.

However, doing it this way does illustrate an interesting point about JavaScript's lexical scoping and closures. Try this example in a prompt or browser console:

{% highlight javascript %}
var methods = {}, 
    items = ['a', 'b', 'c'];

for (var i = 0; i < items.length; i++) {
  var item = items[i];

  methods[item] = function() {
    console.log('Item is: ' + item);
  };
}

console.log('After the for loop, item is: ' + item);

for (var name in methods) {
  console.log('Calling: ' + name);
  methods[name]();
}
{% endhighlight %}

This will result in:

{% highlight sh %}
After the for loop, item is: c
Calling: a
Item is: c
Calling: b
Item is: c
Calling: c
Item is: c
{% endhighlight %}

Why is each item set to 'c' when each function is called? In JavaScript, variables declared in <code>for</code> are in the same scope rather than a new local scope. That means there aren't three <code>item</code> variables, there is just one. And this is why jQuery's version is more readable. I've edited this version of jQuery's [events.js](http://github.com/jquery/jquery/blob/master/src/event.js) to illustrate the point:

{% highlight javascript %}
jQuery.each(aliases, function(i, name) {
  jQuery.fn[name] = function(data, fn) {
    return this.bind(name, data, fn);
  };
});
{% endhighlight %}

Variables declared inside <code>jQuery.each</code> are effectively in a different scope, and of course the <code>name</code> parameter passed in on each iteration is the one we want.

### Trigger vs. Bind

Calling <code>jQuery().click()</code> without a handler actually fires the event, which I've always liked. Can we do something similar?

We just need to check if there's a handler in <code>turing.domChain.bind</code>:

{% highlight javascript %}
if (handler) {
  turing.events.add(element, type, handler);
} else {
  turing.events.fire(element, type);
}
{% endhighlight %}

While I was looking at <code>turing.domChain.bind</code> I changed it to bind to all elements instead of the first one. I thought that way felt more natural.

You could do a quick test with this:

{% highlight javascript %}
$t('p').click(function(event) {
  event.target.style.backgroundColor = '#ff0000'
});
{% endhighlight %}

It'll bind to all of the paragraphs instead of just the first one.

### Conclusion

Generating sets of functions or method aliases iteratively is straightforward in JavaScript, but you need to be aware of scope and pay attention to closures. It's easy to see why <code>jQuery.each</code> is in [core.js](http://github.com/jquery/jquery/blob/master/src/core.js) -- it makes framework code simpler, especially given how jQuery handles context.
