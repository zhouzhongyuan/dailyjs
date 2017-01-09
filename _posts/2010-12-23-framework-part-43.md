---
layout: post
title: "Let's Make a Framework: More Tests"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- testing
- commonjs
---

Welcome to part 43 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Over the last few weeks we've built a test framework based on the CommonJS assert module, a suitable test runner, and started converting Turing's tests to use it. The test framework is called [Turing Test](https://github.com/alexyoung/turing-test.js).

### JavaScript Testing

I get a lot of questions about JavaScript tests, so before continuing converting Turing's tests I'm going to just explain a little bit about the basics behind JavaScript testing.

If you're primarily a front-end developer, a lot of the innovative new work in JavaScript testing might appear confusing. The needs of server-side testing and client-side diverge, so it's perfectly acceptable to use a different testing framework for each part of your project.

According to our [2010 JavaScript survey](http://dailyjs.com/2010/12/13/javascript-survey-results/), [Qunit](http://docs.jquery.com/Qunit) is the most popular test framework. So you could use Qunit for your browser tests, and the server-side developers could use something like [nodeunit](https://github.com/caolan/nodeunit) or [jasmine](https://github.com/pivotal/jasmine).

There are different types of test libraries. The one we've been developing on this tutorial series is a [unit testing](http://en.wikipedia.org/wiki/Unit_testing) framework. The [assert module](http://wiki.commonjs.org/wiki/Unit_Testing/1.0) defined in Unit Testing/1.0 by CommonJS is an example of a large part of a unit testing framework.

Libraries like Jasmine are [Behavior Driven Development](http://en.wikipedia.org/wiki/Behavior_Driven_Development) (BDD) libraries. Some frameworks like [Cucumber](http://cukes.info/) take this to another extreme, where tests are written as executable plain-text.

There are more flavours of test libraries, and some become fashionable and widely used. Because tests can be useful documentation for another developer, if you're writing an open source project it might be a good idea to use what your community uses. If you're writing a jQuery plugin, why not use Qunit?

If you're working on your own project or in a small team, the choice of test libraries can seem bewildering. You could fall back on the CommonJS modules and treat that as the "standard" way to write tests, as it's likely that other people will be able to understand your tests. The point is to actually write tests in a way that makes you feel productive -- some people get bogged down with the abstraction of BDD libraries, other people find that style of testing can help communicate business logic to clients or managers.

### Test Conversion

I've converted the core tests to Turing Test (from [RiotJS](https://github.com/alexyoung/riotjs)).

The enumerable tests demonstrated where using <code>assert.equal</code> or <code>assert.deepEqual</code> are appropriate:

{% highlight javascript %}
exports.testEnumerable = {
  'test array iteration with each': function() {
    var a = [1, 2, 3, 4, 5],
        count = 0;
    turing.enumerable.each(a, function(n) { count += 1; });
    assert.equal(5, count, 'count should have been iterated');
  },

  'test array iteration with map': function() {
    var a = [1, 2, 3, 4, 5],
        b = turing.enumerable.map(a, function(n) { return n + 1; });
    assert.deepEqual([2, 3, 4, 5, 6], b, 'map should iterate');
  },
{% endhighlight %}

The second test here need <code>deepEqual</code> to check the arrays are the same. It's important to remember which type of equal to use when writing CommonJS assertions.

The [OO tests](https://github.com/alexyoung/turing.js/blob/master/test/oo_test.js) had to be reworked slightly to suit CommonJS module constraints in the browser.

### Running All Tests

I've made [run.html](https://github.com/alexyoung/turing.js/blob/master/test/run.html) run all of the tests in the browser. It was actually very easy to do, I just had to list all of the unit test files in script tags:

{% highlight javascript %}
<script src="events_test.js" type="text/javascript"></script>
<script src="alias_test.js" type="text/javascript"></script>
<script src="anim_test.js" type="text/javascript"></script>
<script src="dom_test.js" type="text/javascript"></script>
<script src="net_test.js" type="text/javascript"></script>
<script src="oo_test.js" type="text/javascript"></script>
{% endhighlight %}

### Future Updates

When I started writing the Turing Test tutorials I mentioned I'd like to be able to run all tests, in a similar way to Qunit. This kind of works now, but isn't presented particularly well. The test runner we've built doesn't summarise all of the tests, and the layout isn't quite clear enough.

Turing Test also needs to display benchmarks, and according to our survey 12% of people benchmarking code use unit tests as an indicator of performance.

I'm pleased with the results now everything has been converted to CommonJS tests. As usual, more next week!

### References

-   [2010 JavaScript survey](http://dailyjs.com/2010/12/13/javascript-survey-results/)
-   [Qunit](http://docs.jquery.com/Qunit)
-   [nodeunit](https://github.com/caolan/nodeunit)
-   [jasmine](https://github.com/pivotal/jasmine)
-   [Unit testing](http://en.wikipedia.org/wiki/Unit_testing)
-   [Behavior Driven Development](http://en.wikipedia.org/wiki/Behavior_Driven_Development)
-   [Unit Testing/1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0)
