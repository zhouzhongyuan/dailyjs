---
layout: post
title: "Let's Make a Framework: Writing Documentation"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- documentation
---

Welcome to part 46 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Over the next few parts I'm going to discuss writing documentation for JavaScript projects. If you're writing open source code documentation isn't just a polite addition to a project, it goes a long way to helping your project take off. A well-written README helps, but full-on API documentation makes projects look professional. Even if your project is closed-source, documentation will help new colleagues get on board more quickly, or help you remember how things work for long-running projects.

Let's take a look at the popular JavaScript frameworks to see how they manage their documentation.

### jQuery

![](/images/posts/jquery_docs.png)

jQuery's documentation is hosted on [docs.jquery.com](http://docs.jquery.com/), which is a wiki with searchable documentation for the entire API. Each major API area is listed in the navigation, and each page has a list of the methods for that area. The pages contain example code and comments through Disqus.

The source code comments are mainly concerned with notes about specific bugs or unusual bits of code that need explanation.

### Prototype

Prototype's documentation, available at [api.prototypejs.org](http://api.prototypejs.org/), is generated by [pdoc](http://pdoc.org/). PDoc is:

> ... an inline comment parser and JavaScript documentation generator written in Ruby. It is designed for documenting Prototype and Prototype-based libraries.

Prototype uses comments written in markdown that can be generated into documentation using pdoc. Take a look at [src/prototype/ajax.js](https://github.com/sstephenson/prototype/blob/master/src/prototype/ajax.js) to see an example.

{% highlight javascript %}
/**
 *  == Ajax ==
 *
 *  Prototype's APIs around the `XmlHttpRequest` object.
 *
 *  The Prototype framework enables you to deal with Ajax calls in a manner that
 *  is both easy and compatible with all modern browsers.
 *
 *  Actual requests are made by creating instances of [[Ajax.Request]].
{% endhighlight %}

In contrast to jQuery, the documentation is inside the framework's source itself.

### JSDoc

JSDoc is actually a way to write inline API documentation. Comments can be annotated with @ signs to represent key metadata:

{% highlight javascript %}
/**
 * Example constructor.
 *
 * @constructor
 * @this {Example}
 */
{% endhighlight %}

Full HTML documentation can be generated using [jsdoc-toolkit](http://code.google.com/p/jsdoc-toolkit/). I've noticed that JavaScript projects are also using a project called [dox](https://github.com/visionmedia/dox) with JSDoc-formatted comments.

### Docco

![](/images/posts/docco.png)

[Docco](https://github.com/jashkenas/docco) by Jeremy Ashkenas is an MIT licensed documentation generator that generates html files with inline code based on comments, with the original source next to them. It looks like this: [jashkenas.github.com/docco](http://jashkenas.github.com/docco/). It's installable from npm with <code>npm install docco</code>.

It's a simple library that's easy to use if you're already using Node -- which we're using for Turing's build system. If you want to try it out on Turing, install docco then run this from Turing's directory: <code>docco \*.js</code>.

Because Docco presents the original source code alongside comments it's actually quite unique. In contrast, JSDoc will generate something that looks like Prototype or jQuery's documentation. I've noticed many simple Node libraries are using Docco lately.

### Ronn

[Ronn](https://github.com/rtomayko/ronn) by Ryan Tomayko and [ronnjs](https://github.com/kapouer/ronnjs) can be used to generate documentation from sets of files. With ronnjs, you're limited to markdown files. The results can be man pages or HTML. The name is a pun on [roff](http://en.wikipedia.org/wiki/Roff).

### GitHub Wikis

GitHub projects get a wiki that's actually powered by Git itself. It's possible to write them in textile, and there's a detailed blog post on the GitHub blog: [Git-backed Wikis](https://github.com/blog/699-making-github-more-open-git-backed-wikis). This is a great tool for projects that need documentation outside of the API, like FAQs, installation guidelines, and example code.

GitHub also has [GitHub Pages](https://github.com/blog/272-github-pages), which can be used to host static files at *you.github.com*. Pages could be used to host the documentation generated by a tool like Docco or dox.

### Conclusion

I'd imagine that many of you are interested in learning some JSDoc, so let's look at using [dox](https://github.com/visionmedia/dox) for our API documentation. I also think it would be useful to use [ronnjs](https://github.com/kapouer/ronnjs) to generate other documentation. This could be deployed with GitHub pages, and we could look at how to use the GitHub wiki while we're at it.
