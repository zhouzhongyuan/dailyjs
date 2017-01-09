---
layout: post
title: "Let's Make a Framework: DOM and Events Tests"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- testing
- commonjs
---

Welcome to part 42 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

In this part I'll discuss using our new CommonJS-based testing framework, [turing-test.js](https://github.com/alexyoung/turing-test.js), to test our framework.

### Animation Tests

The old animation tests were fairly basic. Once I'd rewritten them I noticed something interesting:

{% highlight javascript %}
exports.testAnimations = {
  'test hex colour converts to RGB': function() {
    assert.equal('rgb(255, 0, 255)', turing.anim.parseColour('#ff00ff').toString());
  },

  'test RGB colours are left alone': function() {
    assert.equal('rgb(255, 255, 255)', turing.anim.parseColour('rgb(255, 255, 255)').toString());
  },

  'test chained animations': function() {
    var box = document.getElementById('box');
    turing.anim.chain(box)
      .highlight()
      .pause(250)
      .move(100, { x: '100px', y: '100px', easing: 'ease-in-out' })
      .animate(250, { width: '1000px' })
      .fadeOut(250)
      .pause(250)
      .fadeIn(250)
      .animate(250, { width: '20px' });

    // New:
    setTimeout(function() { assert.equal(box.style.top, '100px'); }, 350);
    setTimeout(function() { assert.equal(box.style.width, '20px'); }, 2000);
  }
};
{% endhighlight %}

The chained calls at the end didn't have any tests before, so I tried adding some with <code>setTimeout</code> to test a few steps that would be executed as part of the chain. Another way to test time-based events like this would be to put the asserts inside any callbacks the API supports.

With the current version of the test library, exceptions will be logged for the failed tests rather than printing them to the test reports. There'd need to be a mechanism for waiting for asserts in <code>setTimeout</code> to handle this correctly (maybe an async test helper).

### DOM Tests

The DOM tests can be pretty much translated directly from the original tests:

{% highlight javascript %}
exports.testDOM = {
  'test tokenization': function() {
    assert.equal('class', turing.dom.tokenize('.link').finders(), 'return class for .link');
    assert.equal('name and class', turing.dom.tokenize('a.link').finders(), 'return class and name for a.link');
  },

  'test selector finders': function() {
    assert.equal('dom-test', turing.dom.get('#dom-test')[0].id, 'find with id');
    assert.equal('dom-test', turing.dom.get('div#dom-test')[0].id, 'find with id and name');
    assert.equal('Example Link', turing.dom.get('a')[0].innerHTML, 'find with tag name');

// etc.
{% endhighlight %}

It's a good idea to have a message for each assertion to make it easier to track down failing tests.

### Events Tests

The events tests look a little bit different because my other testing library will run functions when they're passed:

{% highlight javascript %}
given('a delegate handler', function() {
  var clicks = 0;
  turing.events.delegate(document, '#events-test a', 'click', function(e) {
    clicks++;
  });

  should('run the handler when the right selector is matched', function() {
    turing.events.fire(turing.dom.get('#events-test a')[0], 'click');
    return clicks;
  }).equals(1);

  should('only run when expected', function() {
    turing.events.fire(turing.dom.get('p')[0], 'click');
    return clicks;
  }).equals(1);
});
{% endhighlight %}

This is the old test which binds a counter, <code>clicks</code>, then triggers events with handlers that change <code>clicks</code>. Because <code>assert.equal</code> won't run functions, it needs to be rewritten like this:

{% highlight javascript %}
exports.testEvents = {
  'test delegate handlers': function() {
    var clicks = 0;
    turing.events.delegate(document, '#events-test a', 'click', function(e) {
      clicks++;
    });

    turing.events.fire(turing.dom.get('#events-test a')[0], 'click');
    assert.equal(1, clicks, 'run the handler when the right selector is matched');

    turing.events.fire(turing.dom.get('p')[0], 'click');
    assert.equal(1, clicks, 'run the handler when the right selector is matched');
  }
};
{% endhighlight %}

Being acutely aware of closure binding is the key to testing events. I've also ported the other event tests and they run fine with the assertion module.

### Conclusion

These browser-based tests were straightforward to port from the BDD-style library, [Riot.js](https://github.com/alexyoung/riotjs/), to CommonJS. So far it's only really been a cosmetic, syntax shift.

I feel like these CommonJS-based tests read well, but I've worked with unit testing libraries more intensely than more modern BDD libraries. There's no reason why BDD libraries can't be built on CommonJS assertions though, which is what many open source developers have been working on.

This code is available in [commit c3175f](https://github.com/alexyoung/turing.js/tree/c3175fcb229ca60abeac407bdd7f708fac00b772).
