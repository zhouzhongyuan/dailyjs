---
layout: post
title: "Let's Make a Framework: Aliasing and Packaging"
author: Alex Young
categories: 
- frameworks
- tutorials
- jake
- narwhal
- lmaf
---

Welcome to part 12 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part discusses aliasing and packaging.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

### A User-Friendly API

Turing uses namespaced modules and methods, so calls look like this:

{% highlight javascript %}
turing.dom.get('#selector');
{% endhighlight %}

The reason Turing is designed this way is it allows it to be modular -- you don't need to load every module, which is great for server-side work.

Lots of frameworks use a dollar sign, so we could shorten Turing to <code>$t</code>:

{% highlight javascript %}
$t.dom.get('#selector');
{% endhighlight %}

That's better, but <code>get</code> is one of those operations that will be commonly used for browser-based work. Let's take a leaf out of jQuery's book and accept a parameter. If the parameter is a string, it will be considered a selector:

{% highlight javascript %}
$t('#selector');
{% endhighlight %}

This can be implemented using <code>arguments</code>:

{% highlight javascript %}
$t = (function() {
  var alias = function() {
    return turing.dom.get(arguments[0]);
  }

  return alias;
})();
{% endhighlight %}

Then other parts of the framework can be referenced by the <code>alias</code> object:

{% highlight javascript %}
if (turing.dom) {
  alias.dom = turing.dom;
}

if (turing.events) {
  alias.events = turing.events;
}
{% endhighlight %}

### Enumerable Shortcuts

The <code>turing.enumerable</code> module contains a big set of commonly used functions. Rather than retaining the namespace, let's copy those straight into the alias object:

{% highlight javascript %}
if (turing.enumerable) {
  turing.enumerable.each(turing.enumerable, function(fn, method) {
    alias[method] = fn;
  });
}
{% endhighlight %}

Now using <code>each</code> is much easier: <code>$t.each()</code>.

### Configuration

[BBC's Glow framework](http://www.bbc.co.uk/glow/) is interesting because it can work alongside different versions of itself:

> Also, as many BBC pages are built by separate development teams, we needed a library that could work alongside different versions of itself, without encountering namespace issues or CSS clashes. None of the major libraries were capable of doing this.

I like this idea, and I wish more frameworks used careful namespacing and supported concurrent use. One way to support this is to use a loading system that can load the framework and its modules. We're not quite there yet, so I decided to allow Turing's global alias to be changed as an experiment.

At the moment this works using an <code>eval</code> to fetch the framework's alias from the configuration, but this could be changed in the future:

{% highlight javascript %}
eval(turing.alias + ' = turing.aliasFramework()');  
{% endhighlight %}

In <code>turing.core.js</code> I set some high-level configuration options. This is where I've stored a reference to <code>'$t'</code>.

### Packaging and Minification

I've carefully handcrafted Turing to work minified. Concatenating scripts and running a minifier is boring though, so let's script it.

To be fully JavaScript and awesome I'm using [Jake](http://github.com/280north/jake). It's a JavaScript build tool that's easy to work with.

The first thing you need to do is get [Narwhal](http://narwhaljs.org/). Installation is pretty easy:

{% highlight sh %}
git clone git://github.com/280north/narwhal.git
export PATH=$PATH:~/narwhal/bin

# Check it works
narwhal narwhal/examples/hello

# If that didn't work, check that you don't have any Rhino jars kicking around

# Next, install jake and shrinksafe
tusk install jake
tusk install shrinksafe
{% endhighlight %}

With those tools installed we can build Turing into a single file. Jake build scripts are standard JavaScript, usually named <code>Jakefile</code>:

{% highlight javascript %}
var jake = require('jake'),
    shrinksafe = require('minify/shrinksafe'),
    FILE = require('file'),
    SYSTEM = require('system')

jake.task('concat', function(t) {
  var output = '',
      files = ('turing.core.js turing.oo.js turing.enumerable.js '
              + 'turing.functional.js turing.dom.js turing.events.js turing.alias.js').split(' ')

  files.map(function(file) {
    output += FILE.read(file)
  })

  if (!FILE.exists('build')) {
    FILE.mkdir('build')
  }

  FILE.write(FILE.join('build', 'turing.js'), output)
})

jake.task('minify', function(t) {
  var shrunk = shrinksafe.compress(FILE.read(FILE.join('build', 'turing.js')))
  FILE.write(FILE.join('build', 'turing.min.js'), shrunk)
})

jake.task('build', ['concat', 'minify'])
{% endhighlight %}

The task at the end defines the build task which we'll run with <code>jake build</code>. I've used the convention of capitalising libraries like <code>FILE</code> so they don't clash with common variable names. I've hardcoded the Turing source files rather than globbing them because I want to specify the order they appear.

Running <code>jake build</code> will generate <code>build/turing.js</code> and <code>build/turing.min.js</code>.

### Conclusion

Designing good APIs isn't just about avoiding globals, it's also about using concise yet clear naming conventions. Frameworks that use <code>$()</code> really went a long way to making JavaScript development more accessible.

Most mature projects end up with some kind of build chain. Don't rush to add one to your project, but if you notice you're spending a lot of time packaging things, try writing your own Jakefile. You can technically script anything with tools like Jake, just keep an eye out for repeated actions during development.
