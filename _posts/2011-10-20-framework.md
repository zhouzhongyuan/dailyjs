---
layout: post
title: "Asynchronous Resource Loading Part 4"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- network
---

<div class="intro">
*Let's Make a Framework* is an ongoing series about building a JavaScript framework from the ground up.

These articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js). Documentation is available at [turingjs.com](http://turingjs.com/).

</div>
Previous parts:

-   [Part 1: Introduction, library review](http://dailyjs.com/2011/09/22/framework-81/)
-   [Part 2: Script Insertion](http://dailyjs.com/2011/09/29/framework-82/)
-   [Part 3: HTML5](http://dailyjs.com/2011/10/13/framework/)

### Loading with <code>XMLHttpRequest</code>

In last week's tutorial I hinted at a technique script loading libraries use to preload local scripts using <code>XMLHttpRequest</code>. By requesting scripts this way, the contents can be placed into a queue then executed through a <code>script</code> element's <code>.text</code> property when required. This allows libraries like [LABjs](http://labjs.com/) to schedule execution based on the user's requirements.

Now we're starting to go beyond script insertion and into the realms of preloading and scheduling. With that in mind, I've redesigned the code for this module to be easier to test.

### Designing for Testability

Production-ready script loaders will dynamically decide on the best strategy for preloading a given script. That makes them easy to use, but potentially makes them hard to test. I want to write tests like this:

{% highlight javascript %}
$t.require('/load-me.js?test0=0', { transport: 'scriptInsertion' }, function() {
  assert.equal(window.test0, 0);
});

$t.require('/load-me.js?test3=3', { transport: 'XMLHttpRequest' }, function() {
  assert.equal(window.test3, 3);
});
{% endhighlight %}

Given a server-side test harness -- which I've already written in [test/functional/ajax.js](https://github.com/alexyoung/turing.js/tree/master/test/functional) -- we should be able to specify which method is used to load a script, and get the expected results.

The <code>transport</code> option in the previous example allows us to control which loading strategy is used. The <code>scriptInsertion</code> transport is what we created in the previous tutorials. The new one is <code>XMLHttpRequest</code>, which gives us more potential for preloading and scheduling scripts.

### Implementation

To build this, I've broken the problem up into several functions:

-   <code>isSameOrigin</code> determines if a given <code>src</code> is local or remote, so we don't get same origin errors in the browser
-   <code>createScript</code> creates a new script tag and applies an <code>Object</code> of options
-   <code>insertScript</code> inserts the script tag into the document
-   <code>requireWithScriptInsertion</code> loads scripts using insertion
-   <code>requireWithXMLHttpRequest</code> loads scripts using <code>XMLHttpRequest</code>

A lot of this code was originally in <code>require</code>, but when I realised I was doing the same thing in the <code>XMLHttpRequest</code> loader I decided to break it up.

This method loads the script using our built-in <code>XMLHttpRequest</code> support:

{% highlight javascript %}
  /**
   * Loads scripts using XMLHttpRequest.
   *
   * @param {String} The script path
   * @param {Object} A configuration object
   * @param {Function} A callback
   */
  function requireWithXMLHttpRequest(scriptSrc, options, fn) {
    if (!isSameOrigin(scriptSrc)) {
      throw('Scripts loaded with XMLHttpRequest must be from the same origin');
    }

    if (!turing.get) {
      throw('Loading scripts with XMLHttpRequest requires turing.net to be loaded');
    }

    turing
      .get(scriptSrc)
      .end(function(res) {
        // Here's where the magic happens.  This callback is what will get scheduled in future versions.
        options.text = res.responseText;
        
        var script = createScript(options);
        insertScript(script);
        appendTo.removeChild(script);
        fn();
      });
  }
{% endhighlight %}

Which means the public method, <code>require</code>, now has to decide which transport to use:

{% highlight javascript %}
/**
 * Non-blocking script loading.
 *
 * @param {String} The script path
 * @param {Object} A configuration object.  Options: {Boolean} `defer`, {Boolean} `async`
 * @param {Function} A callback
 */
turing.require = function(scriptSrc, options, fn) {
  options = options || {};
  fn = fn || function() {};

  setTimeout(function() {
    if ('item' in appendTo) {
      if (!appendTo[0]) {
        return setTimeout(arguments.callee, 25);
      }

      appendTo = appendTo[0];
    }

    switch (options.transport) {
      case 'XMLHttpRequest':
        return requireWithXMLHttpRequest(scriptSrc, options, fn);

      case 'scriptInsertion':
        return requireWithScriptInsertion(scriptSrc, options, fn);

      default:
        return requireWithScriptInsertion(scriptSrc, options, fn);
    }
  });
};
{% endhighlight %}

### Conclusion

I've tested this in IE 6, 7, Firefox, Chrome, and Safari. A future version of this module will have to decide which loading method to use by default, depending on the scheduling requirements (which we have yet to start work on).

There are actually more script loading techniques to look at before I can get to scheduling. As I've said before, if you're interested in this area take a look at [RequireJS](http://requirejs.org/) and [LABjs](http://labjs.com/) to jump ahead.

This week's code can be found in [commit 649f882](https://github.com/alexyoung/turing.js/commit/649f8822b120ebd86de3c2d57ea7122396e8e2ac).
