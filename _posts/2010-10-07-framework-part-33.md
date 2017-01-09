---
layout: post
title: "Let's Make a Framework: Feature Detection"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
---

Welcome to part 33 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

[Last week](http://dailyjs.com/2010/09/30/framework-part-32/) I wrote about packaging the project using Node scripts. This week I'm going to talk about feature detection.

### querySelectorAll

The selector engine we built for the core of <code>turing.dom</code> was based on the way Firefox interprets CSS selectors. I liked the approach for the context of these tutorials, because it's a very pragmatic approach that's easy to follow.

Browsers have been shipping with [querySelectorAll](http://www.w3.org/TR/selectors-api2/#queryselectorall) for a while, which reduces the amount of work required to implement DOM lookups.

That means <code>turing.dom.get</code> could be rewritten:

{% highlight javascript %}
// Original code minus all the
// magic in the Searcher class and tokenizer
dom.get = function(selector, root) {
  var tokens = dom.tokenize(selector).tokens,
      searcher = new Searcher(root, tokens);
  return searcher.parse();
};

// New selector API
dom.get = function(selector) {
  return document.querySelectorAll(selector);
};
{% endhighlight %}

But not all browsers support this yet, so let's check if it's available:

{% highlight javascript %}
dom.get = function(selector) {
  var root = typeof arguments[1] === 'undefined' ? document : arguments[1];
  if ('querySelectorAll' in document) {
    return root.querySelectorAll(selector);
  } else {
    return get(selector, root);
  }
};
{% endhighlight %}

### In the Wild

jQuery will check for <code>querySelectorAll</code>, but it'll only use it under certain conditions. This is from jQuery 1.4.2:

{% highlight javascript %}
if (document.querySelectorAll ) {
  (function(){
    var oldSizzle = Sizzle, div = document.createElement("div");
    div.innerHTML = "<p class='TEST'></p>";

    // Safari can't handle uppercase or unicode characters when
    // in quirks mode.
    if ( div.querySelectorAll && div.querySelectorAll(".TEST").length === 0 ) {
      return;
    }

    Sizzle = function(query, context, extra, seed){
      context = context || document;

      // Only use querySelectorAll on non-XML documents
      // (ID selectors don't work in non-HTML documents)
      if ( !seed && context.nodeType === 9 && !isXML(context) ) {
        try {
          return makeArray( context.querySelectorAll(query), extra );
        } catch(e){}
      }

      return oldSizzle(query, context, extra, seed);
    };

    for ( var prop in oldSizzle ) {
      Sizzle[ prop ] = oldSizzle[ prop ];
    }

    div = null; // release memory in IE
  })();
}
{% endhighlight %}

The use of an element for capability detection is common in frameworks -- sometimes it's the only reliable way of detecting a browser's behaviour.

It's a little bit different in Dojo 1.5, but the same Safari issue is mentioned:

{% highlight javascript %}
  // some versions of Safari provided QSA, but it was buggy and crash-prone.
  // We need te detect the right "internal" webkit version to make this work.
  var wk = "WebKit/";
  var is525 = (
    d.isWebKit && 
    (nua.indexOf(wk) > 0) && 
    (parseFloat(nua.split(wk)[1]) > 528)
  );

  // IE QSA queries may incorrectly include comment nodes, so we throw the
  // zipping function into "remove" comments mode instead of the normal "skip
  // it" which every other QSA-clued browser enjoys
  var noZip = d.isIE ? "commentStrip" : "nozip";

  var qsa = "querySelectorAll";
  var qsaAvail = (
    !!getDoc()[qsa] && 
    // see #5832
    (!d.isSafari || (d.isSafari > 3.1) || is525 )
  );

  //Don't bother with n+3 type of matches, IE complains if we modify those.
  var infixSpaceRe = /n\+\d|([^ ])?([>~+])([^ =])?/g;
  var infixSpaceFunc = function(match, pre, ch, post) {
    return ch ? (pre ? pre + " " : "") + ch + (post ? " " + post : "") : /*n+3*/ match;
  };

  var getQueryFunc = function(query, forceDOM){
    //Normalize query. The CSS3 selectors spec allows for omitting spaces around
    //infix operators, >, ~ and +
    //Do the work here since detection for spaces is used as a simple "not use QSA"
    //test below.
    query = query.replace(infixSpaceRe, infixSpaceFunc);

    if(qsaAvail){
      // if we've got a cached variant and we think we can do it, run it!
      var qsaCached = _queryFuncCacheQSA[query];
      if(qsaCached && !forceDOM){ return qsaCached; }
    }

    // Snip
{% endhighlight %}

Here the browser and version are derived from the user agent string.

### has.js

I recently wrote about [has.js](http://github.com/phiggins42/has.js) which is a clever little project that contains a library of feature detection tests.

> Browser sniffing and feature inference are flawed techniques for detecting browser support in client side JavaScript. The goal of has.js is to provide a collection of self-contained tests and unified framework around using pure feature detection for whatever library consumes it.

Using has.js as inspiration, we should be able to rewrite the previous code like this:

{% highlight javascript %}
dom.get = function(selector) {
  var root = typeof arguments[1] === 'undefined' ? document : arguments[1];
  return turing.detect('querySelectorAll') ?
    root.querySelectorAll(selector) : get(selector, root);
};
{% endhighlight %}

### Feature Detection Implementation

Making a library of feature tests is fairly easy with a plain <code>Object</code>. Tests can be referred to by name and easily looked up at runtime. Also, a cache can be used to store the results of the tests.

The core of this functionality is something inherent to JavaScript programming rather than just browser-related, so I put this in [turing.core.js](http://github.com/alexyoung/turing.js/blob/master/turing.core.js):

{% highlight javascript %}
var testCache = {},
    detectionTests = {};

turing.addDetectionTest = function(name, fn) {
  if (!detectionTests[name])
    detectionTests[name] = fn;
};

turing.detect = function(testName) {
  if (typeof testCache[testCache] === 'undefined') {
    testCache[testName] = detectionTests[testName]();
  }
  return testCache[testName];
};
{% endhighlight %}

The results are cached because they should only be run once. This type of capability detection is intended to be used against the environment, rather than features that might load dynamically in runtime, so I think it's safe to run the tests once.

Then the jQuery-inspired <code>querySelectorAll</code> test can be added in [turing.dom.js](http://github.com/alexyoung/turing.js/blob/master/turing.dom.js):

{% highlight javascript %}
turing.addDetectionTest('querySelectorAll', function() {
  var div = document.createElement('div');
  div.innerHTML = '<p class="TEST"></p>';

  // Some versions of Safari can't handle uppercase in quirks mode
  if (div.querySelectorAll) {
    if (div.querySelectorAll('.TEST').length === 0) return false;
    return true;
  }

  // Helps IE release memory associated with the div
  div = null;
  return false;
});
{% endhighlight %}

### Conclusion

Looking through popular JavaScript frameworks made me realise that most of them still use the user agent string to determine what capabilities are available. This might be fine most of the time, but I don't consider it best practice. The way jQuery tests capabilities using techniques like the dummy DOM element creation seems like a lot of work, but it relies on browser behaviour rather than the user agent string.

I originally wanted to write this article about <code>querySelectorAll</code> and its family of related methods, but adding generic capability detection support became an interesting little rabbit hole. I hope this illustrates how simple tasks can require extra effort to make code highly reusable.
