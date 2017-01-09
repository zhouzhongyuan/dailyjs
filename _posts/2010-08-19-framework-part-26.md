---
layout: post
title: "Let's Make a Framework: Feedback"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
---

Welcome to part 26 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

I received some feedback last week about Turing -- very useful and interesting points that I thought followers of the series would enjoy.

### Alias Library Eval Usage

[Øyvind Sean Kinsey](http://github.com/oyvindkinsey) commented on one of my commits to say that he thought the way Turing creates a global alias was unnecessary. My code looked like this:

{% highlight javascript %}
eval(turing.alias + ' = turing.aliasFramework()');
{% endhighlight %}

The reason I did this was so <code>turing.alias</code> would expand to the variable name. This was so people using Turing could alias it to a short name, like jQuery's <code>$</code>.

<code>turing.core</code> stores a reference to the global context, which is generally <code>window</code>. As I like to write unit tests outside a browser I pass in an object rather than relying on <code>window</code>. Because the old version of <code>turing.core</code> didn't expose this global, I used <code>eval</code> to create a similar effect.

Taking on board Øyvind's feedback, I've added a method called <code>exportAlias</code> to the core library, and changed <code>eval</code> to:

{% highlight javascript %}
turing.exportAlias(turing.alias, turing.aliasFramework);
{% endhighlight %}

A lot of libraries and frameworks set properties on <code>window</code> in a similar way.

### DOM Ready Not Required

After all the work I put into creating a DOM ready implementation, it wasn't really required! It's possible to insert <code>

<script>
</code> tags at the end of a document to get a similar effect -- this is actually recommended for performance reasons by people like Yahoo! and Google.

[Arnout Kazemier](http://twitter.com/3rdEden) let me know that all my hard work and the source of my steadily worsening RSI was worthless, so thanks Arnout!

In all seriousness, it's useful to be aware of this alternative. Arnout even provided an example that clearly backs up the performance claims:

> [Here's a good example](http://stevesouders.com/cuzillion/?c0=hj1hfff3_1_f&t=1281690264262) of blocking rendering with a script. The script in the head takes 3 seconds to load and 1 second to execute. You will see a white page for 4 seconds because it blocks the rendering of the page because it's loaded in the head. And the browser can't continue because the contents might affect how the rest of page needs to be rendered.

I enjoyed the simplicity of this approach versus dozens of lines of JavaScript. And as far as this tutorial series is concerned, it's good to know when frameworks are doing a lot of heavy lifting for features that aren't always required.
