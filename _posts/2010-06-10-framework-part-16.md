---
layout: post
title: "Let's Make a Framework: Animations Part 2"
author: Alex Young
categories: 
- frameworks
- tutorials
- animation
- lmaf
---

Welcome to part 16 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at JavaScript animations.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Time-Based Animation

Last week I shared an simple example of JavaScript animation code:

{% highlight javascript %}
function animate() {
  var box = document.getElementById('box'),
      duration = 1000,
      start = (new Date).valueOf(),
      finish = start + duration,
      interval;

  interval = setInterval(function() {
    var time = (new Date).valueOf(), position = time > finish ? 1 : (time - start) / duration;
    box.style.left = position * 100 + 'px';
    if (time > finish) {
      clearInterval(interval);
    }
  }, 10);
}
{% endhighlight %}

This makes a div move across the screen. This animation runs every 10 milliseconds, and uses the current time to determine the animation's progress.

The reason this technique is often used for animations in JavaScript is timers compete for attention with other parts of the browser. JavaScript runs in a single thread, but execution is shared by other timers and events. Time is sliced up depending on what needs attention.

Technically <code>setInterval</code> could be called a set number of times, based on the desired animation duration. In reality the overall duration would change depending on what else is going on in the browser.

Using <code>setInterval</code> with <code>Date</code> isn't such a big problem though. Each call to <code>valueOf</code> is fairly minimal.

To understand more about how timers work in JavaScript, read through [How JavaScript Timers Work](http://ejohn.org/blog/how-javascript-timers-work/) by John Resig.

### Animating Properties

As seen last week, manipulating CSS properties is the core part of animation frameworks. Once this is done, Turing can make animation helpers available so common tasks are succinct.

The <code>animate()</code> example above uses the <code>left</code> style property to move the item by multiplying the current animation <code>position</code> by a value. The <code>position</code> value is between 0 and 1. This value is actually the end point of the animation. If we wanted to move something by saying "increase margin-left to 50", we could multiply <code>position</code> by 50 for each frame.

That means we need to be able to pass <code>animate</code> style properties and values. We typically think in terms of stylesheet values rather than DOM properties, so some frameworks parse CSS properties into valid DOM properties.

Values themselves need to be translated. For example, <code>50px</code> or <code>2em</code> need to have the units extracted so the numbers can be manipulated, then added back again:

{% highlight javascript %}
{ 'marginLeft': '10px' }
// Becomes:
parsedStyles['marginLeft'] = { number: '10', units: 'px' }
// Then the transform looks like this:
for (var property in parsedStyles) {
  element.style[property] = parsedStyles[property].number * position + parsedStyles[property].units
}
{% endhighlight %}

Simply allowing properties to be collected together in objects allows multiple properties to be animated at once.

### Parsing Style Values

To get the values into the format above, I've simply used <code>parseFloat</code> and <code>String.prototype.replace</code>:

{% highlight javascript %}
function parseCSSValue(value) {
  var n = parseFloat(value);
  return { number: n, units: value.replace(n, '') };
}
{% endhighlight %}

This can't cope with many types of values -- colours won't work for example, but it will work for animating positions.

### API

The Turing animation API currently looks like this:

{% highlight javascript %}
turing.anim.animate(element, 1000, { 'marginLeft': '8em', 'marginTop': '100px' });
{% endhighlight %}

Durations are in milliseconds and are a required parameter. There's also a fourth parameter that isn't shown here which will allow callbacks to be specified.

I haven't bothered converting CSS property names into DOM names because it seemed like unnecessary complexity.

The full code is available in GitHub in [turing.anim.js](http://github.com/alexyoung/turing.js/blob/master/turing.anim.js).

### Next Week

Next week I'll add support for colour transitions and chained animations.

### References

-   [jQuery Animation](http://api.jquery.com/animate/)
-   [Emile](http://github.com/madrobby/emile/blob/master/emile.js)
-   [How Timers Work](http://ejohn.org/blog/how-javascript-timers-work/)
