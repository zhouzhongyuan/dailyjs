---
layout: post
title: "Let's Make a Framework: Animations Part 4"
author: Alex Young
categories: 
- frameworks
- tutorials
- animation
- lmaf
---

Welcome to part 18 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at JavaScript animations.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Recap

If you're wondering why I'm taking my time over animation, it's because it's fun! Although we rely on frameworks for core features like CSS selector parsing and searching, event handling, and cross-browser fixes, they can be dry topics at times. Animations still demonstrate core browser-based JavaScript concepts, but are a little bit more enjoyable to play with.

Over the last 3 weeks we've covered:

-   The basics of time-based animation functions using <code>setInterval</code> and <code>clearInterval</code>
-   Animating CSS properties
-   Easing functions

DOM-based animations are mostly based on manipulating CSS properties. However, everything else can be applied to other languages and concepts -- animations generally require easing functions and time-based sequencing.

### Animation Helpers

Our API currently looks like this:

{% highlight javascript %}
turing.anim.animate(element, 1000, { 'marginLeft': '8em', 'marginTop': '100px' }, { easing: 'bounce' });
{% endhighlight %}

This is quite unnatural though. It's weird to have to think in terms of CSS properties, and most web sites and apps just need a handful of common effects:

-   Fade -- fade an element by changing its opacity
-   Highlight -- rapidly change an element's background colour to draw attention to it
-   Movement -- move an element

#### Fade

Ideally we want to be able to specify a core fade function to build fade in and out:

{% highlight javascript %}
turing.anim.fade(element, duration, { 'from': '8em', 'to': '100px', 'easing': easing });
{% endhighlight %}

Easing should be optional.

Using the existing animation API, the core of the <code>fade</code> function should look like this:

{% highlight javascript %}
element.style.opacity = options.from;
anim.animate(element, duration, { 'opacity': options.to }, { 'easing': options.easing })
{% endhighlight %}

Now <code>fadeIn</code> and <code>fadeOut</code> can be built with sensible defaults:

{% highlight javascript %}
anim.fadeIn = function(element, duration, options) {
  options = options || {};
  options.from = options.from || 0.0;
  options.to = options.to || 1.0;
  return anim.fade(element, duration, options);
};
{% endhighlight %}

<code>fadeOut</code> needs to use its own easing function to change the <code>from</code> and <code>to</code> values negatively:

{% highlight javascript %}
anim.fadeOut = function(element, duration, options) {
  var from;
  options = options || {};
  options.from = options.from || 1.0;
  options.to = options.to || 0.0;

  // Swap from and to
  from = options.from;
  options.from = options.to;
  options.to = from;

  // This easing function reverses the position value and adds from
  options.easing = function(p) { return (1.0 - p) + options.from; };

  return anim.fade(element, duration, options, { 'easing': options.easing });
};
{% endhighlight %}

This might seem a bit confusing. Recall the <code>animate</code> function's time value scaling equation:

{% highlight javascript %}
element.style[property] = (easingFunction(position) * properties[property].number) + properties[property].units;
{% endhighlight %}

This scales values using elapsed time to the number passed in the DOM style properties. In this case it's the value for <options>from</code>.

Great! But this won't work in IE, and Turing doesn't have any browser detection features.

#### Opacity and IE

IE uses filters to set opacity, like this: <code>filter: alpha(opacity=0-100)</code>. Let's slightly change the way styles are set to use a function that is aware of browser differences:

{% highlight javascript %}
// opacityType = (typeof document.body.style.opacity !== 'undefined') ? 'opacity' : 'filter'

function setCSSProperty(element, property, value) {
  if (property == 'opacity' && opacityType == 'filter') {
    element.style[opacityType] = 'alpha(opacity=' + Math.round(value * 100) + ')';
    return element;
  }
  element.style[property] = value;
  return element;
}
{% endhighlight %}

Another slight glitch in IE's opacity handling is elements need *layout*. I've changed the part of the code that parses CSS values to set <code>zoom</code> to fix this:

{% highlight javascript %}
for (var property in properties) {
  if (properties.hasOwnProperty(property)) {
    properties[property] = parseCSSValue(properties[property]);
    if (property == 'opacity' && opacityType == 'filter') {
      element.style.zoom = 1;
    }
  }
}
{% endhighlight %}

You can read more about layout in [On having layout](http://www.satzansatz.de/cssd/onhavinglayout.html) by Ingo Chao.

### Conclusion

This simple example ended up being a little bit more complicated than I expected. I'll continue next week with the highlight effect, and try to wrap up these helpers so we can move on to chaining animations.

The reason for the break is our framework doesn't handle colours yet -- it's going to need colour animation support in <code>parseCSSValue</code>. This will take a little bit of explaining.

Remember to check out the [full source](http://github.com/alexyoung/turing.js/) with git to try demos and snippets.

### References

[On having layout](http://www.satzansatz.de/cssd/onhavinglayout.html)
