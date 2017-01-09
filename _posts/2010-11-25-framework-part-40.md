---
layout: post
title: "&#9760; Let's Make a Test Framework"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- testing
---

Welcome to part 40 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Last week we continued building [turing-test.js](https://github.com/alexyoung/turing-test.js), a unit testing framework. The idea behind this framework is to build something that follows the CommonJS specifications and works in modern browsers. This will help us test Turing more effectively.

Previous parts:

-   [Part 36](http://dailyjs.com/2010/10/28/framework-part-36/) - An introduction to how JavaScript frameworks handle testing
-   [Part 37](http://dailyjs.com/2010/11/04/framework-part-37/) - A review of CommonJS test specifications, and examples of how to run assert module tests in a browser with a simple test runner
-   [Part 38](http://dailyjs.com/2010/11/11/framework-part-38/) and [commit 9a4c33e](https://github.com/alexyoung/turing-test.js/commit/9a4c33e79cbeb0bf2a719bc155590bdb14c7390b) - Assertions part 1
-   [Part 39](http://dailyjs.com/2010/11/18/framework-part-39/) and [commit 8dd48b5](https://github.com/alexyoung/turing-test.js/commit/8dd48b5b6da1e26ab7f194bded8ad4c1f9abae6a) - Assertions part 2

### Test Output Design

It's time to make the test results easier to read. The test runner runs each method that is prefixed with *test*, as per the CommonJS specifications. I'd like it to display results a little bit like the jQuery test suit.

The test runner should list each test method that was run. Because we can write test method names with full text rather than a JavaScript function name, the output should be easy to follow:

{% highlight javascript %}
exports['test strictEqual'] = function() {
  assert.strictEqual('1', '1', "'1' should be equal to '1'");
  assert.strictEqual(1, 1, '1 should be equal to 1');
};
{% endhighlight %}

The test runner could display this test as:

{% highlight javascript %}
[OK] test strictEqual
{% endhighlight %}

Another thing to consider is the design of failed tests. The code we wrote way back at the start of this test framework detour already checks to see if exceptions have corresponding stack properties:

{% highlight javascript %}
run: function(testName, obj) {
  var result = new Tests.Result(testName);

  function showException(e) {
    if (!!e.stack) {
      logger.display(e.stack);
    } else {
      logger.display(e);
    }
  }
{% endhighlight %}

We should see a full stack trace in the console, but browsers might not display it. At its most basic, failed tests should look like this:

{% highlight javascript %}
✕ Assertion failed in: test strictEqual
  AssertionError: '1' should be equal to '1'
  In "===":
    Expected: 2
    Found: 1
{% endhighlight %}

The HTML output will require styles to preserve white space for stacktraces.

### Colours and Prefixes

In HTML and consoles that support colour, red and green can be used to indicate pass or fail. We'll also need another visual indicator for red/green colourblind people. I've opted to use HTML entities and UTF-8 symbols for these indicators:

-   Error is a skull and crossbones: ☠
-   Pass is a tick: ✓
-   Fail is a cross: ✕

This is purely superficial, I just thought readers might find it more interesting than text. There's a switch statement in [lib/test.js](https://github.com/alexyoung/turing-test.js/blob/5a6cbf61cdad29e688d8c1288c65ce83867f1122/lib/test.js) that converts the HTML entities to JavaScript UTF-8 codes for the console.

Colours are also converted, but this time based on the message type. The message type is used as a CSS class name, and is also used to determine the console colour:

{% highlight javascript %}
function messageTypeToColor(messageType) {
  switch (messageType) {
    case 'pass':
      return '32';
    break;

    case 'fail':
      return '31';
    break;
  }

  return '';
}

// ...

var col    = colorize ? messageTypeToColor(messageType) : false;
  startCol = col ? '\033[' + col + 'm' : '',
  endCol   = col ? '\033[0m' : '',
console.log(startCol + (prefix ? htmlEntityToUTF(prefix) + ' ' : '') + message + endCol);
{% endhighlight %}

All of these functions are embedded within a closure, and they're only evaluated if they're needed, with this simple pattern:

{% highlight javascript %}
printMessage = (function() {
  function htmlEntityToUTF(text) {
    // Removed for clarity
  }

  function messageTypeToColor(messageType) {
    // Removed for clarity
  }

  if (typeof window !== 'undefined') {
    return function(message, messageType, prefix) {
      // Display message with some simple DOM code
    }
  } else if (typeof console !== 'undefined') {
    return function(message, messageType, prefix) {
      // Display message with console.log()
    };
  } else {
    return function() {};
  }
})();
{% endhighlight %}

After this closure is evaluated, <code>printMessage</code> contains everything it needs to display messages. Then all that's required is a helper logger object:

{% highlight javascript %}
logger = {
  display: function(message, className, prefix) {
    printMessage(message, className || 'trace', prefix || '');
  },

  error: function(message) {
    this.display(message, 'error', '&#9760;');
  },

  pass: function(message) {
    this.display(message, 'pass', '&#10003;');
  },

  fail: function(message) {
    this.display(message, 'fail', '&#10005;');
  }
};
{% endhighlight %}

### AssertionError toString

The <code>AssertionError</code> exceptions will need to carefully handle <code>toString</code> to ensure that exceptions are readable.

The way I like to think of failed assertions is they have a *summary* and *extended details*. The summary is basically the custom message supplied by the assertion invocation, and the details display the expected value, actual value, and the assertion operator (from [lib/assert.js](https://github.com/alexyoung/turing-test.js/blob/8dd48b5b6da1e26ab7f194bded8ad4c1f9abae6a/lib/assert.js)):

{% highlight javascript %}
assert.AssertionError.prototype.summary = function() {
  return this.name + (this.message ? ': ' + this.message : '');
};

assert.AssertionError.prototype.details = function() {
  return 'In "' + this.operator + '":\n\tExpected: ' + this.expected + '\n\tFound: ' + this.actual;
};

assert.AssertionError.prototype.toString = function() {
  return this.summary() + '\n' + this.details();
};
{% endhighlight %}

I've based this approach on the test frameworks we looked at in [part 36](http://dailyjs.com/2010/10/28/framework-part-36/).

### The Results

I've made a test fail on purpose here to illustrate the results. Console tests should look like this:

![](/images/posts/console_tt.png)

And a browser is very similar:

![](/images/posts/firefox_tt.png)

This version of turing-test.js is in [commit 5a6cbf61](https://github.com/alexyoung/turing-test.js/commit/5a6cbf61cdad29e688d8c1288c65ce83867f1122).

### References

-   [CommonJS Unit Test 1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0)
