---
layout: post
title: "Let's Make a Framework: Functional Programming Part 2"
author: Alex Young
categories: 
- web
- frameworks
- tutorials
- lmaf
- functional
---

Welcome to part 5 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues the work last week on functional programming.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/). You can contribute! Fork and message me with your changes.

### More Functional Methods

Last week I intimated that <code>each</code> formed the basis of our functional programming library. This week I'll show you how to add more methods that build on <code>each</code>. I'll draw on inspiration from [Underscore](http://github.com/documentcloud/underscore/) and [Prototype](http://prototypejs.org/), not to mention JavaScript's more recent <code>Array.prototype</code> methods.

#### Filter

Filter allows you to remove values from a list:

{% highlight javascript %}
turing.enumerable.filter([1, 2, 3, 4, 5, 6], function(n) { return n % 2 == 0; });
// 2,4,6
{% endhighlight %}

That means the implementation needs to:

1.  Check if there's a native <code>filter</code> method and use it if possible
2.  Else use <code>turing.enumerable.each</code>
3.  Filter objects into multi-dimensional arrays if required

The tests need to check that both arrays and objects are handled. We already used this approach last week:

{% highlight javascript %}
Riot.context('turing.enumerable.js', function() {
  given('an array', function() {
    var a = [1, 2, 3, 4, 5];

    should('filter arrays', function() {
      return turing.enumerable.filter(a, function(n) { return n % 2 == 0; });
    }).equals([2, 4]);
  });

  given('an object', function() {
    var obj = { one: '1', two: '2', three: '3' };

    should('filter objects and return a multi-dimensional array', function() {
      return turing.enumerable.filter(obj, function(v, i) { return v < 2; })[0][0];
    }).equals('one');
  });
});
{% endhighlight %}

I've tried to be sensible about handling both objects and arrays. Underscore supports filtering objects, but returns a slightly different result (it just returns the value instead of key/value).

#### Detect

Detect is slightly different to <code>filter</code> because there isn't an ECMAScript method. It's easy to use though:

{% highlight javascript %}
turing.enumerable.detect(['bob', 'sam', 'bill'], function(name) { return name === 'bob'; });
// bob
{% endhighlight %}

This class of methods is interesting because it requires an early break. You may have noticed that the <code>each</code> method had some exception handling that checked for <code>Break</code>:

{% highlight javascript %}
each: function(enumerable, callback, context) {
  try {
    // The very soul of each
  } catch(e) {
    if (e != turing.enumerable.Break) throw e;
  }

  return enumerable;
}
{% endhighlight %}

Detect simply uses <code>each</code> with the user-supplied callback, until a truthy value is returned. Then it throws a <code>Break</code>.

### Chaining

We need to be able to chain these calls if we can honestly say turing.js is useful. Chaining is natural when you've overridden <code>Array.prototype</code> like some libraries do, but seeing as we're being good namespacers we need to create an API for it.

I'd like it to look like this (which is different to Underscore):

{% highlight javascript %}
turing.enumerable.chain([1, 2, 3, 4]).filter(function(n) { return n % 2 == 0; }).map(function(n) { return n * 10; }).values();
{% endhighlight %}

Chained functions are possible when each function returns an object that can be used to call the next one. If this looks confusing to you, it might help to break it down:

{% highlight javascript %}
.chain([1, 2, 3, 4])                         // Start a new "chain" using an array
.filter(function(n) { return n % 2 == 0; })  // Filter out odd numbers
.map(function(n) { return n * 10; })         // Multiply each number by 10
.values();                                   // Fetch the values
{% endhighlight %}

To make this possible we need a class with the following features:

-   Store temporary values
-   Runs appropriate methods from <code>turing.enumerable</code> by mapping the temporary value into the first argument
-   After running the method, return <code>this</code> so the chain can continue

This is all easily possible using closures and <code>apply</code>:

{% highlight javascript %}
// store temporary values in this.results
turing.enumerable.Chainer = turing.Class({
  initialize: function(values) {
    this.results = values;
  },

  values: function() {
    return this.results;
  }
});

// Map selected methods by wrapping them in a closure that returns this each time
turing.enumerable.each(['map', 'detect', 'filter'], function(methodName) {
  var method = turing.enumerable[methodName];
  turing.enumerable.Chainer.prototype[methodName] = function() {
    var args = Array.prototype.slice.call(arguments);
    args.unshift(this.results);
    this.results = method.apply(this, args);
    return this;
  }
});
{% endhighlight %}

### Conclusion

Now you know how to:

-   Check for native methods that operate on collections of values
-   Implement them using <code>each</code> where required
-   Break early using an exception
-   Chain methods using closures
-   All in a safely namespaced API!

If you'd like to implement more *enumerable* methods, check out the ones that [Underscore](http://documentcloud.github.com/underscore) supports and port them to turing's style and naming conventions. Add a test or two, let me know via [GitHub](http://github.com/alexyoung/turing.js), and I'll include your contribution and add you to the contributor list.
