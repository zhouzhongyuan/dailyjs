---
layout: post
title: "Let's Make a Framework: Node Packaging"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- node
- packaging
---

Welcome to part 32 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

[Last week](http://dailyjs.com/2010/09/23/framework-part-31/) I wrote about building an event delegation chaining API. This week I'm going to talk about packaging.

### Packaging with Node

In [part 12](http://dailyjs.com/2010/05/13/framework-part-12/) I talked about using [Narwhal](http://narwhaljs.org/) to create a *Jakefile* to build and minimise Turing. I thought it would be interesting to look at doing the same thing with Node, because I know a lot of DailyJS's readers already have Node installed.

I'm using [node-jake](http://github.com/mde/node-jake), available from npm with <code>npm install jake</code>.

### Detecting Node or Rhino

At the moment Narwhal uses Rhino. Narwhal and Node have slightly different globals available, so I'm using these to detect the active environment:

{% highlight javascript %}
if (typeof global.system === 'undefined') {
  nodeTasks();
} else {
  narwhalTasks();
}
{% endhighlight %}

If you're writing your own build tasks I strongly recommend selecting a single build tool (or perhaps one to suit each major platform). My <code>Jakefile</code> only caters for both to support these tutorials, it's actually rather impractical.

### File Concatenation

I've made the <code>concat</code> task synchronous to make it easier to read:

{% highlight javascript %}
task('concat', [], function () {
  var files = ('turing.core.js turing.oo.js turing.enumerable.js '
              + 'turing.functional.js turing.dom.js turing.events.js '
              + 'turing.touch.js turing.alias.js turing.anim.js').split(' '),
      filesLeft = files.length,
      pathName = '.',
      outFile = fs.openSync('build/turing.js', 'w+');

  files.forEach(function(fileName) {
    var fileName = path.join(pathName, fileName),
        contents = fs.readFileSync(fileName);
    sys.puts('Read: ' + contents.length + ', written: ' + fs.writeSync(outFile, contents.toString()));
  });
  fs.closeSync(outFile);    
});
{% endhighlight %}

The output file is opened first, then each file is read into it.

### Minifier

There's also a npm package for minimising JavaScript, called [node-jsmin](http://github.com/pkrumins/node-jsmin). It's pretty easy to use:

{% highlight javascript %}
var jsmin = require('jsmin').jsmin;
jsmin(code);
{% endhighlight %}

This is based on Crockford's original script. You might get better results by pasting Turing into [Google Closure](http://code.google.com/closure/) or Dean Edwards' [Packer](http://dean.edwards.name/packer/).

For now I'm going to stub this task out until there's a better minifier.

I haven't found any particularly strong minifiers for Node yet. jQuery actually uses Google's compiler, as a Jar file. That means they can also run jslint as well, which is useful. These could be run in our Jakefile simply by shelling out. It's also interesting to note that jQuery will build with unix <code>make</code>, Ruby's <code>rake</code> and <code>ant</code> (which makes life easier for Windows users).

### Organising Tasks

node-jake allows you to run a list of tasks before the current one:

{% highlight javascript %}
desc('Main build task');
task('build', ['concat', 'minify'], function() {});
{% endhighlight %}

This is equivalent to the old Narwhal build task:

{% highlight javascript %}
jake.task('build', ['concat', 'minify'])
{% endhighlight %}

280 North's Jake is clever enough to realise a function hasn't been specified, whereas the current version of node-jake will crash if you run: <code>task('build', \['concat', 'minify'\]);</code>

### Running Tasks

After installing node-jake, run it with <code>jake -T</code> to see a list of tasks. To build the project, run <code>jake build</code>.

A default task could be added (called <code>default</code>) but I'd like to use that to generate documentation and I haven't picked a JavaScript-friendly documentation system to do that yet.

### Conclusion

Both node-jake and 280 North's Jake are largely compatible, and are good choices for your own project. If you're working on a node or Narwhal project, the choice is obvious, but when you're building a web-based library it's really up to you.
