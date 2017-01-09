---
layout: post
title: "Let's Make a Framework: Ajax Part 2"
author: Alex Young
categories: 
- frameworks
- tutorials
- ajax
- lmaf
---

Welcome to part 14 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues discussing Ajax.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### Cross-Domain Requests

Cross-domain requests are useful because they can be used to fetch data from services like Twitter. This is now a popular technique, despite feeling clunky to implement. This is another case where lack of browser features can be patched by JavaScript.

### Implementations in the Wild

The [Glow](http://www.bbc.co.uk/glow/) framework has <code>glow.net.xDomainGet</code> and <code>glow.net.loadScript</code>. These are similar, but place different requirements on the server. What <code>loadScript</code> does is called JSONP. jQuery also implements JSONP, and has this to say in the documentation:

> The jsonp type appends a query string parameter of callback=? to the URL. The server should prepend the JSON data with the callback name to form a valid JSONP response. We can specify a parameter name other than callback with the jsonp option to $.ajax().

A JSONP URL looks like this:

<code>http://feeds.delicious.com/v1/json/alex\_young/javascript?callback=parseJSON</code>

The reason a callback is specified in the URL is the cross-domain Ajax works by loading remote JavaScript by inserting <code>script</code> tags, and then interpreting the results. The term JSONP comes from *JSON with Padding*, and originated in [Remote JSON - JSONP](http://bob.pythonmac.org/archives/2005/12/05/remote-json-jsonp/) by Bob Ippolito.

### The Algorithm

JSONP works like this:

1.  The framework transparently creates a callback for this specific request
2.  A script tag is inserted into a document. This is of course invisible to the user
3.  The <code>src</code> attribute is set to <code>http://example.com/json?callback=jsonpCallback,</code>
4.  The server generates a response
5.  The server wraps its JSON response like this: <code>jsonpCallback({ ... })</code>
6.  Once the script has loaded, the callback is called, and the client code can process the JSON accordingly
7.  Finally, the callback and <code>script</code> tags are removed

As you can see, servers have to be compliant with this technique -- you can't use it to fetch arbitrary JSON or XML.

### API Design

I've based this on the Glow framework, and the option names are consistent with the other <code>turing.net</code> code:

{% highlight javascript %}
turing.net.jsonp('http://feeds.delicious.com/v1/json/alex_young/javascript?callback={callback}', {
  success: function(json) {
    console.log(json);
  }
});
{% endhighlight %}

The <code>{callback}</code> string must be specified -- it gets replaced by the framework transparently.

### Implementation

To create a callback, no clever meta-programming is required. It's sufficient to create a function and assign as a property on <code>window</code>:

{% highlight javascript %}
methodName = '__turing_jsonp_' + parseInt(new Date().getTime());
window[methodName] = function() { /* callback */ };
{% endhighlight %}

A <code>script</code> tag is created and destroyed as well:

{% highlight javascript %}
scriptTag = document.createElement('script');
scriptTag.id = methodName;

// Replacing the request URL with our internal callback wrapper
scriptTag.src = url.replace('{callback}', methodName);
document.body.appendChild(scriptTag);

// The callback should delete the script tag after the client's callback has completed
document.body.removeChild(scriptTag)
{% endhighlight %}

That's practically all there is to it. You can see the real implementation in [turing.net.js](http://github.com/alexyoung/turing.js/blob/master/turing.net.js) by searching for <code>JSONPCallback</code>.

### Conclusion

Although JSONP requires server support, a wide variety of interesting services support it. This simple technique has made cross-domain requests popular -- how many blogs and sites now feature live Twitter feeds written with pure JavaScript?
