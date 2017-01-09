---
layout: post
title: "Let's Make a Framework: Animations Part 8"
author: Alex Young
categories: 
- frameworks
- tutorials
- animation
- lmaf
---

Welcome to part 22 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at JavaScript animations.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

Last week I explained how CSS3 animations work. In this part I'm going to demonstrate how to implement support for them using JavaScript.

### CSS3 Feature Detection

The state of CSS3 support is currently in flux because the standards aren't ready yet. Most browsers still use vendor prefixed tags, which means we need to know what browser we're dealing with.

Detecting browser support for CSS3 is a little bit tricky, but it's not impossible. WebKit browsers have an event object called <code>WebKitTransitionEvent</code>, and Opera uses <code>OTransitionEvent</code>. Firefox has a style attribute called <code>MozTransition</code>.

I've created an object with a list of properties that can be used to query vendor support:

{% highlight javascript %}
// CSS3 vendor detection
vendors = {
  // Opera Presto 2.3
  'opera': {
    'prefix': '-o-',
    'detector': function() {
      try {
        document.createEvent('OTransitionEvent');
        return true;
      } catch(e) {
        return false;
      }
    }
  },

  // Chrome 5, Safari 4
  'webkit': {
    'prefix': '-webkit-',
    'detector': function() {
      try {
        document.createEvent('WebKitTransitionEvent');
        return true;
      } catch(e) {
        return false;
      }
    }
  },

  // Firefox 4
  'firefox': {
    'prefix': '-moz-',
    'detector': function() {
      var div = document.createElement('div'),
          supported = false;
      if (typeof div.style.MozTransition !== 'undefined') {
        supported = true;
      }
      div = null;
      return supported;
    }
  }
};

function findCSS3VendorPrefix() {
  for (var detector in vendors) {
    detector = vendors[detector];
    if (detector['detector']()) {
      return detector['prefix'];
    }
  }
}
{% endhighlight %}

### Move Animation Implementation

To use CSS3 for move animations, we need to do the following:

-   Detect when a CSS property is being used to move an element
-   Get the vendor prefix
-   Set up CSS3 style properties instead of using Turing's animation engine

### Detecting when a CSS Property Means Move

The convention I've been using is to manipulate the <code>left</code> or <code>top</code> style properties to move an element. Whenever these properties are animated and a vendor prefix has been found, then we can use CSS transitions.

The best place to do this is in the property loop inside <code>anim.animate</code>:

{% highlight javascript %}
for (var property in properties) {
  if (properties.hasOwnProperty(property)) {
    properties[property] = parseCSSValue(properties[property], element, property);
    if (property == 'opacity' && opacityType == 'filter') {
      element.style.zoom = 1;
    } else if (CSSTransitions.vendorPrefix && property == 'left' || property == 'top') {
      // Do CSS3 stuff here and return before Turing animates with its own routines
    }
  }
}
{% endhighlight %}

I've stolen <code>camelize</code> from [Prototype](http://prototypejs.org/) to make writing out CSS easier:

{% highlight javascript %}
element.style[camelize(this.vendorPrefix + 'transition')] = property + ' ' + duration + 'ms ' + (easing || 'linear');
element.style[property] = value;
{% endhighlight %}

In the case of Firefox 4, this would translate to:

{% highlight javascript %}
element.style[MozTransition] = 'left 1000ms linear';
element.style['left'] = '100px';
{% endhighlight %}

I've put this in a function called <code>start</code>, and I've also added an <code>end</code> function to clear the transition afterwards:

{% highlight javascript %}
CSSTransitions = {
  // ...

  start: function(element, duration, property, value, easing) {
    element.style[camelize(this.vendorPrefix + 'transition')] = property + ' ' + duration + 'ms ' + (easing || 'linear');
    element.style[property] = value;
  },

  end: function(element, property) {
    element.style[camelize(this.vendorPrefix + 'transition')] = null;
  }
};
{% endhighlight %}

The core of the property loop now looks like this:

{% highlight javascript %}
CSSTransitions.start(element, duration, property, properties[property].value + properties[property].units, options.easing);
setTimeout(function() { CSSTransitions.end(element, property); }, duration);
return;
{% endhighlight %}

### Conclusion

Defining CSS3 transitions in JavaScript isn't too difficult once the vendor has been detected. A JavaScript API would be nicer, but generating the DOM version of the equivalent CSS isn't too difficult.

This version misses explicit transition support -- the Turing transition names would have to be mapped to CSS3 ones (or we could just switch to the names used by CSS).
