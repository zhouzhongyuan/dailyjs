---
layout: post
title: "Let's Make a Framework: NodeList, Collections and Arrays"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- dom
- NodeList
---

Welcome to part 34 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Last week I talked about using <code>querySelectorAll</code>. John-David Dalton noticed that I missed part of the process -- converting the resulting <code>NodeList</code> into an <code>Array</code>. This is an important step which I'll discuss in this part.

### NodeList and Collections

DOM Level 1 defines <code>NodeList</code> as an abstraction of ordered collections of nodes. The specification is kept simple to avoid constraining the underlying implementation. That means collections of DOM elements can't be directly manipulated like an array, although it's trivial to iterate over each element:

{% highlight javascript %}
// Get the elements
var elements = document.querySelectorAll('p');

// Iterate over each element with a simple for loop
for (var i = 0; i < elements.length; i++) {
  console.log(elements[i]);
}
{% endhighlight %}

To see why <code>NodeList</code> doesn't do what we want, you'll notice <code>Array.prototype</code> methods are missing:

{% highlight javascript %}
document.querySelectorAll('p').push
// returns undefined
{% endhighlight %}

Ouch!

Another interesting point about <code>NodeList</code> is it's an *ordered* collection. From [DOM Level 1 Core](http://www.w3.org/TR/REC-DOM-Level-1/level-one-core.html):

> <code>getElementsByTagName</code> Returns a NodeList of all descendant elements with a given tag name, in the order in which they would be encountered in a preorder traversal of the Element tree.

Over at the [Mozilla NodeList](https://developer.mozilla.org/En/DOM/NodeList) documentation, they have this to say:

> This is a commonly used type which is a collection of nodes returned by getElementsByTagName, getElementsByTagNameNS, and Node.childNodes. The list is live, so changes to it internally or externally will cause the items they reference to be updated as well. Unlike NamedNodeMap, NodeList maintains a particular order (document order). The nodes in a NodeList are indexed starting with zero, similarly to JavaScript arrays, but a NodeList is not an array.

This isn't just what Mozilla implementations do, technically all browsers should. This is from the specifications:

> NodeLists and NamedNodeMaps in the DOM are "live", that is, changes to the underlying document structure are reflected in all relevant NodeLists and NamedNodeMaps.

Mozilla's documentation also points out a gotcha that I've run into before:

> Don't be tempted to use <code>for...in</code> or <code>for each...in</code> to enumerate the items in the list, since that will also enumerate the length and item properties of the NodeList and cause errors if your script assumes it only has to deal with element objects.

### Converting NodeList into an Array

The simplest approach is probably the best:

{% highlight javascript %}
function toArray(collection) {
  var results = [];
  for (var i = 0; i < collection.length; i++) {
    results.push(collection[i]);
  }
  return results;
}
{% endhighlight %}

The major downside of this is the results will no-longer be *live* -- the original NodeList is a reference to a set of objects rather than a fixed result set.

### In the Wild

jQuery has a method called <code>makeArray</code>:

{% highlight javascript %}
makeArray: function( array, results ) {
  var ret = results || [];

  if ( array != null ) {
    // The window, strings (and functions) also have 'length'
    // The extra typeof function check is to prevent crashes
    // in Safari 2 (See: #3039)
    if ( array.length == null || typeof array === "string" || jQuery.isFunction(array) || (typeof array !== "function" && array.setInterval) ) {
      push.call( ret, array );
    } else {
      jQuery.merge( ret, array );
    }
  }

  return ret;
}
{% endhighlight %}

In this code, <code>push</code> refers to <code>Array.prototype.push</code>.

When Prototype uses <code>querySelectorAll</code>, it wraps the output in <code>$A()</code> and uses <code>.map(Element.extend)</code> to make each element a Prototype <code>Element</code>. This is similar to the above, with the exception of Prototype extending each element.

Some other frameworks wrap the results in their own <code>NodeList</code> class, rather than converting them to an array.

### Implementation

The <code>toArray</code> function described above has been added to [turing.core.js](http://github.com/alexyoung/turing.js/blob/master/turing.core.js) in the form of <code>turing.toArray</code> and added to <code>turing.dom.get</code>.

### References

-   [Document Object Model (Core) Level 1](http://www.w3.org/TR/REC-DOM-Level-1/level-one-core.html)
-   [Mozilla NodeList](https://developer.mozilla.org/En/DOM/NodeList) documentation
