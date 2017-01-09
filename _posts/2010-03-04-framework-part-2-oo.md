---
layout: post
title: "Let's Make a Framework: Classes, Inheritance, Extend"
author: Alex Young
categories: 
- web
- frameworks
- tutorials
- lmaf
---

Not all JavaScript frameworks provide classes. [Douglas Crockford](http://javascript.crockford.com) discusses the *classical object model* in [Classical Inheritance in JavaScript](http://javascript.crockford.com/inheritance.html). It's an excellent discussion of ways to implement inheritance in JavaScript. Later, he wrote [Prototypal Inheritance in JavaScript](http://javascript.crockford.com/prototypal.html) in which he basically concludes prototypal inheritance is a strong enough approach without the classical object model.

So why do JavaScript libraries provide tools for OO programming? The reasons vary depending on the author. Some people like to ape an object model from their favourite language. [Prototype](http://prototypejs.org/) is heavily Ruby inspired, and provides [Class](http://prototypejs.org/api/class) which can be useful for organising your own code. In fact, Prototype uses <code>Class</code> internally.

In this article I'm going to explain prototypal inheritance and OO, and start to create a class for OO in JavaScript. This will be used by our framework, [turing.js](http://github.com/alexyoung/turing.js).

### Objects and Classes vs. Prototype Classes

Objects are... everything, so some languages attempt to treat everything as an object. That means a number is an object, a string is an object, a class definition is an object, an instantiated class is an object. The distinction between classes an objects is interesting -- these languages treat classes as objects, and use a more basic object model to implement classes. Remember: it's *object oriented programming* not *class oriented*.

So does that mean JavaScript really needs classical classes? If you're a Java or Ruby programmer you might be surprised to find JavaScript doesn't have a <code>class</code> keyword. That's OK though! We can build our own features if we need them.

### Prototype Classes

Prototype classes look like this:

{% highlight javascript %}
function Vector(x, y) {
  this.x = x;
  this.y = y;
}

Vector.prototype.toString = function() {
  return 'x: ' + this.x + ', y: ' + this.y;
}

v = new Vector(1, 2);
// x: 1, y: 2 
{% endhighlight %}

If you're not used to JavaScript's object model, the first few lines might look strange. I've defined a function called <code>Vector</code>, then said <code>new Vector()</code>. The reason this works is that <code>new</code> creates a new object and then runs the function <code>Vector</code>, with <code>this</code> set to the new object.

The <code>prototype</code> property is where you define instance methods. This approach means that if you instantiate a vector, then add new methods to the <code>prototype</code> property, the old vectors will get the new methods. Isn't that amazing?

{% highlight javascript %}
Vector.prototype.add = function(vector) {
  this.x += vector.x;
  this.y += vector.y;
  return this;
}

v.add(new Vector(5, 5));
// x: 6, y: 7
{% endhighlight %}

### Prototypal Inheritance

There's no formal way of implementing inheritance in JavaScript. If we wanted to make a <code>Point</code> class by inheriting from <code>Vector</code>, it could look like this:

{% highlight javascript %}
function Point(x, y, colour) {
  Vector.apply(this, arguments);
  this.colour = colour;
}

Point.prototype = new Vector;
Point.prototype.constructor = Point;

p = new Point(1, 2, 'red');
p.colour;
// red
p.x;
// 1
{% endhighlight %}

By using <code>apply</code>, <code>Point</code> can call <code>Vector</code>'s constructor. You might be wondering where <code>prototype.constructor</code> comes from. This is a property that allows you to specify the function that creates the object's prototype.

When creating your own objects, you also get some methods for free that descend from <code>Object</code>. Examples of these include <code>toString</code> and <code>hasOwnProperty</code>:

{% highlight javascript %}
p.hasOwnProperty('colour');
// true
{% endhighlight %}

### Prototypal vs. Classical

There are multiple patterns for handling prototypal inheritance. For this reason it's useful to abstract it, and offer extra features beyond what JavaScript has as standard. Defining an API for classes keeps code simpler and makes it easer for people to navigate your code.

The fact that JavaScript's object model splits up portions of a class can be visually noisy. It might be attractive to wrap entire classes up in a definite start and end. Since this is a *teaching framework*, wrapping up classes in discrete and readable chunks might be beneficial.

### A Class Model Implementation Design

The previous example in [Prototype](http://prototypejs.org/) looks like this:

{% highlight javascript %}
Vector = Class.create({
  initialize: function(x, y) {
    this.x = x;
    this.y = y;
  },

  toString: function() {
    return 'x: ' + this.x + ', y: ' + this.y;
  }
});

Point = Class.create(Vector, {
  initialize: function($super, x, y, colour) {
    $super(x, y);
    this.colour = colour;
  }
});
{% endhighlight %}

Let's create a simplified version of this that we can extend in the future. We'll need the following:

1.  The ability to extend classes with new methods by copying them
2.  Class creation: use of <code>apply</code> and <code>prototype.constructor</code> to run the constructors
3.  The ability to determine if a parent class is being passed for inheritance
4.  Mixins

### Extend

You'll find <code>extend</code> littered through Prototype. All it does is copies methods from one <code>prototype</code> to another. This is a good way to really see how prototypes can be manipulated -- it's as simple as you think it is.

The essence of <code>extend</code> is this:

{% highlight javascript %}
for (var property in source)
  destination[property] = source[property];
{% endhighlight %}

### Class Creation

A <code>create</code> method will be used to create new classes. It will need to handle some setup to make inheritance possible, much like the examples above.

{% highlight javascript %}
// This would be defined in our "oo" namespace
create: function(methods) {
  var klass = function() { this.initialize.apply(this, arguments); };

  // Copy the passed in methods
  extend(klass.prototype, methods);

  // Set the constructor
  klass.prototype.constructor = klass;

  // If there's no initialize method, set an empty one
  if (!klass.prototype.initialize)
    klass.prototype.initialize = function(){};

  return klass;
}
{% endhighlight %}

### Get the Code

I've already created a basic OO class for turing in [turing.oo.js](http://github.com/alexyoung/turing.js/blob/master/turing.oo.js). You can read it and experiment with it now.

I'll continue this part next Thursday!
