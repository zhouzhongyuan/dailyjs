---
layout: post
title: "Let's Make a Framework: Animations Part 5"
author: Alex Young
categories: 
- frameworks
- tutorials
- animation
- lmaf
---

Welcome to part 19 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at JavaScript animations.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Colour Support

Last week I mentioned that I wanted to build in a helper for highlighting an element (the Yellow Fade Technique). The problem with this is the library doesn't currently support colour parsing.

The reason colour parsing is required is we need to convert values to <code>rgb(0, 0, 0)</code> notation: most people will expect to be able to pass hexadecimal colours.

I based our colour parser on Stoyan Stefanov's parser here: [RGB color parser in JavaScript](http://www.phpied.com/rgb-color-parser-in-javascript/). The coding style is particularly suitable for this tutorial series, because it presents the concepts in easy to read code.

The core of the colour parser is an array of regular expressions and functions that can extract numbers for a given format and turn it into RGB values:

{% highlight javascript %}
Colour.matchers = [
  {
    re: /^rgb\((\d{1,3}),\s*(\d{1,3}),\s*(\d{1,3})\)$/,
    example: ['rgb(123, 234, 45)', 'rgb(255,234,245)'],
    process: function (bits){
      return [
        parseInt(bits[1]),
        parseInt(bits[2]),
        parseInt(bits[3])
      ];
    }
  },
{% endhighlight %}

Notice the <code>example</code> property documents how the matcher works which is a great touch that I lifted directly from Stoyan's code (which I think he got from Simon Willison's date library).

Now the parser just needs to loop through each matcher. There are only 3, and colour parsing will only be used outside of the main animation loop:

{% highlight javascript %}
Colour.prototype.parse = function() {
  var channels = [], i;
  // Loop through each matcher
  for (i = 0; i < Colour.matchers.length; i++) {
    channels = this.value.match(Colour.matchers[i].re);
    // An array of numbers will be returned if the value was matched by the regex
    // else null
    if (channels) {
      // Set each value to r, g, b properties in this object
      channels = Colour.matchers[i].process(channels);
      this.r = channels[0];
      this.g = channels[1];
      this.b = channels[2];
      break;
    }
  }
  this.validate();
}
{% endhighlight %}

I could have used an array for each property here, but I thought I'd be explicit and use <code>r, g, b</code> properties for each value. This means there is some code repetition later, but at least you can easily see what's going on.

Something else I took from Stoyan's library is the validation code:

{% highlight javascript %}
Colour.prototype.validate = function() {
  this.r = (this.r < 0 || isNaN(this.r)) ? 0 : ((this.r > 255) ? 255 : this.r);
  this.g = (this.g < 0 || isNaN(this.g)) ? 0 : ((this.g > 255) ? 255 : this.g);
  this.b = (this.b < 0 || isNaN(this.b)) ? 0 : ((this.b > 255) ? 255 : this.b);
}
{% endhighlight %}

This just makes sure values are between 0 and 255.

### Channel Surfing

Something I realised early on when developing this was channels should be able to move independently. Imagine if you wanted to animate this:

{% highlight javascript %}
// From:
rgb(55, 255, 0);

// To:
rgb(255, 0, 10);
{% endhighlight %}

The red, green and blue properties are moving in different directions. Our easing-based transforms so far just multiply values by position, so we need something that can:

-   Figure out the "direction" of each colour
-   Correctly multiply each colour by the current position in the animation

Direction can be represented by -1 and 1. We don't need to conditionally check for direction though, the values can just be multiplied by the direction:

{% highlight javascript %}
nextRed = startRed + (redDirection * (Math.abs(startRed - endRed) * easingFunction(position)))
{% endhighlight %}

<code>Math.abs</code> is used here to make sure the value isn't negative. Technically the start value might be less than the end value -- we just need to know the magnitude:

{% highlight javascript %}
155 - 55
// 100
55 - 155
// -100
Math.abs(55 - 155)
100
{% endhighlight %}

### Transformations

We might find CSS values are numbers or colours. To handle this, I've made Turing normalise properties we want to animate into a format that includes a "transformation" function. These functions know how to correctly manipulate colours or numbers. This was already hinted at because we needed the basis to write out units (px, em, etc.)

The core animation loop now includes a reference to a <code>transform</code> property that the CSS value parser sets.

The colour transform looks like this:

{% highlight javascript %}
function colourTransform(v, position, easingFunction) {
  var colours = [];
  colours[0] = Math.round(v.base.r + (v.direction[0] * (Math.abs(v.base.r - v.value.r) * easingFunction(position))));
  colours[1] = Math.round(v.base.g + (v.direction[1] * (Math.abs(v.base.g - v.value.g) * easingFunction(position))));
  colours[2] = Math.round(v.base.b + (v.direction[2] * (Math.abs(v.base.b - v.value.b) * easingFunction(position))));
  return 'rgb(' + colours.join(', ') + ')';
}
{% endhighlight %}

It doesn't bother using <code>new Colour(...)</code> to set up a new colour because we just care about the RGB values here.

I got the idea for normalised CSS values into objects that contain unit and value presentation functions from [Emile](http://github.com/madrobby/emile/).

### Highlight Helper

The highlight helper function can now be written:

{% highlight javascript %}
anim.highlight = function(element, duration, options) {
  var style = element.currentStyle ? element.currentStyle : getComputedStyle(element, null);
  options = options || {};
  options.from = options.from || '#ff9';
  options.to = options.to || style.backgroundColor;
  options.easing = options.easing || easing.sine;
  duration = duration || 500;
  element.style.backgroundColor = options.from;
  return setTimeout(function() {
    anim.animate(element, duration, { 'backgroundColor': options.to, 'easing': options.easing })
  }, 200);
};

// Usage:

turing.anim.highlight(element);
{% endhighlight %}

It sets a bunch of defaults then sets a bright yellow background colour. After that it animates fading the background colour to the original colour. <code>getComputedStyle</code> has to be used to get the current colour. There might be instances where passing <code>options.from</code> is a better idea.

### Conclusion

Animating CSS values is difficult because we need to detect what type of value we're dealing with, parse it, change it, then correctly update the DOM with the new values. By keeping the parsed results non-specific the animation loop can be kept decoupled from the code that does the real work.

Eventually the CSS-related code should be moved to <code>turing.css</code>, because it could be useful outside of animations.

To play with the code and make some experiments of your own, the easiest way is to [check out the source](http://github.com/alexyoung/turing.js/), cd into <code>test</code> and open up <code>anim\_test.html</code> in a browser and <code>anim\_test.js</code> in an editor.

If you find bugs or seriously awesome improvements, why not write a guest post for the series?

### References

-   [RGB color parser in JavaScript](http://www.phpied.com/rgb-color-parser-in-javascript/)
-   [Emile](http://github.com/madrobby/emile/)
-   [Effect.Highlight in scriptaculous](http://wiki.github.com/madrobby/scriptaculous/effect-highlight)
