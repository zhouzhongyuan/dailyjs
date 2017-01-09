---
layout: post
title: "Let's Make a Framework: Chaining"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
---

Welcome to part 28 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

[Last week](http://dailyjs.com/2010/08/26/framework-part-27/) I was talking about chaining DOM finders, jQuery style. I demonstrated a little script that duplicated a fake version of this functionality. This week I'll start building it for real.

Please read [last week's tutorial](http://dailyjs.com/2010/08/26/framework-part-27/) if any of this sounds confusing, everything here draws on that.

### Recap

What we want to be able to do is chain finder methods:

{% highlight javascript %}
turing('.example').find('p')
{% endhighlight %}

This would find things with the class example, then the associated paragraphs. We can use <code>turing.dom.get</code> to implement the core functionality, but <code>get()</code> does not accept a "root" element, so we'll need to add that.

Another thing is, calling <code>turing()</code> makes no sense, because it isn't a function. Let's address that while we're at it.

The alias module will also have to be changed, because it currently wraps <code>turing.dom.get</code> anyway.

### Tests

The implementation should satisfy the following test in <code>test/dom\_test.js</code>:

{% highlight javascript %}
given('chained DOM calls', function() {
  should('find a nested tag', turing('.example3').find('p').length).equals(1);
});
{% endhighlight %}

I should cover more methods and cases, but I'm on a tight schedule here!

### Updating Core

This is simpler that you might expect. The core module currently exposes <code>turing</code> as an object with a bunch of metadata properties. This can be changed to a function to get the jQuery-style API. The only issue is I don't want to make <code>turing.dom</code> a core requirement.

To get around that I'm going to allow an init method to be overridden from outside core. This could be handled in a better way to allow other libraries to extend the core functionality, but let's do it like this for now:

{% highlight javascript %}
function turing() {
  return turing.init.apply(turing, arguments);
}

turing.VERSION = '0.0.28';
turing.lesson = 'Part 28: Chaining';
turing.alias = '$t';

// This can be overriden by libraries that extend turing(...)
turing.init = function() { };

{% endhighlight %}

Then in the DOM library:

{% highlight javascript %}
turing.init = function(selector) {
  return new turing.domChain.init(selector);    
};
{% endhighlight %}

This last snippet is based on the [fakeQuery](http://gist.github.com/551836) example from last week.

### Updating turing.dom

This is all completely taken from the fakeQuery example. The real <code>find</code> method in <code>turing.domChain</code> (which came from <code>fakeQuery.fn</code>) looks like this:

{% highlight javascript %}
find: function(selector) {
  var elements = [],
      ret = turing(),
      root = document;

  if (this.prevObject) {
    if (this.prevObject.elements.length > 0) {
      root = this.prevObject.elements[0];
    } else {
      root = null;
    }
  }

  elements = dom.get(selector, root);
  this.elements = elements;
  ret.elements = elements;
  ret.selector = selector;
  ret.length = elements.length;
  ret.prevObject = this;
  ret.writeElements();
  return ret;
}
{% endhighlight %}

It depends on <code>dom.get</code> for the real work, which I covered way back in [part 6](http://dailyjs.com/2010/04/01/framework-part-6/) (and onwards).

The <code>writeElements</code> method sets each element to a numerical property, so the <code>Array</code>-like API is available:

{% highlight javascript %}
$t('.example3').find('p')[0]
{% endhighlight %}

I also added a shorthand <code>first()</code> method to the same class while I was at it.

### DOM Root

Setting a "root" element for <code>dom.get</code> looks like this:

{% highlight javascript %}
dom.get = function(selector) {
  var tokens = dom.tokenize(selector).tokens,
      root = typeof arguments[1] === 'undefined' ? document : arguments[1],
      searcher = new Searcher(root, tokens);
  return searcher.parse();
};
{% endhighlight %}

An undefined property will become <code>document</code>, which means it can accept <code>null</code>. I had to make the existing <code>find</code> methods check for <code>null</code> as a special case.

### Conclusion

These two chainer tutorials illustrate:

-   How to create an API for jQuery-like DOM traversal
-   How to create a prototype of a feature using abstracted data
-   How to write a test then satisfy it with code

Adapting the fakeQuery prototype was actually surprisingly easy. And now the chainer returns <code>domChain</code>, we can decorate it with lots of helper methods like <code>first()</code> to make DOM traversal very easy.

This is far from a complete solution, but the foundations are now there.
