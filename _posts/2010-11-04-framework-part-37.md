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

Welcome to part 37 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Last week I was discussing how frameworks write and package their tests. Looking at other frameworks was interesting, and made me realise that using the CommonJS assert and test modules would be a good idea. However, I couldn't find a suitable library.

I don't like writing code for the sake of it, so typically I'd make do with something like QUnit. However, I realised that *test* frameworks are full of interesting code, in particular assertions. So rather than reusing something, let's make our own CommonJS test framework for Turing that will run in Node, Rhino, and a browser.

Plus, if we create our own unit testing library, we can call it **Turing Test**!

### Test Module

In [Unit Testing 1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0), CommonJS defines how the <code>test</code> module should work. It's essentially the same as most unit testing frameworks:

-   The test module provides a <code>run</code> method
-   This accepts an object
-   It will run any methods that start with but aren't equal to <code>test</code>
-   Sub-objects will also run as sub-tests if their names match in the same way
-   The <code>run</code> method will catalog the results of the tests

That means tests can look as simple as this:

{% highlight javascript %}
var assert = require('assert');

require('test').run({
  testEqual: function() {
    assert.equal(true, true, 'True should be true');
  },

  testAGroupOfThings: {
    testOK: function() {
      assert.ok("I'm OK!", "If you're OK you're OK, OK?");
    }
  }
});
{% endhighlight %}

### Test Organisation

In real projects, tests are usually organised into several categorised files like this:

{% highlight javascript %}
exports['test equal'] = function() {
  assert.equal(true, true, 'True should be true');
}

exports['test ok'] = function() {
  assert.ok(true, 'True should be OK');
}
{% endhighlight %}

Then a test runner can be written that requires each file and calls <code>require('test').run()</code> on it.

### In a Browser

Running these tests in a browser will require a harness that can provide <code>require</code> and generate suitable HTML reports.

We don't really need to use <code>require</code> at all, so tests could take this pattern:

{% highlight javascript %}
if (typeof require !== 'undefined') {
  if (typeof require.paths !== 'undefined')
    require.paths.unshift('lib/');
  var assert = require('assert');
}

exports['test equal'] = function() {
  assert.equal(true, true, 'True should be true');
};

exports['test ok'] = function() {
  assert.ok(true, 'True should be OK');
};

if (typeof module !== 'undefined')
  if (module === require.main) 
    require('test').run(exports);
{% endhighlight %}

The last two lines are recommended by the CommonJS specification. It makes running single test files possible, like this:

{% highlight sh %}
node test/assertions.js
{% endhighlight %}

This is important because it's useful to run individual test files during development. The test suit will be run less often, perhaps as part of a build process or automatically before deployment.

The only problem with this boilerplate code is it'll have to appear in every file. We could write a test runner script that wraps this, but that would end up being a test runner... runner.

### Basic Assertions

Let's look at two very basic assertions. Writing assertions is actually what motivated me to write this tutorial -- they contain a lot of interesting JavaScript code.

I've based these assertions on <code>assert.js</code> from Node, because it's a very neat little library. These assertions should only really be required in the browser, because platforms like Node and Narwhal already provide them as part of their CommonJS libraries.

The way assertions generally work is to define a <code>fail</code> method that gets called with details about the assertion, and then raises an exception that the test runner will look for -- in this case <code>AssertionError</code>. The test runner can then differentiate between failed assertions and other exceptions.

The signature of <code>fail</code> reveals how any assertion can be written:

{% highlight javascript %}
function fail(actual, expected, message, operator, stackStartFunction) {
  throw new assert.AssertionError({
    message: message,
// etc.
{% endhighlight %}

Almost any assertion you could think up has an actual and expected value.

Messages can be used to make failures clearer. This is a common convention, yet I wrote unit tests in other languages for years before I realised how useful messages are.

In the case of <code>assert.ok</code>, the expected value is always <code>true</code>. This assertion just looks for *truthy* values, or as the spec says:

> Pure assertion tests whether a value is truthy, as determined by !!guard

Which means <code>assert.equal</code> is very similar:

{% highlight javascript %}
assert.equal = function(actual, expected, message) {
  if (actual != expected)
    fail(actual, expected, message, '==', assert.equal);  
};
{% endhighlight %}

The concept of equality is a little bit tricky, given that objects aren't just simple values. The CommonJS specification addresses this with the <code>assert.deepEqual</code> assertion. We can look at this in a future tutorial.

### The Test Runner

The test runner should:

-   Run each test
-   Gather results
-   Display results

Displaying results is the interesting part for browser testing. I've kept the code very simple for this part of the tutorial though, fancy styles can come later.

{% highlight javascript %}
printMessage = (function() {
  if (typeof window !== 'undefined') {
    return function(message) {
      var li = document.createElement('li');
      li.innerHTML = message;
      document.getElementById('results').appendChild(li);
    }
  } else if (typeof console !== 'undefined') {
    return console.log;
  } else {
    return function() {};
  }
})();
{% endhighlight %}

The anonymous function is used to return a function we can reuse depending on the environment. It checks if <code>window</code> is available, and if so assumes the environment is a browser. The results are generated without relying on a JavaScript client-side framework, and Node will provide <code>console.log</code> for us.

The structure of the test runner collects fail, pass, and error counts. This allows it to display a set of results after running through all the tests. Exceptions will be displayed during run time. I haven't really made much effort to display backtraces because it seemed too much for this tutorial, but I may come back to it later.

Gathering results just needs a <code>Test</code> class that can run tests and keep counters. Tests are run in an exception handler. This is pretty simple:

{% highlight javascript %}
try {
  // TODO: Setup
  obj[testName]();
  this.passed += 1;
} catch (e) {
  if (e.name === 'AssertionError') {
    result.message = e.toString();
    logger.fail('Assertion failed in: ' + testName);
    showException(e);
    this.failed += 1;
  } else {
    logger.error('Error in: ' + testName);
    showException(e);
    this.errors += 1;
  }
} finally {
  // TODO: Teardown
}
{% endhighlight %}

Notice where I've marked those TODO comments. Setup and teardown methods are fairly common in JavaScript frameworks, so we could support those in the future.

As it turns out Node doesn't currently have a test module, so the one I intended to be used by the browser will get used. That's why I was conditionally messing around with require paths in the unit test boilerplate code.

### Conclusion

I don't actually intend turing-test to be a definitive test framework, it's merely part of the *Let's Make a Framework tutorial* series. However, because tests give a deep insight into the language, I hope this part of the series helps you sharpen up your JavaScript skills.

Our goals are modest: to write a test framework that facilitates browser and console unit tests which uses CommonJS. Yet writing test frameworks can reveal a lot about the language.

I wrote some proof-of-concept code and posted it under [turing-test.js](https://github.com/alexyoung/turing-test.js) on GitHub. I only wrote it to make sure browsers wouldn't explode on the code and destroy all lifeforms in a 10 mile radius<sup>[1](#footnote-1</sup>). As this part of *Let's Make a Framework* continues hopefully it'll become something more useful.

### Resources

If you want to build the next awesome test framework, get a head start:

-   [CommonJS Unit Test 1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0)
-   [Narwhal's tests](https://github.com/280north/narwhal/tree/master/lib/)
-   [Narwhal's test runner](https://github.com/280north/narwhal/blob/5b42fabf4444b07a668d6011f2d29c93b33b9074/lib/test/runner.js)
-   [Node's assert.js](https://github.com/ry/node/blob/master/lib/assert.js)

<p id="footnote-1">
\[1\] I've been playing Fallout: New Vegas

</p>
