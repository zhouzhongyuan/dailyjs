---
layout: post
title: "Let's Make a Framework: Animations Part 3"
author: Alex Young
categories: 
- frameworks
- tutorials
- animation
- lmaf
---

Welcome to part 17 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at JavaScript animations.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Easing

Easing is an important animation technique that most people will never realise exists. It's a surprisingly important technique for creating natural-looking animations, even for simple web animations. Most animations use easing functions, and animation frameworks usually use one by default. Animations can seem strangely abrupt without non-linear easing.

The [script.aculo.us wiki](http://wiki.github.com/madrobby/scriptaculous/effect-transitions) has a useful interactive example of many common easing functions. Another great resource is in the [documentation for Tweener](http://hosted.zeh.com.br/tweener/docs/en-us/misc/transitions.html).

From [wikipedia](http://en.wikipedia.org/wiki/Inbetweening):

> "Ease-in" and "ease-out" in digital animation typically refer to a mechanism for defining the 'physics' of the transition between two animation states, eg. the linearity of a tween.

### Adding Easing Function Support

Last week the <code>turing.anim.animate</code> method accepted parameters but didn't do anything with them. Let's change it to work like this:

{% highlight javascript %}
turing.anim.animate(box, 1000, { 'marginLeft': '8em', 'marginTop': '100px' }, { easing: 'bounce' });
{% endhighlight %}

The default easing will be linear, and I'll include a few easing functions so other people can reference them and create their own:

{% highlight javascript %}
var easing = {};
easing.linear = function(position) {
  return position;
};
{% endhighlight %}

Then the <code>animate</code> method just needs to ensure the user has specified either a string or a function:

{% highlight javascript %}
if (options.hasOwnProperty('easing')) {
  if (typeof options.easing === 'string') {
    easingFunction = easing[options.easing];
  } else {
    easingFunction = options.easing;
  }
}
{% endhighlight %}

### Writing Easing Functions

The classic easing function is based on <code>cos</code>:

{% highlight javascript %}
easing.sine = function(position) {
  return (-Math.cos(position * Math.PI) / 2) + 0.5;
};
{% endhighlight %}

Mathematical functions can be exploited to create interesting effects. Functions like <code>sin</code> and <code>cos</code> are oscillators, and their periodic nature is useful for many types of animation. The visual impact of this easing function is more like acceleration than linear movement.

If you'd like to create your own functions based on trigonometry, it's worth reading the wikipedia page on [sine waves](http://en.wikipedia.org/wiki/Sinusoidal) to get a feel for the basics first.

![](/images/posts/grapher.png)

I explored the easing functions I've used here with Grapher, which comes with some versions of Mac OS. If you're familiar with the mathematical notation for basic trigonometric equations it can be a useful tool for finding out why a particular function makes everything move backwards suddenly.

More "programmatic" equations can be created using <code>if</code> statements to change the position multiplier at certain points:

{% highlight javascript %}
easing.bounce = function(position) {
  if (position < (1 / 2.75)) {
    return 7.6 * position * position;
  } else if (position < (2 /2.75)) {
    return 7.6 * (position -= (1.5 / 2.75)) * position + 0.74;
  } else if (position < (2.5 / 2.75)) {
    return 7.6 * (position -= (2.25 / 2.75)) * position + 0.91;
  } else {
    return 7.6 * (position -= (2.625 / 2.75)) * position + 0.98;
  }
};
{% endhighlight %}

I based these equations on the [Tweener](http://code.google.com/p/tweener/) library and script.aculo.us.

### Next Week

Last week I said I'd cover animation helpers and how to build a chained animation API. I forgot to factor the topic of easing into my plans, so I'll try to get to these topics next week.

### References

-   [Wikipedia on Inbetweening](http://en.wikipedia.org/wiki/Inbetweening):
-   [script.aculo.us wiki on transitions](http://wiki.github.com/madrobby/scriptaculous/effect-transitions)
-   [Tweener's interactive easing library](http://hosted.zeh.com.br/tweener/docs/en-us/misc/transitions.html)
-   [Tweener](http://code.google.com/p/tweener/)
