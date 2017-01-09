---
layout: post
title: "Let's Make a Framework: Functional Programming"
author: Alex Young
categories: 
- web
- frameworks
- tutorials
- lmaf
- functional
---

Welcome to part 4 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part introduces the *functional* part of the library, with a brief introduction to functional programming.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Functional Programming 101

The name *functional programming* is annoying because it reminds novices of *procedural programming*, which people learn first then discard in favour of object oriented programming. Forget all that for a bit.

Functional programming is about:

-   Describing problems rather than focusing on the mechanics of their solution
-   Treating functions as first class citizens, and manipulating them like variables
-   Avoiding state and mutable data

There are functional programming languages like Erlang and Haskell. JavaScript isn't strictly functional, but neither are Ruby, Python, or Java. However, these languages use ideas from functional programming to simplify common programming tasks.

Functional languages usually focus on lists, and treat functions as *first class* and have useful features like closures. JavaScript is actually pretty good at functions and closures, and JavaScript arrays and objects are similar to lists and [property lists](http://www.cs.cmu.edu/Groups/AI/html/cltl/clm/node108.html) in lisp-like languages.

I once saw a talk by Tom Stuart called [Thinking Functionally in Ruby](http://skillsmatter.com/podcast/ajax-ria/enumerators). It has some animations that *really* make functional programming paradigms easy to understand. If you're still finding the concept nebulous try watching his presentation.

### Iterators

JavaScript frameworks use <code>each</code> a lot. Some frameworks define classes early on, but almost all define this basic tool for iteration.

Ruby programmers use <code>each</code> all the time:

{% highlight ruby %}
[1, 2, 3].each { |number| puts number }
# 1
# 2
# 3
{% endhighlight %}

This sends a block to <code>each</code>, and <code>each</code> runs the block multiple times. [Enumerable](http://ruby-doc.org/core/classes/Enumerable.html) uses <code>each</code> to create lots of other methods that are inspired by functional languages. Any collection-style object can mixin Enumerable to get all those methods for free.

Equivalent JavaScript could look like this:

{% highlight javascript %}
Array.prototype.each = function(callback) {
  for (var i = 0; i < this.length; i++) {
    callback(this[i]);
  }
}

[1, 2, 3].each(function(number) {
  print(number);
});
{% endhighlight %}

However, JavaScript actually has <code>Array.forEach</code>, <code>Array.prototype.forEach</code>, <code>for (var i in objectWithIterator)</code> and even more ways to iterate. So why do frameworks bother defining their own method? One of the reasons you see [jQuery.each()](http://api.jquery.com/jQuery.each/) and [each in Prototype](http://www.prototypejs.org/api/enumerable/each) is because browser support is inconsistent.

You can see the source for jQuery's each in [core.js](http://github.com/jquery/jquery/blob/master/src/core.js).

Prototype's implementation uses <code>forEach</code> if it exists:

{% highlight javascript %}
(function() {
  var arrayProto = Array.prototype,
      slice = arrayProto.slice,
      _each = arrayProto.forEach; // use native browser JS 1.6 implementation if available
{% endhighlight %}

[Underscore](http://github.com/documentcloud/underscore) uses a similar approach:

{% highlight javascript %}
  // The cornerstone, an each implementation.
  // Handles objects implementing forEach, arrays, and raw objects.
  // Delegates to JavaScript 1.6's native forEach if available.
  var each = _.forEach = function(obj, iterator, context) {
    try {
      if (nativeForEach && obj.forEach === nativeForEach) {
        obj.forEach(iterator, context);
      } else if (_.isNumber(obj.length)) {
        for (var i = 0, l = obj.length; i < l; i++) iterator.call(context, obj[i], i, obj);
      } else {
        for (var key in obj) {
          if (hasOwnProperty.call(obj, key)) iterator.call(context, obj[key], key, obj);
        }
      }
    } catch(e) {
      if (e != breaker) throw e;
    }
    return obj;
  };
{% endhighlight %}

This approach uses JavaScript's available datatypes and features, rather than mixing an Enumerable-style class into objects and <code>Array</code>.

### Benchmarks

I've written some benchmarks to test each implementation. You can view them here: [test.html](/files/functional/test.html), and [iteratortest.js](/files/functional/iteratortest.js).

Each will form a cornerstone of Turing's functional programming features, so let's create a benchmark to see if the native function is really faster.

|               |            |          |           |          |          |            |            |             |            |
|---------------|------------|----------|-----------|----------|----------|------------|------------|-------------|------------|
|               | Rhino      | Node     | Firefox   | Safari   | Chrome   | Opera      | IE8        | IE7         | IE6        |
| eachNative    | **1428ms** | 69ms     | **709ms** | 114ms    | 62ms     | 1116ms     |            |             |            |
| eachNumerical | 2129ms     | **55ms** | 904ms     | **74ms** | **58ms** | **1026ms** | **3674ms** | **10764ms** | **6840ms** |
| eachForIn     | 4223ms     | 309ms    | 1446ms    | 388ms    | 356ms    | 2378ms     | 4844ms     | 21782ms     | 14224ms    |

The native method performs well, and generally close to the simple for loop. This probably explains why most JavaScript library implementors use it when it's there. And <code>for ... in</code> performs so terribly in Internet Explorer that we need to be careful about using it.

### API Design

An important consideration is the API design for functional features. The Prototype library modifies <code>Object</code> and <code>Array</code>'s prototypes. This makes the library easy to use, but makes it difficult to use concurrently with other libraries: it isn't safely namespacing.

[Underscore](http://documentcloud.github.com/underscore/) has clearly namespaced design, with optional use for what it calls functional or object-oriented (which allows chained calls).

Our library could look like this:

{% highlight javascript %}
turing.enumerable.each([1, 2, 3], function(number) { number + 1; });
turing.enumerable.map([1, 2, 3], function(number) { return number + 1; });
// 2, 3, 4
{% endhighlight %}

These methods could be mapped to shorthands later on.

### Tests

A basic test of <code>each</code> and <code>map</code> should ensure arrays are iterated over:

{% highlight javascript %}
Riot.context('turing.enumerable.js', function() {
  given('an array', function() {
    var a = [1, 2, 3, 4, 5];
    should('iterate with each', function() {
      var count = 0;
      turing.enumerable.each(a, function(n) { count += 1; });
      return count;
    }).equals(5);

    should('iterate with map', function() {
      return turing.enumerable.map(a, function(n) { return n + 1; });
    }).equals([2, 3, 4, 5, 6]);
  });
});
{% endhighlight %}

Objects should also work:

{% highlight javascript %}
given('an object', function() {
  var obj = { one: '1', two: '2', three: '3' };
  should('iterate with each', function() {
    var count = 0;
    turing.enumerable.each(obj, function(n) { count += 1; });
    return count;
  }).equals(3);

  should('iterate with map', function() {
    return turing.enumerable.map(obj, function(n) { return n + 1; });
  }).equals(['11', '21', '31']);
});
{% endhighlight %}

The resulting implementation is based heavily on Underscore, because it's easy to understand and my benchmarks show it's pretty smart. View the code here: [turing.enumerable.js](http://github.com/alexyoung/turing.js/blob/8e9cd9a479b92e69371513b58de14968ac8a9474/turing.enumerable.js)

### Conclusion

In this part of *Let's Make a Framework*, I've introduced the basic premise of functional programming. The key to this in JavaScript is <code>each</code>, which will let us implement lots of useful methods to use on collections. I've benchmarked different ways of implementing <code>each</code> and discussed how different libraries make this functionality available.

You might remember me mentioning *mutable data*. Some programming languages don't let you modify data, so everything changed becomes a copy. Optimisation can actually make this fast, and in some cases, due to the advantages of lazy evaluation, performance isn't a great problem. I've previously tried to implement lazy data types in JavaScript, so I'll see if I can work this into the framework in the future.

Further reading:

-   For a different iterator implementation, try [callbacks and for](http://plasmasturm.org/log/311/)
-   [Underscore](http://documentcloud.github.com/underscore/) has great documentation
-   [Iterators in JavaScript 1.7](https://developer.mozilla.org/en/New_in_JavaScript_1.7#Iterators)
-   [How many ways can you iterate?](http://ajaxian.com/archives/how-many-ways-can-you-iterate-over-an-array-in-javascript)
-   [Clojure](http://clojure.org/) is predominantly functional and should appeal to JavaScripters
