---
layout: post
title: "Let's Make a Framework: Selectors Part 3"
author: Alex Young
categories: 
- web
- frameworks
- tutorials
- lmaf
---

Welcome to part 8 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at the selector engine.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

[Last week](http://dailyjs.com/2010/04/08/framework-part-7-selectors/) I discussed implementing the framework's selector engine. The selector engine uses a regex-based tokenizer and scanner, which breaks up selectors so they can be analysed.

In this part I'll continue explaining how the selector engine searches for nodes using the <code>Searcher</code> class.

### Searcher

The <code>Searcher</code> class uses the output of the <code>Tokenizer</code> to search the DOM. I've based the algorithm on how Firefox works. To recap from last week:

> The last part of a selector is called the *key selector*. The ancestors of the key selector are analysed to determine which elements match the entire selector.

The <code>Searcher</code> is instantiated with a root element to search from and an array of tokens. The key selector is the last item in the tokens array. Each token has an identity and a *finder* -- the finder is the glue between selectors and JavaScript's DOM searching methods. At the moment I've only implemented class and ID-based finders. They're stored in an object called <code>findMap</code>:

{% highlight javascript %}
find = {
  byId: function(root, id) {
    return [root.getElementById(id)];
  },

  byNodeName: function(root, tagName) {
    var i, results = [], nodes = root.getElementsByTagName(tagName);
    for (i = 0; i < nodes.length; i++) {
      results.push(nodes[i]);
    }
    return results;
  },

  byClassName: function(root, className) {
    var i, results = [], nodes = root.getElementsByTagName('*');
    for (i = 0; i < nodes.length; i++) {
      if (nodes[i].className.match('\\b' + className + '\\b')) {
        results.push(nodes[i]);
      }
    }
    return results;
  }
};

findMap = {
  'id': function(root, selector) {
    selector = selector.split('#')[1];
    return find.byId(root, selector);
  },

  'name and id': function(root, selector) {
    var matches = selector.split('#'), name, id;
    name = matches[0];
    id = matches[1];
    return filter.byAttr(find.byId(root, id), 'nodeName', name.toUpperCase());
  }

  // ...
};
{% endhighlight %}

The <code>byClassName</code> method is replaced when browsers support <code>getElementsByClassName</code>. The implementation I've used here is the most basic and readable one I could think of.

The <code>Searcher</code> class abstracts accessing this layer of the selector engine by implementing a <code>find</code> method:

{% highlight javascript %}
Searcher.prototype.find = function(token) {
  if (!findMap[token.finder]) {
    throw new InvalidFinder('Invalid finder: ' + token.finder); 
  }
  return findMap[token.finder](this.root, token.identity); 
};
{% endhighlight %}

An exception is thrown when the finder does not exist. The core part of the algorithm is <code>matchesToken</code>. This is used to determine if a token matches a given node:

{% highlight javascript %}
Searcher.prototype.matchesToken = function(element, token) {
  if (!matchMap[token.finder]) {
    throw new InvalidFinder('Invalid matcher: ' + token.finder); 
  }
  return matchMap[token.finder](element, token.identity);
};
{% endhighlight %}

This looks a lot like the <code>find</code> method. The only difference is <code>matchMap</code> is used to check if an element matches a selector, rather than searching for nodes that match a selector.

Each token for a given list of nodes is matched using this method in <code>matchesAllRules</code>. A <code>while</code> loop is used to iterate up the tree:

{% highlight javascript %}
while ((ancestor = ancestor.parentNode) && token) {
  if (this.matchesToken(ancestor, token)) {
    token = tokens.pop();
  }
}
{% endhighlight %}

If there are no tokens at the end of this process then the element matches all of the tokens.

With all that in place, searching the DOM is simple:

{% highlight javascript %}
Searcher.prototype.parse = function() {
  // Find all elements with the key selector
  var i, element, elements = this.find(this.key_selector), results = [];

  // Traverse upwards from each element to see if it matches all of the rules
  for (i = 0; i < elements.length; i++) {
    element = elements[i];
    if (this.matchesAllRules(element)) {
      results.push(element);
    }
  }
  return results;
};
{% endhighlight %}

Each element that matches the key selector is used as a starting point. Its ancestors are analysed to see if they match all of the selector's rules.

As I said in the previous tutorial this isn't going to be the fastest selector engine ever made, but it is easy to understand and extend.

### Implementing the API

Now we have all these tools for searching nodes and parsing selectors we need to knit them into a usable API. I decided to expose the tokenizer during testing -- this was inspired by [Sly](http://github.com/digitarald/sly/) which gives access to its parser's output.

The public methods look like this:

{% highlight javascript %}
dom.tokenize = function(selector) {
  var tokenizer = new Tokenizer(selector);
  return tokenizer;
};

dom.get = function(selector) {
  var tokens = dom.tokenize(selector).tokens,
      searcher = new Searcher(document, tokens);
  return searcher.parse();
};
{% endhighlight %}

The method most people will generally use is <code>dom.get</code>, which takes a selector and returns an array of elements. The elements aren't currently wrapped in a special object like jQuery uses, but this would be a useful future improvement. I thought it would be interesting to leave this out and implement it when other parts of framework actually need it.

### Tests

As I was writing the selector engine I wrote tests. Unlike other turing tests these require a browser to run -- I could get around this, and it would be nice if the selector engine could search arbitrary XML, but to keep it simple I've kept this constraint.

I started developing the selector engine by writing this test:

{% highlight javascript %}
  given('a selector to search for', function() {
    should('find with id', turing.dom.get('#dom-test')[0].id).isEqual('dom-test');
{% endhighlight %}

Once that was done I moved on to searching for classes, then combinations of tag names and classes.

I've built the framework with this test-first approach, and I highly recommend it for your own projects.

### Next Week

The selector engine still needs work. I haven't yet touched on caching, and it's not close to full CSS 2.1 support yet. After three parts on the same topic I thought I'd switch gears and explore events.
