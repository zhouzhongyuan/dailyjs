---
layout: post
title: "Let's Make a Framework: IE Testing"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- ie
---

Welcome to part 45 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

### IE Testing

The way I usually test my projects in IE is with [IETester](http://my-debugbar.com/wiki/IETester/HomePage). I actually run this in a virtual machine, just to be safe. Some of my clients still require IE6, some are on 7, and I usually test in 8 and 9 as well. Supporting IE6 is usually the most difficult thing to do, but I usually find JavaScript debugging less painful than design glitches.

I opened IETester and tried running Turing's tests in IE6 and found they didn't run. That's... just life, isn't it?

To focus a little bit more I tried a simple unit test instead of the whole set. The [alias\_test.html](https://github.com/alexyoung/turing.js/blob/master/test/alias_test.html) seemed like a simple one. It produced this error:

![](/images/posts/ie_alias_test.png)

It's probably something to do with the slightly ambitious CommonJS module style test system we created. Note that you can press the escape key to quickly close these IE popup Windows. I tried removing the *alias\_test.js* script and the error didn't appear, which means there isn't a problem parsing the files that the test depends on.

After commenting out the code in the test itself, I figured out <code>require</code> was failing somewhere. I used the IETester debug tools to run <code>require('')</code> and the same error popped up. The problem was here, in the Turing Test library:

{% highlight javascript %}
head[0].insertBefore(scriptTag, head.firstChild);
{% endhighlight %}

### IE Script Loading

At this point I found IE wouldn't attach <code>load</code> events to script tags. I dimly recall something about this from [Dean Edwards' posts on window.onload](http://dean.edwards.name/weblog/2006/06/again/).

This is the kind of issue we had to deal with until the post web 2.0 revolution, but I came up with a scheme to use <code>setInterval</code> to watch for changes on <code>readyState</code> on script tags. This will work in IE, but not in anything else. That gave rise to this:

{% highlight javascript %}
function loadIEScript() {
  var id = '__id_' + (new Date()).valueOf(),
      timer,
      scriptTag;
  document.write('<script id="' + id + '" type="text/javascript" src="' + script + '"></script>');
  scriptTag = document.getElementById(id);

  timer = setInterval(function() {
    if (/loaded|complete/.test(scriptTag.readyState)) {
      clearInterval(timer);
      tt.doneLoading();
    }
  }, 10);
}

function loadOtherScript() {
  var scriptTag = document.createElement('script'),
      head = document.getElementsByTagName('head');
  scriptTag.onload = function() { tt.doneLoading(); };
  scriptTag.setAttribute('type', 'text/javascript');
  scriptTag.setAttribute('src', script);
  head[0].insertBefore(scriptTag, head.firstChild);
}

this.loading();
if (document.attachEvent) {
  loadIEScript();
} else {
  loadOtherScript();
}
{% endhighlight %}

The IE code uses <code>document.write</code> to insert a script tag with a unique ID, then it watches for changes to the <code>readyState</code> attribute. Confused? Well, it's not too difficult to follow this code, but it would be nicer if we didn't need it.

Here's the proof it works!

![](/images/posts/ie_working.png)

If you're interested in this area, [RequireJS](http://requirejs.org/) is shaping up to be a good library.

### Conclusion

The single unit tests can now actually run in IE, even IE6. There's still a lot left to do to get full cross-browser support into Turing, but getting the tests going is an important first step.

To get this update, make sure you run <code>git submodule update</code> to get the latest version of the [Turing Test](https://github.com/alexyoung/turing-test.js) project.

### References

-   [IETester](http://my-debugbar.com/wiki/IETester/HomePage)
-   [window.onload (again)](http://dean.edwards.name/weblog/2006/06/again/) by Dean Edwards
-   [The window.onload Problem – Solved!](http://dean.edwards.name/weblog/2005/09/busted/) by Dean Edwards
-   [RequireJS](http://requirejs.org/)
