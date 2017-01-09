---
layout: post
title: "Asynchronous Resource Loading Part 3"
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

### HTML5 Asynchronous Loading Support

In the last part I created this simple API for loading scripts asynchronously:

{% highlight javascript %}
$t.require('/load-me.js', function() {
  // Loaded
});
{% endhighlight %}

The next step is to look at how to handle *execution order control*. The <code>script</code> element gets two new attributes in HTML5: <code>async</code> and <code>defer</code>. Technically <code>defer</code> was present in HTML4:

> When set, this boolean attribute provides a hint to the user agent that the script is not going to generate any document content (e.g., no "document.write" in javascript) and thus, the user agent can continue parsing and rendering.

From: [HTML4 Scripts](http://www.w3.org/TR/html4/interact/scripts.html#adef-defer)

That means there are the following possible states:

-   <code>async</code>: Execute the script asynchronously as soon as it is available
-   <code>defer</code>: Execute the script when the page has finished parsing, thereby allowing other scripts to download and execute
-   <code>async</code> and <code>defer</code>: Use <code>async</code> if available, else legacy browsers will fall back to <code>defer</code> (from: [Using HTML 5 for performance improvements](http://code.google.com/speed/articles/html5-performance.html))
-   If neither are present, the script is fetched and executed immediately before the page has finished parsing

Out of interest, I tried setting these attributes on the generated <code>script</code> elements, and it didn't cause IE6 to break. I suspect this should really use feature detection to be safe.

In order to support the baseline HTML5 API, I added some options to the <code>require</code> method:

{% highlight javascript %}
$t.require('/load-me.js', { async: true, defer: true }, function() {
  assert.equal(loadMeDone, 1);
});
{% endhighlight %}

And this just sets the properties as you'd expect:

{% highlight javascript %}
function require(scriptSrc, options, fn) {
  var script = document.createElement('script');
  script.type = 'text/javascript';
  script.src = scriptSrc;

  if (options.async) {
    script.async = options.async;
  }

  if (options.defer) {
    script.defer = options.defer;
  }
{% endhighlight %}

### Execution Order and Preloading

Now <code>require</code> works as asynchronously as possible, have we solved the problem of asynchronous resource loading? Not by a long shot. Although execution order could now be controlled through callbacks, this wouldn't do what we really want to do:

{% highlight javascript %}
// This is wrong:
$t.require('/library-a.js', function() {
  $t.require('/library-b.js', function() {
    $t.require('/app-code.js', function() {
      // Done!
    });
  });
});
{% endhighlight %}

Why is this wrong? It fails to distinguish between *loading* and *executing* scripts. Libraries like LABjs utilise multiple strategies for managing script execution order. Going back to [Loading Scripts Without Blocking](http://www.stevesouders.com/blog/2009/04/27/loading-scripts-without-blocking/) by Steve Souders, we can find six techniques for downloading scripts without blocking. Steve even includes a table to compare the properties of each of these approaches, and it shows that only three techniques can be used to control execution order.

There are more techniques available -- many libraries use <code>object</code> or <code>img</code> to preload scripts. A <code>script</code> element is used once the script has loaded, effectively rerequesting the script when it's required, and therefore hitting the cache. This preloading approach is fairly widely used, but has to account for a lot of browser quirks. In some cases it can cause scripts to be loaded twice.

### XMLHttpRequest Loading

LABjs has a whole load of code for managing loading scripts with XMLHttpRequest. Why? Well, it makes preloading possible and avoids loading scripts twice. However, it can only be used to load local scripts due to the same origin policy.

### Dynamic Script Execution Order

In [Dynamic Script Execution Order](http://wiki.whatwg.org/wiki/Dynamic_Script_Execution_Order), an extension to the behaviour of the <code>async</code> attribute is considered. This proposal is known as <code>async=false</code>:

> If a parser-inserted script element has the \`async\` attribute present, but its value is exactly "false" (or any capitalization thereof), the script element should behave EXACTLY as if no \`async\` attribute were present.

This would allow our script loader to set <code>scriptTag.async = false</code> when execution order is important, else <code>scriptTag.async = true</code> could be used to load it and run it whenever possible.

### Conclusion

Despite script loading libraries like LABjs and [RequireJS](http://requirejs.org/) existing for a few years, the problem still hasn't been completely solved, and we still need to support legacy browsers. Simply loading non-blocking JavaScript by inserting script tags is possible, but controlling execution order requires an inordinate amount of effort.

If you've ever wondered why LABjs is around 500 lines of code, then I hope you can now appreciate the lengths the author has gone to!

The HTML5 support I added can be found in [commit c007251](https://github.com/alexyoung/turing.js/commit/c0072517994036d421654fd14fe549f4b61ead76).

### References

-   [HTML4 Script Tags, defer](http://www.w3.org/TR/html4/interact/scripts.html#adef-defer)
-   [Using HTML 5 for performance improvements](http://code.google.com/speed/articles/html5-performance.html)
-   [Comments on ControlJS part 1: async loading](http://www.stevesouders.com/blog/2010/12/15/controljs-part-1/)
-   [Dynamic Script Execution Order](http://wiki.whatwg.org/wiki/Dynamic_Script_Execution_Order)
