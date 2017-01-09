---
layout: post
title: "Let's Make a Framework: Animations"
author: Alex Young
categories: 
- frameworks
- tutorials
- animation
- lmaf
---

Welcome to part 15 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part starts looking at JavaScript animations.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### JavaScript Animation

JavaScript animation libraries are usually comprised of the following elements:

-   Methods to work with CSS properties and animate them
-   A queuing system, for scheduling animations, and animating each "frame"
-   CSS colour parsing
-   Helpers that make common web effects easy to use alongside events

### Animation Frameworks

One of the first popular libraries that addressed animation was [script.aculo.us](http://script.aculo.us/). The [effects.js](http://github.com/madrobby/scriptaculous/blob/master/src/effects.js) script offered many pre-baked effects that developers wanted to use on their sites, and transition effects like <code>Effect.Highlight</code>, <code>Effect.Appear</code> and <code>Effect.BlindDown</code> quickly became popular. These effects are built using some JavaScript logic to manipulate CSS properties.

The script.aculo.us API is based around instantiated objects:

{% highlight javascript %}
new Effect.EffectName(element, required parameters, [options]);
{% endhighlight %}

Most effects take a <code>duration</code>, <code>from</code> and <code>to</code> parameter:

{% highlight javascript %}
new Effect.Opacity('element', { 
  duration: 2.0,
  transition: Effect.Transitions.linear,
  from: 1.0, 
  to: 0.5
});
{% endhighlight %}

The <code>transition</code> option refers to a control function that determines the rate of change.

[MooTools](http://mootools.net/) has a similar API in its [Fx](http://mootools.net/docs/core/Fx/Fx) module:

{% highlight javascript %}
new Fx.Reveal($('element'), { duration: 500, mode: 'horizontal' });
{% endhighlight %}

Shortcuts can be added to the <code>Element</code> class as well:

{% highlight javascript %}
$('element').reveal({ duration: 500, mode: 'horizontal' });
{% endhighlight %}

jQuery provides another animation API with helpers and CSS property manipulation. jQuery UI and other plugins build or extend this functionality. The main method is [animate()](http://api.jquery.com/animate/) which accepts properties to animate, duration, easing function and a termination callback. Most tasks can be completed with the helpers.

The nice thing about jQuery's animation API is it's very easy to create sequences of animations -- just chain a list of calls:

{% highlight javascript %}
$('#element').slideUp(300).delay(800).fadeIn(400);
{% endhighlight %}

jQuery builds a queue to create these animation sequences. The [queue](http://api.jquery.com/queue/) documentation has an example that demonstrates how a queue is built up based on the animation call order and durations.

The Glow framework builds a set of helper methods for common animation tasks from a core animation class called [glow.anim.Animation](http://www.bbc.co.uk/glow/docs/1.7/api/glow.anim.animation.shtml). This can be combined with [glow.anim.Timeline](http://www.bbc.co.uk/glow/docs/1.7/furtherinfo/animtimeline/) to create complex animations.

### Queues and Events

Animation frameworks usually use <code>setInterval</code> and <code>clearInterval</code> interval to sequence events. This can be combined with custom events. Due to JavaScript's single-threaded environment, <code>setInterval</code> and events are the best way of managing the asynchronous nature of animations.

### Animation Basics

As we've seen, animation frameworks build on top of CSS property manipulation. At its most basic, animation looks like this:

{% highlight javascript %}
// Box CSS: #box { background-color: red; width: 20px; height: 20px; position: absolute; left: 0; top: 10 }

function animate() {
  var box = document.getElementById('box'),
      duration = 1000,
      start = (new Date).valueOf(),
      finish = start + duration,
      interval;

  interval = setInterval(function() {
    var time = (new Date).valueOf(), frame = time > finish ? 1 : (time - start) / duration;

    // The thing being animated
    box.style.left = frame * 100 + 'px';

    if (time > finish) {
      clearInterval(interval);
    }
  }, 10);
}
{% endhighlight %}

I based this code on [emile.js](http://github.com/madrobby/emile/blob/master/emile.js). It uses <code>setInterval</code> which calls a function every few milliseconds. Time calculations (based on the number of milliseconds returned by <code>new Date</code>) are used to set up the animation and stop it. Once the end of the animation has been reached, <code>clearInterval</code> is called.

This is essentially what jQuery's animation library does with its queues.

### Next Week

Next week I'll expand on this code to start the basis of an animation library.

### References

-   [emile.js](http://github.com/madrobby/emile/blob/master/emile.js)
-   [Getting Started with Emile](http://dailyjs.com/2010/02/17/getting-started-with-emile/)
-   [Scriptaculous Wiki](http://wiki.github.com/madrobby/scriptaculous/core-effects)
-   [Scriptaculous effects.js](http://github.com/madrobby/scriptaculous/blob/master/src/effects.js)
-   [Glow's anim.js](http://github.com/glow/glow1/blob/master/src/anim/anim.js)
-   [Glow's animtimeline example](http://www.bbc.co.uk/glow/docs/1.7/furtherinfo/animtimeline/)
-   [MooTools' Fx](http://mootools.net/docs/more/Fx/Fx.Elements)
-   [jQuery's effects.js](http://github.com/jquery/jquery/blob/master/src/effects.js)
