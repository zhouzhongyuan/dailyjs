---
layout: post
title: "Let's Make a Framework: OO Part 2"
author: Alex Young
categories: 
- web
- frameworks
- tutorials
- lmaf
---

Welcome to part 3 of *Let's Make a Framework* wherein I discuss building a JavaScript framework suited to both command line and web development. This part completes the discussion of object oriented JavaScript support. In the last part, [Classes, Inheritance, Extend](http://dailyjs.com/2010/03/04/framework-part-2-oo/) I talked about Prototypal inheritance and classical object models. This part will conclude the discussion, explaining more about how <code>turing.oo</code> works, and will explore <code>super()</code>.

If you haven't been following along, I've been tagging these articles with [lmaf](http://dailyjs.com/tags.html#lmaf) so you can find them. The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Class Creation in More Depth

Last week I was parading around <code>initialize</code> as if you were intimately familiar with prototype.js or Ruby. I apologise for that. In case I confused you, all you need to know is <code>initialize</code> is our way of saying *call this method when you set up my class*.

Turing's <code>Class.create</code> method sets up classes. During the setup it defines a function that will be called when the class is instantiated. So when you say <code>new</code>, it will run <code>initialize</code>. The code behind this is very simple:

{% highlight javascript %}
  create: function() {
    var methods = null,
        parent  = undefined,
        klass   = function() {
          this.initialize.apply(this, arguments);
        };
{% endhighlight %}

I sometimes feel like <code>apply</code> is magical, but it's not really *magic* in the negative programmer sense of the word -- it doesn't hide too much from you. In this case it just calls your <code>initialize</code> method against the newly created class using the supplied arguments. Again, the <code>arguments</code> variable seems like magic... but that's just a helpful variable JavaScript makes available when a function is called.

Because I've made this contract -- that all objects will have initialize methods -- we need to define one in cases where classes don't specify initialize:

{% highlight javascript %}
    if (!klass.prototype.initialize)
      klass.prototype.initialize = function(){};
{% endhighlight %}

### Syntax Sugar \* Extend === Mixin

It would be cool to be able to mix other object prototypes into our class. Ruby does this and I've often found it useful. The syntax could look like this:

{% highlight javascript %}
var MixinUser = turing.Class({
  include: User,

  initialize: function(log) {
    this.log = log;
  }
});
{% endhighlight %}

Mixins should have some simple rules to ensure the resulting object compositions are sane:

-   Methods should be included from the specified classes
-   The <code>initialize</code> method should not be overwritten
-   Multiple includes should be possible

Since our classes are being run through <code>turing.oo.create</code>, we can easily look for an <code>include</code> property and include methods as required. Rather than including the bulk of this code in <code>create</code>, it should be in another <code>mixin</code> method in <code>turing.oo</code> to keep <code>create</code> readable.

To satisfy the rules above (which I've turned into unit tests), this pseudo-code is required:

{% highlight javascript %}
  mixin: function(klass, things) {
    if "there are some valid things" {
      if "the things are a class" {
        "use turing.oo.extend to copy the methods over"
      } else if "the things are an array" {
        for "each class in the array" {
          "use turing.oo.extend to copy the methods over"
        }
      }
    }
  },
{% endhighlight %}

### Super

Last week I used an example that featured [prototype.js's](http://prototypejs.org/) <code>super</code> handling. Prototype allows classes to be extended using <code>addMethods</code> so it can track which methods have been added and provide access to overriden methods using <code>$super</code>.

I want to be able to inherit from a class and call methods that I override. Given a <code>User</code> class (from the [fixtures/example\_classes.js](http://github.com/alexyoung/turing.js/blob/master/test/fixtures/example_classes.js) file), we can inherit from it to make a <code>SuperUser</code>:

{% highlight javascript %}
var User = turing.Class({
  initialize: function(name, age) {
    this.name = name;
    this.age  = age;
  },

  login: function() {
    return true;
  },

  toString: function() {
    return "name: " + this.name + ", age: " + this.age;
  }
});

var SuperUser = turing.Class(User, {
  initialize: function() {
    // Somehow call the parent's initialize
  }
});
{% endhighlight %}

A test to make sure the parent's <code>initialize</code> gets called is simple enough:

{% highlight javascript %}
  given('an inherited class that uses super', function() {
    var superUser = new SuperUser('alex', 104);
    should('have run super()', superUser.age).equals(104);
  });
{% endhighlight %}

If I run this without a <code>super</code> implementation, I get this:

{% highlight text %}
Given an inherited class that uses super
  - should have run super(): 104 does not equal: undefined
{% endhighlight %}

To fix this all we need to do is call a previously defined method. Hey, it's time for <code>apply</code> again!

{% highlight javascript %}
var SuperUser = turing.Class(User, {
  initialize: function() {
    User.prototype.initialize.apply(this, arguments);
  }
});
{% endhighlight %}

This isn't perfect though. Most languages make their <code>super</code> implementation simpler for the caller -- forcing people to use <code>apply</code> like this is unwieldy. One way around this is to make the parent class prototype available and add a super method to the class. The super method can simply use <code>apply</code> in the same manner. The only downside is you have to specify the method name:

{% highlight javascript %}
var SuperUser = turing.Class(User, {
  initialize: function() {
    this.$super('initialize', arguments);
  },

  toString: function() {
    return "SuperUser: " + this.$super('toString');
  }
});
{% endhighlight %}

This is simple, lightweight, and easy to understand. The method name could be inferred by other means, but this would complicate the library beyond the scope of this article (meaning if you can do it and make it cross-browser then cool!)

### Conclusion

Now we've got a simple, readable OO class. This will allow us to structure other parts of Turing in a reusable way. I hope the last two articles have demonstrated that JavaScript's simplicity allows you to define your own behaviour for things that feel like fundamental language features.
