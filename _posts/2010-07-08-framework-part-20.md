---
layout: post
title: "Let's Make a Framework: Animations Part 6"
author: Alex Young
categories: 
- frameworks
- tutorials
- animation
- lmaf
---

Welcome to part 20 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at JavaScript animations.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Movement Helper

Continuing on from last week with helpers, I would like to be able to do this:

{% highlight javascript %}
turing.anim.move(element, duration, { x: '100px', y: '100px' });
{% endhighlight %}

And the element should move 100 pixels across and down from its current position. The API we've built over the last few weeks will do this for relative positioned elements with the following code:

{% highlight javascript %}
anim.animate(element, duration, { 'left': options.x, 'top': options.y });
{% endhighlight %}

Using an absolute positioned element will cause it to jump to the first point in the animation. That means we need to move absolute elements *relative* to their original location. I'll come back to this later.

### Chained API

Ideally animation calls, whether they be from helpers or <code>animate</code>, should work when chained. Chaining calls should create a sequence of animations:

{% highlight javascript %}
turing.anim.chain(element)
  .highlight()
  .move(1000, { x: '100px', y: '100px' })
  .animate(2000, { height: '0px' });
{% endhighlight %}

It would be useful to be able to pause animations, too:

{% highlight javascript %}
turing.anim.chain(element)
  .highlight()
  .pause(2000)
  .move(1000, { x: '100px', y: '100px' })
  .animate(2000, { width: '1000px' })
  .fadeOut(2000)
  .pause(2000)
  .fadeIn(2000)
  .animate(2000, { width: '20px' })
{% endhighlight %}

I've used as similar technique to the [Enumerable library's Chainer class](http://dailyjs.com/2010/03/25/framework-part-5/) (part 5). This isn't reused here but it could be made more generic.

The basic principle is to wrap each chainable method in a function that can correctly sequence the animation calls. The <code>anim</code> object contains all of the relevant methods; this object can be iterated over:

{% highlight javascript %}
for (methodName in anim) {
  (function(methodName) {
    var method = anim[methodName];
    // ...    
  })(methodName);
}
{% endhighlight %}

The anonymous function causes the <code>methodName</code> variable to get bound to the scope. The <code>Chainer</code> class needs to keep track of the element we're animating, and the current position in time:

{% highlight javascript %}
Chainer = function(element) {
  this.element = element;
  this.position = 0;
};
{% endhighlight %}

Then as the animation progresses, we need to update the position:

{% highlight javascript %}
for (methodName in anim) {
  (function(methodName) {
    var method = anim[methodName];
    Chainer.prototype[methodName] = function() {
      this.position += current position;
    };
  })(methodName);
}
{% endhighlight %}

<code>setTimeout</code> can be used to schedule the animations:

{% highlight javascript %}
setTimeout(function() {
  method.apply(null, args);
}, this.position);
{% endhighlight %}

This will not block, so all of the chained animations will be started at around the same time. Since <code>position</code> is incremented with each animation's duration, the animations should fire at approximately the right time.

The only other thing left to do is progress the <code>arguments</code> passed to the function to insert the element.

### Putting it Together

{% highlight javascript %}
Chainer = function(element) {
  this.element = element;
  this.position = 0;
};

for (methodName in anim) {
  (function(methodName) {
    var method = anim[methodName];
    Chainer.prototype[methodName] = function() {
      var args = Array.prototype.slice.call(arguments);
      args.unshift(this.element);
      this.position += args[1] || 0;
      setTimeout(function() {
        method.apply(null, args);
      }, this.position);
      return this;
    };
  })(methodName);
}

anim.chain = function(element) {
  return new Chainer(element);
};
{% endhighlight %}

### Conclusion

Now chaining has been added it's apparent that the animation framework has some warts: in particular, movement animations will start from start positions. I'll look at fixing these issues over the coming weeks.
