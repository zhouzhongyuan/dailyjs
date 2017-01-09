---
layout: post
title: "Let's Make a Framework: Testing"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- testing
---

Welcome to part 36 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Last week I added more shortcuts to the chained API. This makes the framework feel more immediate and efficient, in a jQuery-inspired fashion.

When I started this tutorial series I used to spend one week discussing current framework implementations, then another week or more creating our own implementation. I'm going back to that format, because I get a lot of feedback where people indicate they're uncomfortable how much they rely on frameworks like jQuery without understanding how they work.

This week I'm going to look at testing in JavaScript frameworks. I've tried make this series test-driven to inspire good habits, but I keep hitting problems with the more gnarly browser-based testing.

With that in mind, let's take a look at how jQuery, Prototype, and MooTools run their tests.

### On CommonJS Assertions

So far we've been using my own test framework (called RiotJS) because the syntax is extremely light, which means it fits nicely into an article. That's not exactly the best motivation for choosing a test framework, however.

In the JavaScript world we have [CommonJS Unit Testing 1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0). This defines a module called <code>assert</code> that provides several assertions. It's cropping up all over the place, in Ringo, Node, SproutCore, and many more environments. It also defines a <code>test</code> module for encapsulating and running tests.

The problem with these modules is they aren't quite enough for writing friendly unit tests with easy to read output. That means we need something else on top. A good example of this is [nodeunit](http://github.com/caolan/nodeunit) by Caolan McMahon, which is a great piece of work. However, while console tests are useful we're building browser-based code as well; ideally we want something that'll wrap around CommonJS <code>assert</code> and <code>test</code> and run in the browser with some readable HTML output.

### jQuery

As I've said before, I like to see jQuery plugins that include tests. It seems polite to ship something with tests. jQuery's tests are extremely thorough, so I'm not sure why plugin authors don't take inspiration from the source of their favourite framework and bother to write any.

Each major part of the framework has its own unit test file. Tests are written with [QUnit](http://docs.jquery.com/Qunit) and a few small jQuery-specific modifications in [testrunner.js](http://github.com/jquery/jquery/blob/master/test/data/testrunner.js). QUnit is also on [jquery/qunit at GitHub](http://github.com/jquery/qunit).

QUnit has a terse syntax and generates friendly HTML reports. A test looks like this:

{% highlight javascript %}
test('DailyJS Example', function() {
  var example = 'hello world';
  ok(true, 'This test will pass');
  equals('hello world', example, 'This is a message');
});
{% endhighlight %}

The tests look like this when they run:

![](/images/posts/jquery_tests.png)

They're extremely thorough, with 2907 tests.

### Prototype

If you download Prototype from GitHub you'll need to build the framework before running the tests:

{% highlight sh %}
git clone http://github.com/sstephenson/prototype
cd prototype
rake
rake test
{% endhighlight %}

It'll actually fetch missing libraries using git submodules, then start running the tests in a browser. The tests are written with [unittest.js](http://github.com/tobie/unittest_js) which is maintained by Tobie Langel. Tests look like this:

{% highlight javascript %}
new Test.Unit.Runner({
  testTruth: function() {
    this.assert(true);
  }
});
{% endhighlight %}

It's a simple API that doesn't use any scary JavaScript magic. I think this library is based on Thomas Fuchs' unittest.js, which I seem to remember appearing back in 2005.

### MooTools

MooTools packages the tests as a separate project called [mootools-core-specs](http://github.com/mootools/mootools-core-specs). It uses PHP for packaging and NodeJS for running tests. It seems ridiculously over-complicated next to jQuery's approach and it's harder to get it to run.

The tests are "specs" so they actually look a bit like our Riot tests. Here's an example:

{% highlight javascript %}
describe('Array.from', function(){
  it('should return an array for an Elements collection', function(){
    var div1 = document.createElement('div');
    var div2 = document.createElement('div');
    var div3 = document.createElement('div');

    div1.appendChild(div2);
    div1.appendChild(div3);

    var array = Array.from(div1.getElementsByTagName('*'));
    expect(Type.isArray(array)).toEqual(true);
  });
});
{% endhighlight %}

### Conclusion

These frameworks have very different approaches to testing, but they all take it seriously which is encouraging. MooTools has the most dependencies, Prototype depends on Ruby, and jQuery requires a basic build environment.

I like the fact I can build jQuery with <code>make</code> then open up the tests in my browser with minimal effort. Actually running tests in MooTools takes more work, and Prototype opens a browser tab for each test file. If we're going to refactor Turing's tests, I'd prefer something CommonJS compatible that can run in a browser and console environments.

Using CommonJS's <code>assert</code> module seems like a good idea because it might let us easily run tests in a variety of environments. What we need is a small CommonJS-based library that will also make the <code>assert</code> module produce readable reports in a browser.
