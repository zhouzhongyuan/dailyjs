---
layout: post
title: "Let's Make a Framework: Return of the DOM"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- events
- dom
---

Welcome to part 25 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part is about handling when the DOM had loaded.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js).

### The Point

I'm taking us on a diversion this week because I'm tired of creating Turing examples without a DOM ready handler. This is a core feature of browser-based frameworks, and it's the key to unobtrusive JavaScript.

The point is to watch for the DOM to finish loading, rather than the entire document (with all of the related images and other assets). This is useful because it gives the illusion of JavaScript-related code being instantly available. If this wasn't done, JavaScript code could be evaluated after the user has started interacting with the document.

Most jQuery users use this feature without realising it's there:

{% highlight javascript %}
$(document).ready(function() {
  // Let the fun begin
});
{% endhighlight %}

Here's the core of what makes this happen:

{% highlight javascript %}
bindReady: function() {
  if ( readyBound ) {
    return;
  }

  readyBound = true;

  // Catch cases where $(document).ready() is called after the
  // browser event has already occurred.
  if ( document.readyState === "complete" ) {
    return jQuery.ready();
  }

  // Mozilla, Opera and webkit nightlies currently support this event
  if ( document.addEventListener ) {
    // Use the handy event callback
    document.addEventListener( "DOMContentLoaded", DOMContentLoaded, false );

    // A fallback to window.onload, that will always work
    window.addEventListener( "load", jQuery.ready, false );

  // If IE event model is used
  } else if ( document.attachEvent ) {
    // ensure firing before onload,
    // maybe late but safe also for iframes
    document.attachEvent("onreadystatechange", DOMContentLoaded);

    // A fallback to window.onload, that will always work
    window.attachEvent( "onload", jQuery.ready );

    // If IE and not a frame
    // continually check to see if the document is ready
    var toplevel = false;

    try {
      toplevel = window.frameElement == null;
    } catch(e) {}

    if ( document.documentElement.doScroll && toplevel ) {
      doScrollCheck();
    }
  }
}
{% endhighlight %}

Prototype and other frameworks have similar code. It's not surprising that Prototype's DOM loaded handling references Resig and other prominent developers (Dan Webb, Matthias Miller, Dean Edwards, John Resig, and Diego Perini). <code>jQuery.ready</code> gets called through either a modern <code>DOMContentLoaded</code> event, or <code>onload</code> events.

I like the way Prototype fires a custom event when the document is ready. Prototype uses a colon to denote a custom event, because these have to be handled differently by IE -- <code>element.fireEvent('something else', event)</code> causes an argument error. I tried to duplicate that in Turing, but it'd take a fair amount of work to adapt Turing's current event handling so I left it out for now and used an array of callbacks instead.

Setting up multiple observer callbacks will work:

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
  <style>p { color:red; }</style>
  <script src="http://code.jquery.com/jquery-latest.min.js"></script>
  <script>
  $(document).ready(function() {
    $("#a").text("A.");
  });
  
  $(document).ready(function() {
    $("#b").text("B.");
  });
  
  $(document).ready(function() {
    $("#c").text("C.");
  });
  </script>

</head>
<body>
  <p id="a"></p>
  <p id="b"></p>
  <p id="c"></p>
</body>
</html>
{% endhighlight %}

### Our API

I'm shooting for this:

{% highlight javascript %}
turing.events.ready(function() {
  // The DOM is ready, but some other stuff might be
});
{% endhighlight %}

### Implementing "onready"

I've based the current code on jQuery. It's just enough to work cross-browser and do what I need it to do. It's all handled by private methods and variables. The last thing to get called is <code>ready()</code>:

{% highlight javascript %}
function ready() {
  if (!isReady) {
    // Make sure body exists
    if (!document.body) {
      return setTimeout(ready, 13);
    }

    isReady = true;

    for (var i in readyCallbacks) {
      readyCallbacks[i]();
    }

    readyCallbacks = null;
  }
}
{% endhighlight %}

This is just a tiny snippet, but to summarise the rest of the code:

1.  The public method, <code>turing.events.ready</code> calls <code>bindOnReady</code>
2.  If <code>bindOnReady</code> has already been called once, it will return
3.  This method sets up <code>DOMContentLoaded</code> as an event, and will fall over to simply call <code>ready()</code>
4.  The "doScroll check" is used for IE through <code>DOMReadyScrollCheck</code>, which also calls <code>ready()</code> when it's done
5.  <code>setTimeout</code> and recursive calls to <code>DOMReadyScrollCheck</code> make this happen

The <code>isReady</code> variable is bound to the closure for <code>turing.events</code>, so it's safely tucked away where we need it. The rest of the code is similar to jQuery's -- after going through the tickets referenced in the code comments relating to IE problems, I didn't have enough time to create my own truly original version.

### Something from the Archives

If you're interested, I previously used the following code which I created (based on various blog posts and other bits of research) about 4 years ago:

{% highlight javascript %}
function init(run) {
  // quit if this function has already been called
  if (arguments.callee.done) return;

  // flag this function so we don't do the same thing twice
  arguments.callee.done = true;

  // kill the timer
  if (_timer) clearInterval(_timer);

  // do stuff
  run();
}

/* for Mozilla/Opera9 */
if (document.addEventListener) {
  document.addEventListener("DOMContentLoaded", init, false);
}

/* for Internet Explorer */
/*@cc_on @*/
/*@if (@_win32)
  document.write("<script id=__ie_onload defer src=javascript:void(0)><\/script>")
  var script = document.getElementById("__ie_onload")
  
  script.onreadystatechange = function() {
    if (this.readyState == "complete") {
      init() // call the onload handler
    }
  }
/*@end @*/

/* for Safari */
if (/WebKit/i.test(navigator.userAgent)) {
  var _timer = setInterval(function() {
    if (/loaded|complete/.test(document.readyState)) {
      init(); // call the onload handler
    }
  }, 10);
}

/* for other browsers */
window.onload = init;
{% endhighlight %}

As you can see, the concept of a loaded DOM is a messy thing to deal with. It's not really the fault of any one browser manufacturer, it's just growing pains as a consequence of JavaScript evolving through the DOM levels.

I had this old code kicking around for years, and I was convinced the whole DOM loaded thing was over. But last year Dean Edwards wrote [Callbacks vs Events](http://dean.edwards.name/weblog/2009/03/callbacks-vs-events/) on the topic. In particular he deals with handling <code>DOMContentLoaded</code> through custom events, and the article has some interesting comments.

### Conclusion

The current version of <code>turing.dom.ready</code> isn't exactly what I wanted, and I had to cut short some features due to time constraints (I spent about 4 hours on this article and had to call it a day). The code is here: [turing.dom.js](http://github.com/alexyoung/turing.js/blob/master/turing.dom.js) -- if you can help simplify or improve it, I'd appreciate it! Maybe it could be a guest post?
