---
layout: post
title: "Let's Make a Framework: Selectors Part 2"
author: Alex Young
categories: 
- web
- frameworks
- tutorials
- lmaf
---

Welcome to part 7 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework. This part continues looking at CSS selectors.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called Turing and is available on GitHub: [turing.js](http://github.com/alexyoung/turing.js/).

In this part I'll demonstrate implementing a selector engine. It's harder than it looks, but I'll try to keep focused so it doesn't get too complicated.

I've based this selector engine on basic parser concepts, which means you should learn some basic computer science principles here as well as getting down and dirty with the DOM, CSS grammar, and how browsers search using selectors.

### CSS Selectors

Before trying to parse anything, it's a good idea to become familiar with the input. CSS selectors aren't trivial, but to mitigate unnecessary complications I'll limit our library to a subset of CSS2 for now.

CSS2 selectors are explained in detail by in the spec: [Selectors: Pattern Matching](http://www.w3.org/TR/CSS2/selector.html) and [Appendix G. Grammar of CSS 2.1](http://www.w3.org/TR/CSS2/grammar.html). These documents are readable, don't be afraid to skim them!

I want to initially focus on the following syntaxes:

-   <code>E</code> - Matches any element named <code>E</code>
-   <code>E F</code> - Matches any element <code>F</code> that descends from <code>E</code>
-   <code>.class</code> - Matches any element with the class <code>class</code>
-   <code>E.class</code> - Matches any element named <code>E</code> with the class <code>class</code>
-   <code>\#id</code> - Matches any element with the id <code>id</code>
-   <code>E\#id</code> - Matches any element named <code>E</code> with the id <code>id</code>

Any one of these rules forms a *simple selector*. Simple selectors can be chained with *combinators*: white space, '&gt;', and '+'.

### Parsing and Searching Strategy

A good place to look for parsing strategies (other than existing selector engines) is browsers. The Mozilla developer site has an article entitled [Writing Efficient CSS](https://developer.mozilla.org/en/Writing_Efficient_CSS) that actually explains how the style system matches rules.

It breaks up selectors into four categories:

1.  ID Rules
2.  Class Rules
3.  Tag Rules
4.  Universal Rules

The last part of a selector is called the *key selector*. The ancestors of the key selector are analysed to determine which elements match the entire selector. This allows the engine to ignore rules it doesn't need. The upshot of this is that <code>element\#idName</code> would perform slower than <code>\#idName</code>.

This algorithm isn't necessarily the fastest -- many rules would rely on <code>getElementsByTagName</code> returning a lot of elements. However, it's an incredibly easy to understand and pragmatic approach.

Rather than branching off rule categories, we could just put the categories in an object:

{% highlight javascript %}
  findMap = {
    'id': function(root, selector) {
    },

    'name and id': function(root, selector) {
    },

    'name': function(root, selector) {
    },

    'class': function(root, selector) {
    },

    'name and class': function(root, selector) {
    }
  };
{% endhighlight %}

### Tokenizer

Tokens are just categorised strings of characters. This stage of a parser is called a *lexical analyser*. It sounds more complicated than it is. Given a selector, we need to:

-   Normalise it to remove any extra white space
-   Process it by some means to transform it into a sequence of instructions (tokens) for our parser
-   Run the tokens through the parser against the DOM

We can model a token as a class like this:

{% highlight javascript %}
function Token(identity, finder) {
  this.identity = identity;
  this.finder   = finder;
}

Token.prototype.toString = function() {
  return 'identity: ' + this.identity + ', finder: ' + this.finder;
};
{% endhighlight %}

The <code>finder</code> property is one of those keys in <code>findMap</code>. The <code>identity</code> is the original rule from the selector.

### Scanner

Both [Sly](http://github.com/digitarald/sly) and [Sizzle](http://sizzlejs.com/) use a giant regular expression to break apart and process a selector. Sizzle calls this the <code>Chunker</code>.

Given the performance and flexibility of regular expressions in JavaScript, this is a good approach. However, I don't want to confuse readers with a giant regular expression. What we need is an intermediate approach.

Most programming languages have tools that generate tokenizers based on lexical patterns. Typically a lexical analyser outputs source code based on the output of a parser generator

Lexers were traditionally used for building programming language parsers, but we now live in a world filled with a gamut of computer-generated data. This means that you'll even find these tools cropping up in projects like [nokogiri](http://nokogiri.org/), a Ruby HTML and XML parser.

The power of lexers lies in the fact that there's a layer of abstraction between the programmer and the parser. Working on simple rules is easier than figuring out the finer implementation details.

Let's use an incredibly simplified version of a lexer to create the regular expression that drives the tokenizer. These rules can be based on the lexical scanner description in the [CSS grammer specification](http://www.w3.org/TR/CSS2/grammar.html).

It will be useful to embed regular expressions within other ones, and not worry about escaping them. Objects like this will drive the process:

{% highlight javascript %}
macros = {
  'nl':        '\n|\r\n|\r|\f',
  'nonascii':  '[^\0-\177]',
  'unicode':   '\\[0-9A-Fa-f]{1,6}(\r\n|[\s\n\r\t\f])?',
  'escape':    '#{unicode}|\\[^\n\r\f0-9A-Fa-f]',
  'nmchar':    '[_A-Za-z0-9-]|#{nonascii}|#{escape}',
  'nmstart':   '[_A-Za-z]|#{nonascii}|#{escape}',
  'ident':     '[-@]?(#{nmstart})(#{nmchar})*',
  'name':      '(#{nmchar})+'
};

rules = {
  'id and name':    '(#{ident}##{ident})',
  'id':             '(##{ident})',
  'class':          '(\\.#{ident})',
  'name and class': '(#{ident}\\.#{ident})',
  'element':        '(#{ident})',
  'pseudo class':   '(:#{ident})'
};
{% endhighlight %}

The scanner will work by:

-   Expanding <code>\#{}</code> in the macros
-   Expanding <code>\#{}</code> in the rules based on the expanded macros
-   Escaping the backslashes
-   Joining each of the patterns with <code>|</code>
-   Building a global regular expression with the <code>RegExp</code> class

This will output a giant regular expression much like the ones used by Sizzle and Sly. The advantage of this approach is you can see the relationship between the tokenized output and the DOM matchers/searchers.

### Processing the Giant Regular Expression

After a selector is normalised, it will be broken apart using the regular expression from the scanner. This works based on the indexes of the matched elements:

{% highlight javascript %}
while (match = r.exec(this.selector)) {
  finder = null;

  if (match[10]) {
    finder = 'id';
  } else if (match[1]) {
    finder = 'name and id';
  } else if (match[29]) {
    finder = 'name';
  } else if (match[15]) {
    finder = 'class';
  } else if (match[20]) {
    finder = 'name and class';
  }
  this.tokens.push(new Token(match[0], finder));
}
{% endhighlight %}

Despite being obtuse, This is more efficient than looking at <code>match\[0\]</code> with each of the regexes in the <code>rules</code> object.

### Next Week

The <code>Searcher</code> class implements the algorithm that Firefox uses. It's simple enough that turing.dom.js currently passes tests in IE6, IE7, IE8, Firefox, Safari, Chrome and Opera. I'll explain this next week, and also demonstrate how I used test-driven development to develop the parser and tokenizer.

To see the most recent code, have a look at [turing.dom.js](http://github.com/alexyoung/turing.js/blob/master/turing.dom.js) on GitHub.
