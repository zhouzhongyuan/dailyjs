---
layout: post
title: "Let's Make a Framework: JSDoc"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- documentation
---

Welcome to part 47 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

### JSDoc

Last week I reviewed various approaches to JavaScript documentation, and I decided to use [dox](https://github.com/visionmedia/dox). Dox can be installed with npm:

{% highlight sh %}
npm install dox
{% endhighlight %}

By the end of this tutorial, you'll be able to make something like this:

![](/images/posts/turing_documentation.png)

Although dox produces readable documentation -- which is incidentally perfect for writing tutorials because the code is inline -- it can't parse everything JSDoc offers. It's a simple tool designed to suit the author's requirements, but I like it and would like to see it grow in the future.

Despite not supporting everything outlined by [jsdoc-toolkit](http://code.google.com/p/jsdoc-toolkit/), I'm going to use dox and write JSDoc comments as if it fully supports them.

### JSDoc Comment Structure

Each file in your project should contain author, copyright, and license details. For example:

{% highlight javascript %}
/*!
 * Turing - Module Name
 * Copyright (C) 2010-2011 Alex R. Young 
 * MIT Licensed
 */

/**
 * A private namespace to set things up against the global object.
 */
(function(global) {
{% endhighlight %}

I like to write comments in a relatively informal conversational style, as full sentences. I might be extra-verbose because this is a tutorial, so you can be a bit more succinct in your own projects.

Where JSDoc comments come in handy is outlining the intended inputs and outputs of a function:

{% highlight javascript %}
  /**
   * Determine if an object is an `Array`.
   *
   * @param {Object} object An object that may or may not be an array
   * @returns {Boolean} True if the parameter is an array
   */
  turing.isArray = Array.isArray || function(object) {
    return !!(object && object.concat
              && object.unshift && !object.callee);
  };
{% endhighlight %}

Notice I've used backticks around the <code>Array</code> keyword. Dox has markdown support, so this will be converted into a <code>code</code> tags in the resulting HTML.

The <code>`param</code> and <code>`returns</code> directives are known as *tags*. The [jsdoc-toolkit](http://code.google.com/p/jsdoc-toolkit/wiki/) wiki lists lots of tags. The [@param](http://code.google.com/p/jsdoc-toolkit/wiki/TagParam) tag actually has quite a few ways of defining method parameters, including support for optional parameters.

### Long Examples

Because dox supports markdown, long code examples can be included simply through indentation:

{% highlight javascript %}
/**
 * Chain animation module calls, for example:
 *
 *     turing.anim.chain(element)
 *       .highlight()
 *       .pause(250)
 *       .move(100, { x: '100px', y: '100px', easing: 'ease-in-out' })
 *       .animate(250, { width: '1000px' })
 *       .fadeOut(250)
 *       .pause(250)
 *       .fadeIn(250)
 *       .animate(250, { width: '20px' });
 *
 * @param {Object} element A DOM element
 * @returns {Chainer} Chained API object
 */
anim.chain = function(element) {
  return new Chainer(element);
};
{% endhighlight %}

### Generating the Documentation

Dox can be run with just <code>dox file.js</code>, and it'll output the HTML to the console. The HTML is totally self-contained, which means there are no extra image, CSS, or JavaScript files. This will generate the documentation for [turing.core.js](https://github.com/alexyoung/turing.js/blob/master/turing.core.js):

{% highlight sh %}
dox --title Turing turing.core.js  > index.html
{% endhighlight %}

I also added this to the project's Jakefile (node-jake):

{% highlight javascript %}
var exec = require('child_process').exec;

desc('Documentation');
task('docs', [], function() {
  exec('dox --title Turing *.js  > docs/index.html');
});
{% endhighlight %}

### Build Systems

The build system is closely tied to project management tasks like generating documentation. Generating documentation should be consistent and easy to run so it doesn't get forgotten as the project changes over time, therefore adding a *docs* task is a good idea.

Every time I go over TJ Holowaychuk's projects (and [jQuery](https://github.com/jquery/jquery/blob/master/Makefile)) I'm reminded that GNU <code>make</code> was more than enough for our needs. Build system choice is partially dependent on how much JavaScript from Node, Narwhal, Rhino, etc. you want to reuse in your build system -- Python, Java, and Ruby projects often use native build systems to get access to native libraries. Also, you already know how to write JavaScript pretty well but might not be familiar with <code>make</code>.

However, when writing your own projects <code>make</code> is more likely to be on a system than [node-jake](https://github.com/mde/node-jake), and it could always invoke JavaScript scripts anyway. I recommend giving it a look before deciding on the build system for your own projects.

Side note: I've removed the Narwhal support from the Jakefile to focus on using Node for the build system instead. The main motivation for this was to get some cross-pollination between the Node tutorial series and this series.

### References

-   [dox](https://github.com/visionmedia/dox)
-   [jsdoc-toolkit wiki](http://code.google.com/p/jsdoc-toolkit/w/list)
-   [node-jake](https://github.com/mde/node-jake)
-   [jQuery's Makefile](https://github.com/jquery/jquery/blob/master/Makefile)
