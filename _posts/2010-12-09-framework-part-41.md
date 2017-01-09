---
layout: post
title: "Let's Make a Framework: Submodules and Browser CommonJS Modules"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- testing
- commonjs
---

Welcome to part 41 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

The last few parts have been concerned with building a test framework based on CommonJS:

-   [Part 36](http://dailyjs.com/2010/10/28/framework-part-36/) - An introduction to how JavaScript frameworks handle testing
-   [Part 37](http://dailyjs.com/2010/11/04/framework-part-37/) - A review of CommonJS test specifications, and examples of how to run assert module tests in a browser with a simple test runner
-   [Part 38](http://dailyjs.com/2010/11/11/framework-part-38/) and [commit 9a4c33e](https://github.com/alexyoung/turing-test.js/commit/9a4c33e79cbeb0bf2a719bc155590bdb14c7390b) - Assertions part 1
-   [Part 39](http://dailyjs.com/2010/11/18/framework-part-39/) and [commit 8dd48b5](https://github.com/alexyoung/turing-test.js/commit/8dd48b5b6da1e26ab7f194bded8ad4c1f9abae6a) - Assertions part 2
-   [Part 40](http://dailyjs.com/2010/11/25/framework-part-40/) and [commit 5a6cbf61](https://github.com/alexyoung/turing-test.js/commit/5a6cbf61cdad29e688d8c1288c65ce83867f1122) - Test runner

In this part I'll show you how to use external libraries through git submodules, and how to simulate CommonJS modules in a browser (in a limited fashion).

### Submodules

Writing good reusable code isn't just about well-written code, it also involves project management. I don't mean a pointy-haired boss telling you what to do, just practical techniques for managing projects and sharing code between them.

I've already hit on the issues around packaging JavaScript frameworks, and we've looked at libraries like jQuery and Prototype to see how they package their code. Now that [turing-test.js](https://github.com/alexyoung/turing-test.js) is in a usable state we need a way of distributing it with Turing, without having to manually update it every time we make changes.

One way Real Open Source Projects&trade; do this is using git submodules. From [Pro Git](http://progit.org/):

> It often happens that while working on one project, you need to use another project from within it. Perhaps it's a library that a third party developed or that you're developing separately and using in multiple parent projects.

### Adding Submodules

Adding a submodule is easy:

{% highlight sh %}
git submodule add git@github.com:alexyoung/turing-test.js.git turing-test
{% endhighlight %}

### Getting and Updating Submodules

Git can't do everything, so it's important to communicate the fact we're using submodules in our README. I've put a note in Turing's README with instructions on how to get the required submodules. Of course, users don't really need to worry about this, it's only for people who want to run tests.

If you check Turing out from Git, you'll need to do this:

{% highlight sh %}
git submodule init
git submodule update
{% endhighlight %}

If the submodule has been updated, you'll need to run <code>git submodule update</code> to get the latest version.

### Integrating Turing Test

Now we have a process for sharing Turing-Test with Turing, we need to actually use the library to test something. For now I've put the test library submodule in <code>test/turing-test/</code> because it makes the path handling easier between Node and the web-based file tests.

The goal of Turing Test was to make tests use CommonJS assertions and a CommonJS test runner, which could work in a browser and server-side JavaScript. We've got most of what we need so far, except for one thing:

{% highlight javascript %}
var module = require('library').property;
{% endhighlight %}

Supporting modules in browsers isn't easy because it's incredibly awkward to block execution until something has loaded. Turing Test addressed this initially by mocking out <code>require</code> and forcing the user to load scripts with script tags in a HTML stub.

This is a simple solution, but can we go even further?

### Giving Browsers CommonJS Module Support

I decided to see how close I could get a browser-based test to resemble a CommonJS test. This will run using Turing Test in both environments:

{% highlight javascript %}
require.paths.unshift('./turing-test/lib');

turing = require('../turing.core.js').turing;
var test = require('test'),
    assert = require('assert'),
    $t = require('../turing.alias.js');

exports.testAlias = {
  'test turing is present': function() {
    assert.ok(turing, 'turing.core should have loaded');
  },

  'test alias exists': function() {
    assert.ok($t, 'the $t alias should be available');
  }
};

test.run(exports);
{% endhighlight %}

This test, [alias\_test.js](https://github.com/alexyoung/turing.js/blob/a7be08a55d5825359d1a5b61f67fa0f205c0b8f9/test/alias_test.js) will work in both a browser and Node. How? Well, the browser has a HTML test harness file that performs some setup before the test is run:

{% highlight html %}
<html>
  <head>
    <title>Alias Test</title>

    <!-- Test Library -->
    <link rel="stylesheet" href="turing-test/stylesheets/screen.css"></link>
    <script src="turing-test/turing-test.js" type="text/javascript"></script>
  </head>
  <body>
    <ul id="results"></ul>
    <script src="alias_test.js" type="text/javascript"></script>
  </body>
</html>
{% endhighlight %}

The file <code>turing-test.js</code> referenced here is where some browser patching occurs to provide <code>require</code>. Now <code>require</code> in the browser loads scripts by inserting a script tag and watching for the script to finish loading. Certain parts of the library block with <code>setTimeout</code> until the scripts have finished loading.

{% highlight javascript %}
function addEvent(obj, type, fn)  {
  if (obj.attachEvent) {
    obj['e' + type + fn] = fn;
    obj[type + fn] = function(){obj['e' + type + fn](window.event);}
    obj.attachEvent('on' + type, obj[type + fn]);
  } else
    obj.addEventListener(type, fn, false);
}

var scriptTag = document.createElement('script'),
    head = document.getElementsByTagName('head');
this.loading();
addEvent(scriptTag, 'load', tt.doneLoading);
scriptTag.setAttribute('type', 'text/javascript');
scriptTag.setAttribute('src', script);
head[0].insertBefore(scriptTag, head.firstChild);
{% endhighlight %}

The first call to <code>require</code> causes Turing Test to install some browser-patching code. The final call to <code>test.run</code> at the end of the tests will actually hit a fake test runner that uses <code>setTimeout</code> to block until the scripts have loaded:

{% highlight javascript %}
fakeTest: {
  run: function(tests) {
    if (tt.isLoading) {
      setTimeout(function() { tt.fakeTest.run(tests); }, 10);
    } else {
      return tt.testRunner.run(tests);
    }
  }
}
{% endhighlight %}

As long as our tests haven't been run yet they won't hit the empty objects returned by our hacked version of <code>require</code>.

### Conclusion

At this point we've got a test framework that looks 100% CommonJS in the browser and console, with a very simple format for the HTML loading stubs. That doesn't mean tests that depend on the DOM will work, and module loading doesn't really work very well outside of this limited example.

In fact, the insanity I had to pull to get this to work was quite ridiculous, and took hours of work. Because it was a stint from 6pm until 11pm I haven't had chance to fully explore it yet, but hopefully it gets people thinking about 'write once, run anywhere' testing.

The code I checked in was [commit b16caa90a for turing-test.js](https://github.com/alexyoung/turing-test.js/tree/b16caa90adf1f4174a301036fdf1a63067f49793) and [commit a7be08a5 for turing.js](https://github.com/alexyoung/turing.js/tree/a7be08a55d5825359d1a5b61f67fa0f205c0b8f9).

### References

-   [Git Book - Submodules](http://book.git-scm.com/5_submodules.html)
-   [Pro Git - Submodules](http://progit.org/book/ch6-6.html)
