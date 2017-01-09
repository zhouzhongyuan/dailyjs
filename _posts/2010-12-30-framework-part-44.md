---
layout: post
title: "Let's Make a Framework: Patterns"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- language
---

Welcome to part 44 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Over the last few weeks we've built a test framework based on the CommonJS assert module, a suitable test runner, and started converting Turing's tests to use it. The test framework is called [Turing Test](https://github.com/alexyoung/turing-test.js).

### Patterns

This week I want to discuss the patterns we've used to create the framework. I realise a lot of new readers find the prospect of reading a 44 part tutorial series daunting, so I thought I'd go back and explore some of the important patterns that you can use in your own JavaScript code.

### Anonymous Functions

Anonymous functions are often used as callbacks for API calls:

{% highlight javascript %}
var a = 1;
MyAPI.method('parameter', function() {
  console.log(a);
});
{% endhighlight %}

Because this is a *closure* the <code>a</code> variable is visible to the anonymous function.

Another important use of anonymous functions is to control scope. This is used throughout our framework to create self-contained modules that only expose a small part of their functionality, thereby creating private scopes:

{% highlight javascript %}
(function(global) {
  function privateFunction() {
    console.log('Hello from privateFunction');
  }

  var MyAPI = {
    method: function() {
      privateFunction();
    }
  };

  global.MyAPI = MyAPI;
})(window);

MyAPI.method();
// Hello from privateFunction
{% endhighlight %}

### CommonJS Module Compatibility

The previous example expects *window* to be defined. It can be made CommonJS module compatible like this:

{% highlight javascript %}
function getGlobal() {
  if (typeof window !== 'undefined') {
    return window;
  } else if (typeof exports !== 'undefined') {
    return exports;
  }
}

(function(global) {
  function privateFunction() {
    console.log('Hello from privateFunction');
  }

  var MyAPI = {
    method: function() {
      privateFunction();
    }
  };

  global.MyAPI = MyAPI;
})(getGlobal() || this);
{% endhighlight %}

Now in a CommonJS-compatible interpreter this should work:

{% highlight javascript %}
> var MyAPI = require('./test').MyAPI;
> MyAPI.method();
{% endhighlight %}

Many predominantly browser-based libraries are now adding CommonJS module compatibility so people can reuse them in server-side projects (Underscore and Backbone are good examples of this).

### Iterators

To get around poor iterator support in some browsers, we had to build our own [enumerable module](https://github.com/alexyoung/turing.js/blob/master/turing.enumerable.js). This is all built from the *each* function, which jQuery defines in its core. We built <code>each</code> like this:

{% highlight javascript %}
turing.enumerable = {
  Break: {},

  each: function(enumerable, callback, context) {
    try {
      if (Array.prototype.forEach && enumerable.forEach === Array.prototype.forEach) {
        enumerable.forEach(callback, context);
      } else if (turing.isNumber(enumerable.length)) {
        for (var i = 0, l = enumerable.length; i < l; i++) callback.call(enumerable, enumerable[i], i, enumerable);
      } else {
        for (var key in enumerable) {
          if (hasOwnProperty.call(enumerable, key)) callback.call(context, enumerable[key], key, enumerable);
        }
      }
    } catch(e) {
      if (e != turing.enumerable.Break) throw e;
    }

    return enumerable;
  },

  // etc.
{% endhighlight %}

This approach is used by many JavaScript frameworks, and works like this:

1.  <code>Array.prototype.forEach</code> is checked to see if we can use the existing functionality based into most interpreters and browsers
2.  Else if the passed-in collection has a length, use a for loop to repeatedly call each value
3.  If the collection is not an Array, use a <code>for ... in ...</code> loop instead
4.  If <code>Break</code> has been thrown by a callback, catch it and return the collection

### Chained APIs

We're used to chained APIs from jQuery (and now a growing number of JavaScript libraries):

{% highlight javascript %}
$('selector')
  .attr('attr', 'value')
  .css({ 'attr', 'value' })
  .find('selector')
    .css({ 'attr', 'value' })
{% endhighlight %}

This is a useful technique because it cuts down the noise that a lot of callbacks would introduce. It's expressive and easy to get the hang of.

We built this in [turing.enumerable.js](https://github.com/alexyoung/turing.js/blob/master/turing.enumerable.js) by defining a <code>Chainer</code> class that collects results, and a method that wraps each *chainable* call into a form that returns *this* after each method has been called.

Returning *this* and collecting results is how jQuery works. Because our module was designed to return results, the chained API style is triggered by calling <code>chain</code> first:

{% highlight javascript %}
turing.enumerable.chain([1, 2, 3, 4])
      .filter(function(n) { return n % 2 == 0; })
      .map(function(n) { return n * 10; })
      .values();
{% endhighlight %}

This is similar to [Underscore](http://documentcloud.github.com/underscore/#chain).

### Prototypal Classes

I've used JavaScript's native prototypal classes in Turing a lot, rather than relying on a library that adds classical object-oriented features. Prototypal classes are easy to get the hang of, and they're a quick way to organise your code:

{% highlight javascript %}
function MyClass() {
  this.instanceVar = 'a dull dude';
}

MyClass.classMethod = function() {
  console.log('Greetings from MyClass');
};

MyClass.prototype.method = function() {
  console.log('All work and no play makes Alex', this.instanceVar);
};

MyClass.classMethod();
var myClass = new MyClass();
myClass.method();
{% endhighlight %}

At first it seems strange using the <code>function</code> keyword to define what appears to be a class, but once you've got used to prototypes it becomes a powerful code organisation tool.

### Prototypal Inheritance

In JavaScript 1.8.5, Mozilla introduced [Object.create](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Object/create) which can be used to inherit from objects:

{% highlight javascript %}
function User() {
}

User.prototype.toString = function() {
  return 'User';
};

User.prototype.state = function() {
  return 'Logged in';
}

function Admin() {
}

// Some browsers won't have Object.create,
// this is a simple version
if (typeof Object.create !== 'function') {
  Object.create = function(o) {
    function F() {}
    F.prototype = o;
    return new F();
  };
}

Admin.prototype = Object.create(User.prototype);

Admin.prototype.toString = function() {
  return 'Admin';
}

var user = new User(),
    admin = new Admin();

console.log('User:', user);
console.log('Admin:', admin);
console.log(user.state());
console.log(admin.state());
{% endhighlight %}

This example uses Crockford's [Object.create](http://javascript.crockford.com/prototypal.html) in browsers and interpreters that don't have <code>Object.create</code>. It creates two prototypal classes, <code>User</code> and <code>Admin</code>, then extends <code>Admin</code>'s prototype to include the methods in <code>User</code>.

### References

-   [Underscore](http://documentcloud.github.com/underscore/)
-   [turing.enumerable.js](https://github.com/alexyoung/turing.js/blob/master/turing.enumerable.js)
-   [Object.create at MDC](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Object/create)
-   [Prototypal Inheritance in JavaScript by Douglas Crockford](http://javascript.crockford.com/prototypal.html)
