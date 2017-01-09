---
layout: post
title: "The How and Why of AMD"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- amd
---

<div class="intro">
*Let's Make a Framework* is an ongoing series about building a JavaScript framework from the ground up.

These articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js). Documentation is available at [turingjs.com](http://turingjs.com/).

</div>
In [Improving Client-Side Modularity](http://dailyjs.com/2011/11/24/framework-88/) I talked about better ways to load modules, and showed how jQuery supports the [Asynchronous Module Definition (AMD)](https://github.com/amdjs/amdjs-api/wiki/AMD) API. As open source developers, supporting this specification makes it easier for others to reuse our code. It also helps resource loading frameworks like [RequireJS](http://requirejs.org/).

In the *Let's Make a Framework* series, we've already covered [Common JS Modules](http://wiki.commonjs.org/wiki/Modules/1.1) and seen how they can make our code accessible for Node developers. However, implementing a convincing client-side version of this is difficult because <code>require</code> is generally implemented as a synchronous statement. There are projects to wrap this for browser-based development, but AMD offers a native solution that's easy to work with. This is where AMD steps in as a "transport format".

### Conditional AMD Support

jQuery provides conditional AMD support. When writing smaller, tightly focused libraries, this is a useful approach because it allows AMD to be supported where available. jQuery checks for the existence of <code>define</code>, and this approach can be reused:

{% highlight javascript %}
var myModule = {
  awesome: 'not really'
};

if (typeof define === 'function' && define.amd) {
  define('myModule', [], function() {
    return myModule;
  });
}
{% endhighlight %}

The <code>amd</code> property is explained by the AMD specification:

> To allow a clear indicator that a global define function (as needed for script src browser loading) conforms to the AMD API, any global define function SHOULD have a property called "amd" whose value is an object. This helps avoid conflict with any other existing JavaScript code that could have defined a define() function that does not conform to the AMD API.

We need to check for both the <code>define</code> function, and that it conforms to the specification.

### Internal AMD Use

Another approach is to build an entire project around AMD. [Dojo](http://dojotoolkit.org/) was restructured to work this way. This comment is from Dojo's source:

> This function defines an AMD-compliant loader that can be configured to operate in either synchronous or asynchronous modes.

Then modules are wrapped with <code>define</code>:

{% highlight javascript %}
define(["./_base/kernel", "./_base/lang", "./_base/Color", "./_base/array"], function(dojo, lang, Color, ArrayUtil) {

});
{% endhighlight %}

Notice how dependencies in AMD are the same order as the callback (or "factory function", as the AMD specification refers to it). This is explained in the specification:

> \[...\] the resolved values should be passed as arguments to the factory function with argument positions corresponding to indexes in the dependencies array.

As I mentioned in *Improving Client-Side Modularity*, Kris Zyp documented this move to AMD in [Asynchronous Modules Come to Dojo 1.6](http://dojotoolkit.org/features/1.6/async-modules).

### CommonJS Wrapping

The AMD specification also addresses CommonJS modules with this example:

{% highlight javascript %}
define(function(require, exports, module) {
  var a = require('a'),
      b = require('b');

  exports.action = function() {};
});
{% endhighlight %}

The RequireJS documentation points out that this can help with cases where there are a lot of dependencies:

{% highlight javascript %}
define([ 'require', 'jquery', 'blade/object', 'blade/fn', 'rdapi',
         'oauth', 'blade/jig', 'blade/url', 'dispatch', 'accounts',
         'storage', 'services', 'widgets/AccountPanel', 'widgets/TabButton',
         'widgets/AddAccount', 'less', 'osTheme', 'jquery-ui-1.8.7.min',
         'jquery.textOverflow'], // ...
{% endhighlight %}

This is difficult to read. The RequireJS documentation goes on to say that using <code>require</code> is potentially easier to follow. An AMD module loader has to parse out the <code>require</code> calls and insert the <code>define</code> dependencies transparently.

Since the authors of RequireJS have experience implementing such things, they're all too aware of browser inconsistencies and limitations:

> Not all browsers give a usable Function.prototype.toString() results. As of October 2011, the PS 3 and older Opera Mobile browsers do not. Those browsers are more likely to need an optimized build of the modules for network/device limitations, so just do a build with an optimizer that knows how to convert these files to the normalized dependency array form, like the RequireJS optimizer.

This means a pre-compilation step may be necessary to support a broad range of browsers using this approach.

### Conclusion

If you've followed our previous tutorials on building an asynchronous script loader, you'll know that writing JavaScript APIs for loading scripts isn't trivial. Supporting CommonJS isn't necessarily difficult, but supporting features like parsing <code>require</code> calls is where things get tricky.

This leads us to an important point that must be considered when building our own libraries and frameworks: should we ship our own script loading solution, or conditionally support AMD? Given the complexity of building a script loader, it may be better to conditionally support AMD as jQuery does.

This doesn't quite satisfy our needs for Turing, however. Turing does have one or two modules that are dependent on each other, and <code>define</code> provides a way to express this programatically. It may be better to use <code>define</code> to wrap modules, as we already do using an IIFE (Immediately-Invoked Function Expression), and effectively no-op it if it isn't present.

Next week I'll look at adding AMD to Turing, then test it out with RequireJS.

### References

-   [Asynchronous Modules Come to Dojo 1.6](http://dojotoolkit.org/features/1.6/async-modules)
-   [Improving Client-Side Modularity](http://dailyjs.com/2011/11/24/framework-88/)
-   [RequireJS: Why AMD?](http://requirejs.org/docs/whyamd.html)
-   [AMD Wiki](https://github.com/amdjs/amdjs-api/wiki/AMD)
-   [CommonJS Modules](http://wiki.commonjs.org/wiki/Modules/1.1)
