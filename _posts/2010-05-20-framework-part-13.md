---
layout: post
title: "Let's Make a Framework: Ajax"
author: Alex Young
categories: 
- frameworks
- tutorials
- ajax
- lmaf
---

Welcome to part 13 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part discusses Ajax.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### XMLHttpRequest

All XMLHttpRequest really does is allows browsers to send messages *back* to the server. Combined with DOM manipulation, this allows us to update or replace parts of a page. This simple API marked a radical change in web application development, and rapidly became commonplace.

Requests must adhere to the *same origin policy*. This means a request has to call a server at a given domain. This limitation can be side-stepped by using dynamically inserted script tags, image tags, and iframes.

### History

XMLHttpRequest is another technology that one player in the browser war pioneered, while the others had to catch up. What's interesting about XMLHttpRequest is Microsoft created the concept behind it -- not Mozilla or the W3C. Although ActiveX was used, equivalent functionality shipped with Internet Explorer 5.0 in 1999. It wasn't until 2006 that the W3C published a working draft for XMLHttpRequest itself.

### Request Objects

In this series of tutorials I'm going to focus on XMLHttpRequest as well as other network-related techniques. Following [Glow's](http://www.bbc.co.uk/glow/docs/articles/what_is_glow.shtml) lead, I'll use the namespace <code>turing.net</code>.

Frameworks like jQuery make XMLHttpRequest look incredibly simple, but there's actually a fair bit going on behind the scenes:

1.  Differences between Microsoft's implementation and W3C have to be dealt with
2.  Request headers must be set for the type of data and HTTP methpd
3.  State changes must be handled through callbacks
4.  Browser bugs must be compensated for

Creating a request object looks like this:

{% highlight javascript %}
// W3C compliant
new XMLHttpRequest();
// Microsoft
new ActiveXObject("Msxml2.XMLHTTP.6.0");
new ActiveXObject("Msxml2.XMLHTTP.3.0");
new ActiveXObject('Msxml2.XMLHTTP');
{% endhighlight %}

Internet Explorer has different versions of MSXML -- these are the preferred versions based on [Using the right version of MSXML in Internet Explorer](http://blogs.msdn.com/xmlteam/archive/2006/10/23/using-the-right-version-of-msxml-in-internet-explorer.aspx).

jQuery's implementation looks like this:

{% highlight javascript %}
xhr: window.XMLHttpRequest && (window.location.protocol !== "file:" || !window.ActiveXObject) ?
  function() {
    return new window.XMLHttpRequest();
  } :
  function() {
    try {
      return new window.ActiveXObject("Microsoft.XMLHTTP");
    } catch(e) {}
  },
{% endhighlight %}

This code uses a ternary operator to find a valid transport. It also has a note about preventing IE7 from using XMLHttpRequest, because it can't load local files.

More explicitly, finding the right request object looks like this:

{% highlight javascript %}
function xhr() {
  if (typeof XMLHttpRequest !== 'undefined' && (window.location.protocol !== 'file:' || !window.ActiveXObject)) {
    return new XMLHttpRequest();
  } else {
    try {
      return new ActiveXObject('Msxml2.XMLHTTP.6.0');
    } catch(e) { }
    try {
      return new ActiveXObject('Msxml2.XMLHTTP.3.0');
    } catch(e) { }
    try {
      return new ActiveXObject('Msxml2.XMLHTTP');
    } catch(e) { }
  }
  return false;
}
{% endhighlight %}

### Sending Requests

There are three main parts to sending a request:

1.  Set a callback for <code>onreadystatechange</code>
2.  Call <code>open</code> on the request object
3.  Set the request headers
4.  Call <code>send</code> on the object

The <code>onreadystatechange</code> callback can be used to call user-supplied callbacks for request success and failure. The following states may trigger this callback:

-   0: *Uninitialized*: </code>open</code> has not been called
-   1: *Loading*: <code>send</code> has not been called
-   2: *Loaded*: <code>send</code> has been called, headers and status are available
-   3: *Interactive*: Downloading, <code>responseText</code> holds the partial data
-   4: *Completed*: Request finished

The <code>open</code> function is used to initialize the HTTP method, set the URL, and determine if the request should be asynchronous. Request headers can be set with <code>request.setRequestHeader(header, value)</code>. The post body can be set with <code>send(postBody)</code>.

This simple implementation uses the previously defined <code>xhr</code> function to send requests:

{% highlight javascript %}
function ajax(url, options) {
  function successfulRequest(request) {
    return (request.status >= 200 && request.status < 300) ||
        request.status == 304 ||
				(request.status == 0 && request.responseText);
  }

  function respondToReadyState(readyState) {
    if (request.readyState == 4 ) {
      if (successfulRequest(request)) {
        if (options.success) {
          options.success(request);
        }
      } else {
        if (options.failure) {
          options.failure();
        }
      }
    }
  }

  function setHeaders() {
    var headers = {
      'Accept': 'text/javascript, text/html, application/xml, text/xml, */*'
    };

    for (var name in headers) {
      request.setRequestHeader(name, headers[name]);
    }
  }

  var request = xhr();
  if (typeof options === 'undefined') {
    options = {};
  }

  options.method = options.method ? options.method.toLowerCase() : 'get';
  options.asynchronous = options.asynchronous || true;
  options.postBody = options.postBody || '';

  request.onreadystatechange = respondToReadyState;
  request.open(options.method, url, options.asynchronous);
  setHeaders();
  request.send(options.postBody);
}
{% endhighlight %}

### Popular APIs

[jQuery](http://api.jquery.com/jQuery.ajax/) uses methods like <code>get</code> and <code>post</code>. These are shorthands for the <code>ajax</code> function:

{% highlight javascript %}
$.ajax({
  url: url,
  data: data,
  success: success,
  dataType: dataType
});
{% endhighlight %}

Prototype uses classes -- <code>Ajax.Request</code> is the main one. Typical usage looks like this:

{% highlight javascript %}
new Ajax.Request('/your/url', {
  onSuccess: function(response) {
    // Handle the response content...
  }
});
{% endhighlight %}

Glow framework 1.7 is interesting because it fires a custom event when an event succeeds or fails:

{% highlight javascript %}
if (response.wasSuccessful) {
  events.fire(request, "load", response);
} else {
  events.fire(request, "error", response);
}
{% endhighlight %}

### Putting it Together

Using the examples above, I've built <code>turing.net</code> which implements <code>get</code>, <code>post</code>, and <code>ajax</code>:

{% highlight javascript %}
turing.net.get('/example', { success: function(r) { alert(r.responseText); } });
{% endhighlight %}

It's on GitHub now: [turing.net.js](http://github.com/alexyoung/turing.js/blob/master/turing.net.js)

### Next Week

Next week I'll continue expanding on the basics created here, and demonstrate how to implement cross-domain requests.

### References

-   [Glow](http://www.bbc.co.uk/glow/)
-   [jQuery/ajax.js](http://github.com/jquery/jquery/blob/master/src/ajax.js)
-   [The history of XMLHttpRequest](http://en.wikipedia.org/wiki/XMLHttpRequest)
-   [Using the right version of MSXML in Internet Explorer](http://blogs.msdn.com/xmlteam/archive/2006/10/23/using-the-right-version-of-msxml-in-internet-explorer.aspx)
-   [XMLHTTP notes: readyState and the events](http://www.quirksmode.org/blog/archives/2005/09/xmlhttp_notes_r_2.html)
