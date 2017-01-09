---
layout: post
title: "Let's Make a Test Framework"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- testing
---

Welcome to part 39 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Last week we continued building [turing-test.js](https://github.com/alexyoung/turing-test.js), a basic unit testing framework. The idea behind this framework is to build something that follows the CommonJS specifications and works in modern browsers. This will help us test Turing more effectively.

The source code for last week's tutorial can be found in [commit 9a4c33e79cbeb0bf2a71](https://github.com/alexyoung/turing-test.js/commit/9a4c33e79cbeb0bf2a719bc155590bdb14c7390b). This week's code is in [commit 8dd48b5b6da1e26ab7f1](https://github.com/alexyoung/turing-test.js/commit/8dd48b5b6da1e26ab7f194bded8ad4c1f9abae6a).

### Testing Exceptions

[CommonJS Unit Test 1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0) defines <code>assert.throws</code> pretty briefly:

{% highlight javascript %}
// Expected to throw an error:
assert.throws(block, Error_opt, message_opt);
{% endhighlight %}

The error and message arguments are optional. The error argument should check the type of the exception, and the message is an optional message.

When I was reading through [Node's assert.js](https://github.com/ry/node/blob/master/lib/assert.js) I noticed they also define <code>doesNotThrow</code>. I debated the usefulness of this for a while, but I realised there are a lot of cases where testing for no exception is more meaningful than a simple <code>assert.ok</code>, and it should make the tests for the throw assertion easier to write.

If we're going to implement that as well, we'll need a generic <code>throws()</code> function with an option to determine if an exception was expected.

The basic <code>throws</code> function, called by the assertions, looks like this:

{% highlight javascript %}
function throws(expected, block, error, message) {
  // Set up
  try {
    block();
  } catch (e) {
    actual = true;
    exception = e;
  }
  // Test outcome
{% endhighlight %}

If an error was expected and one has been passed, we can test the exception matches like this:

{% highlight javascript %}
exception.constructor != error
{% endhighlight %}

In this case, <code>error</code> is expected to be an exception constructor like <code>CustomException</code>:

{% highlight javascript %}
function CustomException() {
  this.message = 'Custom excpetion';
  this.name = 'CustomException';
}

assert.throws(function() {
  throw new CustomException();
}, CustomException);
{% endhighlight %}

### Exception Assertion Pattern

The structure of the <code>throws</code> and <code>doesNotThrow</code> method looks like this:

1.  Setup
2.  Check if the error parameter is actually a message
3.  Run the function to test and capture the results in a <code>try/catch</code>
4.  If an exception was thrown and one wasn't expected, fail
5.  If an exception was not thrown and one was expected, fail
6.  If an exception was thrown but doesn't match the specific type, fail

Putting this together with the above snippets, we get something like the following:

{% highlight javascript %}
function throws(expected, block, error, message) {
  var exception,
      actual,
      actual = false,
      operator = expected ? 'throws' : 'doesNotThrow';
      callee = expected ? assert.throws : assert.doesNotThrow;

  if (typeof error === 'string' && !message) {
    message = error;
    error = null;
  }

  message = message || '';

  try {
    block();
  } catch (e) {
    actual = true;
    exception = e;
  }

  if (expected && !actual) {
    fail((exception || Error), (error || Error), 'Exception was not thrown\n' + message, operator, callee); 
  } else if (!expected && actual) {
    fail((exception || Error), null, 'Unexpected exception was thrown\n' + message, operator, callee); 
  } else if (expected && actual && error && exception.constructor != error) {
    fail((exception || Error), null, 'Unexpected exception was thrown\n' + message, operator, callee); 
  }
};

assert.throws = function(block, error, message) {
  throws.apply(this, [true].concat(Array.prototype.slice.call(arguments)));
};

assert.doesNotThrow = function(block, error, message) {
  throws.apply(this, [false].concat(Array.prototype.slice.call(arguments)));
};
{% endhighlight %}

Please excuse the crudity of this model, I didn't have time to build it to scale or to paint it.

Notice that we can use the expected result to determine the callee. That means the <code>fail</code> method will correctly track the operator and start stack function.

### Tests

I promised tests of tests, so here they are. This is actually an interesting example because it illustrates how useful <code>doesNotThrow</code> is -- we can use it to test the inverse without doing any hacking to simulate failed exceptions.

{% highlight javascript %}
exports['test throws'] = function() {
  assert.throws(function() {
    throw 'This is an exception';
  });

  function CustomException() {
    this.message = 'Custom excpetion';
    this.name = 'CustomException';
  }

  assert.throws(function() {
    throw new CustomException();
  }, CustomException);

  assert.throws(function() {
    throw new CustomException();
  }, CustomException, 'This is an error');
};

exports['test doesNotThrow'] = function() {
  assert.doesNotThrow(function() {
    return true;
  }, 'this is a message');

  assert.throws(function() {
    throw 'This is an exception';
  }, 'this is a message');
};
{% endhighlight %}

The two and three argument versions are being tested here as well.

### Object.keys

I noticed the code I ported from Node in last week's tutorial included <code>Object.keys</code>. Some browsers don't have that, so I wrote a little function to provide the functionality.

{% highlight javascript %}
function objKeys(o) {
  var result = [];
  for (var name in o) {  
    if (o.hasOwnProperty(name))  
      result.push(name);  
  }  
  return result;  
}
{% endhighlight %}

### Conclusion

That's all of the assertions! We've discovered a few interesting things here, most notably that defining an inverse for <code>assert.throws</code> makes testing the assertions easier.

If you're reading this in the future and you want to check out this version of the project, it's [commit 8dd48b5b6da1e26ab7f1](https://github.com/alexyoung/turing-test.js/commit/8dd48b5b6da1e26ab7f194bded8ad4c1f9abae6a).

### References

-   [CommonJS Unit Test 1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0)
-   [Node's assert.js](https://github.com/ry/node/blob/master/lib/assert.js)
