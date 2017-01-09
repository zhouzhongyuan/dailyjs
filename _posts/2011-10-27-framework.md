---
layout: post
title: "Asynchronous Resource Loading Part 5"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- network
---

<div class="intro">
*Let's Make a Framework* is an ongoing series about building a JavaScript framework from the ground up.

These articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js). Documentation is available at [turingjs.com](http://turingjs.com/).

</div>
Previous parts:

-   [Part 1: Introduction, library review](http://dailyjs.com/2011/09/22/framework-81/)
-   [Part 2: Script Insertion](http://dailyjs.com/2011/09/29/framework-82/)
-   [Part 3: HTML5](http://dailyjs.com/2011/10/13/framework/)
-   [Part 4: Preloading with XMLHttpRequest](http://dailyjs.com/2011/10/20/framework/)

### Creating a Preloading API

Last week I fleshed out <code>turing.request</code> so it can handle preloading with <code>XMLHttpRequest</code>. It's time to look at how to build an API that will allow scripts to load based on the required order.

I thought a lot about this and decided that an array is a good way to model dependencies between scripts. Given this array:

{% highlight javascript %}
[
  'framework.js',
  ['plugin.js', 'plugin2.js'],
  'app.js']
]
{% endhighlight %}

then the following actions should occur:

-   <code>framework.js</code> is downloaded and executed
-   <code>plugin.js</code> and <code>plugin2.js</code> are downloaded asynchronously, but only executed when <code>framework.js</code> is ready
-   Finally, <code>app.js</code> is executed

These scripts can be loaded at any time, as long as they're executed in the specified order. If the scripts are local, they can be preloaded with <code>XMLHttpRequest</code>.

### Preloading Test

To break this down into something implementable, I created a test first:

{% highlight javascript %}
'test async queue loading': function() {
  $t.require([
    '/load-me.js?test9=9',
    ['/load-me.js?test4=4', '/load-me.js?test5=5'],
    ['/load-me.js?test6=6', '/load-me.js?test7=7'],
    '/load-me.js?test8=8'
  ]).on('complete', function() {
    assert.ok(true);
  }).on('loaded', function(item) {
    if (item.src === '/load-me.js?test9=9') {
      assert.equal(typeof test4, 'undefined');
    }

    if (item.src === '/load-me.js?test4=4') {
      assert.equal(test9, 9);
      assert.equal(test4, 4);
      assert.equal(typeof test6, 'undefined');
      assert.equal(typeof test8, 'undefined');
    }

    if (item.src === '/load-me.js?test6=6') {
      assert.equal(test9, 9);
      assert.equal(test4, 4);
      assert.equal(test6, 6);
      assert.equal(typeof test8, 'undefined');
    }
  });
{% endhighlight %}

I originally wrote this with a callback like the old <code>require</code> signature implied, but I realised that different types of callbacks are required, so it felt natural to use something based on events.

Using this event-based approach should make it easier to build internally, but also makes a pretty rich API.

### Modifying <code>require</code>

To work with this new signature (and event-based API), last week's <code>require</code> implementation will have to be updated. Recall that a <code>setTimeout</code> method was used to delay loading until the <code>head</code> part of the document is ready. That makes returning a value -- and thus chaining off <code>require</code> -- impossible.

To get around this, I extracted the <code>setTimeout</code> part and made it into a method:

{% highlight javascript %}
function runWhenReady(fn) {
  setTimeout(function() {
    if ('item' in appendTo) {
      if (!appendTo[0]) {
        return setTimeout(arguments.callee, 25);
      }

      appendTo = appendTo[0];
    }

    fn();
  });
}
{% endhighlight %}

When an array is passed to <code>require</code> we can safely assume queuing is necessary. The old method signature is still available. In the case when an array has been passed, a <code>Queue</code> object can be returned to enable chaining:

{% highlight javascript %}
turing.require = function(scriptSrc, options, fn) {
  options = options || {};
  fn = fn || function() {};

  if (turing.isArray(scriptSrc)) {
    return new Queue(scriptSrc);
  }

  runWhenReady(function() {
    // Last week's code follows
{% endhighlight %}

### The <code>Queue</code> Class

The <code>Queue</code> class is based on <code>turing.events.Emitter</code> which is a simple event management class, created in [Let's Make a Framework: Custom Events](http://dailyjs.com/2011/07/07/framework-70/). The basic algorithm works like this:

1.  <code>Queue.prototype.parseQueue</code>: Iterate over each item in the array to work out which scripts require preloading, and group them together based on the array groupings. Single script items will be added to a group that contains one item, so the array of arrays becomes normalised into something easy to manage.
2.  <code>Queue.prototype.enqueue</code>: Whenever <code>parseQueue</code> finds a script, call <code>enqueue</code> to add it to a sequentially indexed object.
3.  <code>runQueue</code>: Iterate over each group and preload each local script using <code>XMLHttpRequest</code>. Add an event emitter to the <code>XMLHttpRequest</code> callback to signal a <code>preload</code> event has completed.
4.  <code>preload</code> event: When this event is fired, mark the group item as 'preloaded' and start executing scripts if the whole group has finished preloading.
5.  <code>complete</code> event: This event is fired when all items have executed.

I'll explain these methods and events below.

### Initialization <code>Queue</code> and Maintaining the Chain

To allow the <code>on</code> methods to be called after <code>turing.require</code>, <code>Queue</code> has to use <code>runWhenReady</code> itself, and proxy calls to <code>turing.events.Emitter</code> and return <code>this</code>:

{% highlight javascript %}
function Queue(sources) {
  this.sources = sources;
  this.events = new turing.events.Emitter();
  this.queue = [];
  this.currentGroup = 0;
  this.groups = {};
  this.groupKeys = [];
  this.parseQueue(this.sources, false, 0);

  this.installEventHandlers();
  this.pointer = 0;

  var self = this;
  runWhenReady(function() {
    self.runQueue();
  });
}

Queue.prototype = {
  on: function() {
    this.events.on.apply(this.events, arguments);
    return this;
  },

  emit: function() {
    this.events.emit.apply(this.events, arguments);
    return this;
  },

  // ...
{% endhighlight %}

### Parsing the Queue

The array of script sources must be parsed into something that's easy to execute sequentially. The <code>enqueue</code> method helps sort items into groups, and flags if they're suitable for preloading:

{% highlight javascript %}
enqueue: function(source, async) {
  var preload = isSameOrigin(source),
      options;

  options = {
    src: source,
    preload: preload,
    async: async,
    group: this.currentGroup
  };

  if (!this.groups[this.currentGroup]) {
    this.groups[this.currentGroup] = [];
    this.groupKeys.push(this.currentGroup);
  }

  this.groups[this.currentGroup].push(options);
},
{% endhighlight %}

The actual job of queuing is fairly simple, but care must be taken to increment the <code>currentGroup</code> counter as groups are added:

{% highlight javascript %}
parseQueue: function(sources, async, level) {
  var i, source;
  for (i = 0; i < sources.length; i++) {
    source = sources[i];
    if (turing.isArray(source)) {
      this.currentGroup++;
      this.parseQueue(source, true, level + 1);
    } else {
      if (level === 0) {
        this.currentGroup++;
      }
      this.enqueue(source, async);
    }
  }
},
{% endhighlight %}

The reason the <code>level</code> variable is used is to differentiate between grouped items and single scripts.

### Running the Queue

Each script item is iterated over in sequence, and <code>XMLHttpRequest</code> is used to preload scripts:

{% highlight javascript %}
runQueue: function() {
  var i, g, group, item, self = this;

  for (g = 0; g < this.groupKeys.length; g++) {
    group = this.groups[this.groupKeys[g]];
    
    for (i = 0; i < group.length; i++ ) {
      item = group[i];

      if (item.preload) {
        (function(groupItem) {
          requireWithXMLHttpRequest(groupItem.src, {}, function(script) {
            self.emit('preloaded', groupItem, script);
          })
        }(item));
      }
    }
  }
}
{% endhighlight %}

The anonymous function wrapper helps keep the right <code>item</code> around (because I've used <code>for</code> loops instead of a callback-based iterator).

As each request comes back and <code>preloaded</code> is fired, the event handler will check to see if the entire group has been loaded, and if so, execute each item.

### Conclusion

When I had the idea to use events to manage script loading, I thought I was onto something and the code would be very simple. However, the <code>Queue</code> class I wrote for this tutorial ended up becoming quite complex, and it only handles local scripts at this stage.

I'll attempt to add support for remote preloading as well (where available), and also add support for script tag insertion as a last resort.

This week's code is in [commit 74a0f7f](https://github.com/alexyoung/turing.js/commit/74a0f7f27290b088664ac2bf8159f61d8468be6f). If you've got any feedback, post a comment (or fork) and I'll see if I can incorporate it.
