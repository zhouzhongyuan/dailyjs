---
layout: post
title: "Let's Make a Framework: Animations Part 7"
author: Alex Young
categories: 
- frameworks
- tutorials
- animation
- lmaf
---

Welcome to part 21 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at JavaScript animations.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

In this part I'm going to introduce CSS3 animations, and hardware accelerated animation.

### The Specifications

CSS3 introduces several modules for dealing with animation. All of the relevant specifications are currently working drafts, with a lot of input from Apple. Therefore, if you're interested in trying these examples WebKit-based browsers (and particularly Safari in Mac OS) are currently the best way of trying them out.

The specifications are:

-   [CSS Transitions Module Level 3](http://www.w3.org/TR/css3-transitions/)
-   [CSS 2D Transforms Module Level 3](http://www.w3.org/TR/css3-2d-transforms/)
-   [CSS 3D Transforms Module Level 3](http://www.w3.org/TR/css3-3d-transforms/)
-   [CSS Animations Module Level 3](http://www.w3.org/TR/css3-animations/)

### CSS3 Transitions

Transitions are useful because CSS properties can be animated, much like our framework code. The syntax is fairly easy to follow:

{% highlight css %}
.example {
	-webkit-transition: all 1s ease-in-out;
	-moz-transition: all 1s ease-in-out;
	-o-transition: all 1s ease-in-out;
	-webkit-transition: all 1s ease-in-out;
	transition: all 1s ease-in-out;
}
{% endhighlight %}

The <code>all</code> instruction refers to the CSS property to animate -- you could single out <code>margin</code> if required.

{% highlight html %}
<style>
#move {
  background-color: #ff0000;
  width: 100px;
  height: 100px;
	-webkit-transition: all 5s ease-in-out;
	-moz-transition: all 5s ease-in-out;  
	-o-transition: all 5s ease-in-out;  
	transition: all 5s ease-in-out;  
}

#move-container:hover #move {
  margin-left: 440px;
  width: 200px
}
</style>

<div id="move-container">
  <div id="move">
  </div>
</div>
{% endhighlight %}

The previous example should create a red box that stretches and moves (a *translation*) over 5 seconds.

### CSS3 Transforms

There are several different transforms:

{% highlight css %}
transform: translate(50px, 200px);
transform: rotate(45deg);
transform: skew(45deg);
transform: scale(1, 2);
{% endhighlight %}

These transforms can be combined with <code>transition</code> to animate for an event, like hovering over an element.

### CSS3 Animations

Animations aren't well supported right now, but they do provide a simple way of creating *keyframes* for the animation over time. The CSS looks like this:

{% highlight javascript %}
@keyframes 'wobble' {

  0% {
    left: 100px;
  }

  40% {
    left: 150px;
  }

  60% {
    left: 75px;
  }

  100% {
    left: 100px;
  }

}
{% endhighlight %}

I think it's likely that many JavaScript animation frameworks will leave out support for animations, since they'll have equivalent functionality already.

### Animation Performance Problems

In [QtWebkit and Graphics](http://trac.webkit.org/wiki/QtWebKitGraphics) at [webkit.org](http://webkit.org), the authors discuss the problems caused when trying to animate using the box model. Animating margin, padding, background, outline, and border will all result in relatively slow animations.

The [CSS Animations Module Level 3](http://www.w3.org/TR/css3-animations/) is seen as a way around traditional animation performance issues. The working draft's editors are all from Apple, and people are still arguing about whether or not animation belongs in CSS or JavaScript. However, it's currently the only way of getting hardware acceleration that I'm aware of.

### Hardware Acceleration

Most computers and devices feature specialised graphics chips. If hardware acceleration sounds mysterious to you, it's nothing more than using these chips instead of the CPU. The browser has to perform a huge amount of work to calculate changes when animating elements, but offloading this work to software and hardware that deals with graphics reduces CPU load.

CSS3 animations should look noticeably slicker on devices with slower CPUs like the iPhone.

### Feature Detection

We generally test browser support by checking if an object or property is available, rather than forking code based on the browser's version strings. There are no properties that can give away hardware acceleration support, [so in Scripty2](http://github.com/madrobby/scripty2/blob/master/src/effects/css_transitions.js) Thomas Fuchs does this:

{% highlight javascript %}
function isHWAcceleratedSafari() {
  var ua = navigator.userAgent, av = navigator.appVersion;
  return (!ua.include('Chrome') && av.include('10_6')) ||
   Prototype.Browser.MobileSafari;
}
{% endhighlight %}

CSS3 support is detected by checking if <code>WebKitTransitionEvent</code> or <code>MozTransition</code> exists:

{% highlight javascript %}
var div = document.createElement('div');

try {
  document.createEvent("WebKitTransitionEvent");
  supported = true;
  
  hardwareAccelerationSupported = isHWAcceleratedSafari();
} catch(e) {
  if (typeof div.style.MozTransition !== 'undefined') {
    supported = true;
  }
}

div = null;
{% endhighlight %}

Then there's the problem of translating CSS3 animation properties to vendor-specific properties.

### Conclusion

It will be certainly possible to make us of CSS3 animations in our framework, but it will require a fair bit of work:

-   CSS3 support needs to be sniffed
-   Anything that can be replaced with CSS3 animations should be swapped out when Turing's <code>animate</code> method is called with the relevant CSS properties
-   Vendor-specific properties should be used where appropriate
