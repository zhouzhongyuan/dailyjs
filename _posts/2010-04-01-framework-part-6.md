---
layout: post
title: "Let's Make a Framework: Selectors"
author: Alex Young
categories: 
- web
- frameworks
- tutorials
- lmaf
---

Welcome to part 6 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part explores CSS selectors.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/). You can contribute! Fork and message me with your changes.

I've been adding more functional methods to the framework behind the scenes, if you want to keep up follow on GitHub.

In this part I'll explain the basics behind selector engines, explore the various popular ones, and examine their characteristics. This is a major area for JavaScript web frameworks -- it'll take me a few posts to do it justice.

### History

The importance and need for cross-browser selector APIs is a big thing. Whether the selectors are XPath or CSS, browser implementors are not consistent.

To understand why a selector engine is important, imagine working in the late 90s without a JavaScript framework:

{% highlight javascript %}
document.all    // A proprietary property in IE4
document.getElementById('navigation');
{% endhighlight %}

The first thing that people realised was this could be shortened. [Prototype](http://prototypejs.org) does this with <code>$()</code>:

{% highlight javascript %}
function $(element) {
  if (arguments.length > 1) {
    for (var i = 0, elements = [], length = arguments.length; i < length; i++)
      elements.push($(arguments[i]));
    return elements;
  }
  if (Object.isString(element))
    element = document.getElementById(element);
  return Element.extend(element);
}
{% endhighlight %}

Rather than simply aliasing <code>document.getElementById</code>, Prototype can search for multiple IDs and extends elements with its own features.

What we really wanted though was <code>getElementsBySelector</code>. We don't just think in terms of IDs or tag names, we want to apply operations to sets of elements. CSS selectors are how we usually do this for styling, so it makes sense that similarly styled objects should *behave* in a similar way.

Simon Willison wrote [getElementsBySelector()](http://simonwillison.net/2003/Mar/25/getElementsBySelector/) back in 2003. This implementation is partially responsible for a revolution of DOM traversal and manipulation.

Browsers provide more than just <code>getElementById</code> -- there's also [getElementsByClassName](http://www.whatwg.org/specs/web-apps/current-work/multipage/dom.html#dom-getelementsbyclassname), [getElementsByName](http://www.whatwg.org/specs/web-apps/current-work/multipage/dom.html#dom-document-getelementsbyname), and lots of other DOM-related methods. Most non-IE browsers support searching using XPath expressions using [evaluate](http://www.w3.org/TR/DOM-Level-3-XPath/xpath.html#XPathEvaluator-evaluate).

Modern browsers also support [querySelector and querySelectorAll](http://www.w3.org/TR/selectors-api/). Again, these methods are limited by poor IE support.

### Browser Support

Web developers consider browser support a necessary evil. In selector engines, however, browser support is a major concern. Easing the pain of working with the DOM and getting reliable results is fundamental.

I've seen some fascinating ways to work around browser bugs. Look at this code from [Sizzle](http://sizzlejs.com/):

{% highlight javascript %}
// Check to see if the browser returns elements by name when
// querying by getElementById (and provide a workaround)
(function(){
  // We're going to inject a fake input element with a specified name
  var form = document.createElement("div"),
  id = "script" + (new Date()).getTime();
  form.innerHTML = "<a name='" + id + "'/>";
   
  // Inject it into the root element, check its status, and remove it quickly
  var root = document.documentElement;
  root.insertBefore( form, root.firstChild );
{% endhighlight %}

It creates a fake element to probe browser behaviour. Supporting browsers is a black art beyond the patience of most well-meaning JavaScript hackers.

### Performance

Plucking elements from arbitrary places in the DOM is useful. So useful that developers do it a lot, which means selector engines need to be fast.

Falling back to native methods is one way of achieving this. Sizzle looks for <code>querySelectorAll</code>, with code to support browser inconsistencies.

Another way is to use caching, which Sizzle and Prototype both do.

One way to measure selector performance is through a tool like [slickspeed](http://github.com/kamicane/slickspeed) or a library like [Woosh](http://github.com/jakearchibald/Woosh/). These tools are useful, but you have to be careful when interpreting the results. Libraries like Prototype and MooTools extend elements, whereas pure selector engines like Sizzle don't. That means they might look slow compared to Sizzle but they're actually fully-fledged frameworks rather than a pure selector engine.

### Other Selector Engines

I've been referencing popular frameworks and Sizzle so far, but there are other Sizzle-alikes out there. Last year there was a burst of several, perhaps in reaction to the first release of Sizzle. Sizzle has since been gaining popularity with framework implementors again, so I'm not sure if any really made an impact.

[Peppy](http://github.com/jdonaghue/Peppy) draws on inspiration from lots of libraries and includes some useful comments about dealing with caching. [Sly](http://github.com/digitarald/sly) has a lot of work on optimisation and allows you to expose the parsing of selectors. Both Sly and Peppy are fairly large chunks of code -- anything comparable to Sizzle is not a trivial project.

### API Design

Generally speaking, there are two kinds of APIs. One approach uses a function that returns elements that match a selector, but wraps them in a special class. This class can be used to chain calls that perform complex DOM searches or manipulations. This is how jQuery, Dojo and Glow work.

jQuery gives us <code>$()</code>, which can be used to query the document for nodes, then chain together calls on them.

Dojo has <code>dojo.query</code> which returns an Array-like [dojo.NodeList](http://www.dojotoolkit.org/reference-guide/dojo/NodeList.html#dojo-nodelist). Glow is similar, with [glow.dom.get](http://www.bbc.co.uk/glow/docs/1.7/api/glow.dom.shtml)

The second approach is where elements are returned and extended. Prototype and MooTools do this.

Prototype has <code>$()</code> for <code>getElementById</code> and <code>$$()</code> for querying using CSS or XPath selectors. Prototype extends elements with its own methods. MooTools behaves a lot like this. Both can work with strings or element references.

Turing has been designed much like Glow-style libraries, so we'll use this approach for our API. There's a lazy aspect to this design that I like -- it might be possible to return unprocessed elements wrapped in an object, then only deal with masking browser inconsistencies when they're actually manipulated:

{% highlight javascript %}
turing.dom.find('.class')                  // return elements wrapped in a class without looking at each of them
.find('a')                                 // find elements that are links
.css({ 'background-color': '#aabbcc' })    // apply the style by actually processing elements
{% endhighlight %}

### Goals

Because Turing exists as an educational framework, it's probably wise if we keep it simple:

-   CSS selectors only (XPath could be a future upgrade or plugin)
-   Limited pseudo-selectors. Implementing one or two would be nice just to establish the API for them
-   Defer to native methods as quickly as possible
-   Cache for performance
-   Expose the parsing results like Sly does
-   Reuse the wisdom of existing selector engines for patching browser nightmares

### Conclusion

I hope you've learned why selector engines are so complex and why we need them. Granted, it's mostly due to browser bugs that don't need to exist, but browsers and standards both evolve out of step, so sometimes it's our job to patch the gaps while they catch up.

While it's true that Sizzle is the mother of all selector engines, I think implementing a small, workable one will be a worthwhile project. Join me in part 7 where I'll start implementing the selector engine.
