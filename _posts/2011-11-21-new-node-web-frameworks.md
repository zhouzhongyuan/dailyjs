---
layout: post
title: "New Node Web Frameworks: Derby, Flatiron"
author: Alex Young
categories: 
- node
- modules
---

### Derby

[Derby](http://derbyjs.com/) (GitHub: [codeparty / derby](https://github.com/codeparty/derby), License: *MIT*, npm: *derby*), other than being the city closest to where I grew up, is a new web framework for Node by Nate Smith and Brian Noguchi. It features a data synchronisation library called [Racer](https://github.com/codeparty/racer) that synchronises data between browsers, servers, and the database. This is one of the few libraries to truly realise the goal of running the same code on clients and servers. This data synchronisation layer is far from simple -- it even includes features to help resolve conflicts.

Derby isn't just a solution for data syncing, however. It also renders templates, and handles view bindings. Derby uses a single page application structure, so the same routes are used on both the client and the server.

Applications are created using a command-line tool, much like other popular web frameworks:

{% highlight sh %}
derby new dailyjs
{% endhighlight %}

Derby's template language may take some getting used to, but it's very close to [mustache](http://mustache.github.com/). It also includes HTML extensions to make it easier to bind models and views without lots of client-side "glue" code:

{% highlight html %}
<info:>
  <div id=info>
    <!--
      Mustache syntax is used for conditional blocks, except that the
      name need not be repeated in the closing tag or if-else blocks
    -->
    ((^connected))
      ((#canConnect))
        <!-- Leading space is removed, and trailing space is maintained -->
        Offline 
        ((#_showReconnect))&ndash; <a x-bind=click:connect>Reconnect</a>((/))
      ((^))
        Unable to reconnect &ndash; <a x-bind=click:reload>Reload</a>
      ((/))
    ((/))
  </div>
{% endhighlight %}

In terms of models, Racer uses Redis for publish/subscribe and to store a journal of transactions; MongoDB is used as the data store. Racer is currently structured so that there are database adapters, so hopefully it shouldn't be too hard to add support for more databases. It seems like the Redis dependency is more central to the design of Racer.

The motivation behind Derby is to offer flexibility while removing the requirement for what the authors call "glue code". If you've ever worked with Express, Backbone, jQuery, and Mongoose, you've probably found yourself writing a lot of the same code to bind things together between projects. As an alternative, Derby is both interesting and opinionated -- it seems likely to appeal to a certain type of developer rather than for certain projects.

### Flatiron

![](/images/posts/flatiron.png)

[Flatiron](http://flatironjs.org/) (GitHub: [flatiron / flatiron](https://github.com/flatiron/flatiron), License: *MIT*, npm: *flatiron*) from Nodejitsu is a full-stack framework but it's actually very loosely coupled. Each library in the framework can be used in isolation, and it's entirely possible to cherry pick parts of it.

The libraries that make up Flatiron are as follows:

-   Routing: [Director](https://github.com/flatiron/director)
-   Middleware: [Union](https://github.com/flatiron/union)
-   Templating: [Plates](https://github.com/flatiron/plates)
-   ODM: [Resourceful](https://github.com/flatiron/resourceful)
-   Composition: [Broadway](http://github.com/flatiron/broadway)

The routing library, Director, works in both browsers and Node with no external dependencies. Objects are used to represent the relationship between URLs and methods. Union, the routing middleware, is simple but compatible with [Connect](http://senchalabs.github.com/connect/) middleware.

Plates also works in browsers and Node, but it's slightly different to other template libraries. All templates are actually valid HTML, with no special characters for value insertion. The relationship between tags and values is defined through object literals that are passed to <code>Plates.bind</code>:

{% highlight javascript %}
Plates.bind(
  // The first parameter is the HTML template
  '<span></span><img id="bar" class="foo bazz" src=""/>',

  // The second parameter is key/value pairs of data
  { 'bazz': 'Hello, World' },

  // The third parameter defines how values map to tags
  { 'bazz': ['class', 'src'] }
);
{% endhighlight %}

I thought ODM stood for Object Data Manager, or something to do with object databases, but in this context it means "Object-Document Mapper". What's nice about this library is it uses simple prototype objects. Properties can be defined and validated. It also has a chainable property definition API:

{% highlight javascript %}
var resourceful = require('resourceful')
  , Creature = resourceful.define('creature');

Creature.property('legs', Number)
  .required()
  .minimum(0)
  .maximum(8)
  .assert(function (val) { return val % 2 === 0 });
{% endhighlight %}

Judging by the source, only CouchDB is supported right now, but like Racer this library has been designed with a view to adding support for more databases in the future.

The Broadway library provides a simple base for application-like objects that can be extended through plugins. Each app inherits from EventEmitter2.

### Conclusion

Like a lot of new Node web frameworks, Derby capitalises on the relationship between WebSockets, server-side JavaScript, and data storage to make something extremely attractive to both server-side and client-side developers. Conversely, Flatiron doesn't mandate anything, and the example that ships with Flatiron itself is a command-line application.

Derby has addressed a complaint many people have when working with client-side technologies like Backbone, so I'd like to see how it gets used in the future. Send in your Derby, Flatiron, Express, or other apps for review in the weekly Node Roundup!
