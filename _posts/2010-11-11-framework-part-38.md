---
layout: post
title: "Let's Make a Test Framework"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
- testing
---

Welcome to part 38 of *Let's Make a Framework*, the ongoing series about building a JavaScript framework.

If you haven't been following along, these articles are tagged with [lmaf](http://dailyjs.com/tags.html#lmaf). The project we're creating is called [Turing](http://github.com/alexyoung/turing.js).

Last week I started building [turing-test.js](https://github.com/alexyoung/turing-test.js) -- a basic unit testing framework. The idea behind this framework is to build something that follows the CommonJS specifications and works in modern browsers. This will help us test Turing more effectively.

### Asserts

Since browsers don't support [CommonJS Unit Test 1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0), we starting building our own <code>assert</code> module. The architecture of our module is similar to many other unit testing frameworks -- a <code>fail</code> function raises an exception whenever the *actual* value isn't the same as the *expected* value.

The assertions defined by the specification are:

<code>assert.ok(guard, message\_opt)</code>
Pure assertion tests whether a value is *truthy*.

<code>assert.equal(actual, expected, message\_opt)</code>
Shallow, coercive equality with <code>==</code>.

<code>assert.notEqual(actual, expected, message\_opt)</code>
Tests if two objects are not equal with <code>!=</code>.

<code>assert.deepEqual(actual, expected, message\_opt)</code>
Equivalence, as determined by <code>===</code>, and special handling for dates, <code>Object</code>, <code>Array</code>.

<code>assert.notDeepEqual(actual, expected, message\_opt)</code>
The inverse of the above.

<code>assert.strictEqual(actual, expected, message\_opt)</code>
Tests strict equality, as determined by <code>===</code>

<code>assert.notStrictEqual(actual, expected, message\_opt)</code>
The inverse, using <code>!==</code>

<code>assert.throws(block, Error\_opt, message\_opt)</code>
Expects an exception.

For those of you who tl;dr'd that, the CommonJS assertion module defines methods for equality, *deep* equality, and *strict* equality. The difference between these is important, and if my tutorial fails to elucidate you please refer to Douglas Crockford's <a href="http://www.amazon.co.uk/gp/product/0596517742/ref=as_li_qf_sp_asin_tl?ie=UTF8&tag=da0b-21&linkCode=as2&camp=1634&creative=6738&creativeASIN=0596517742">JavaScript: The Good Parts</a><img src="http://www.assoc-amazon.co.uk/e/ir?t=da0b-21&l=as2&o=2&a=0596517742" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />.

### Equality Assertions

I've written about the equality operators on this very blog before, but let's recap. JavaScript offers two distinct sets of equality operators. As JSLint will tell you, <code>=&lt;/code&gt; and &lt;code&gt;!</code> are generally what you want. Crockford calls them *good*, because they work how we expect: if two values have the same type and value, <code>===</code> will result in <code>true</code>.

Meanwhile, <code>&lt;/code&gt; and &lt;code&gt;!=&lt;/code&gt; will coerce values if they're not of the same type.  This can lead to confusing behaviour, like &lt;code&gt;0  ''</code> equating to <code>true</code>.

That explains coercive equality and strict, but what about *deep*? The CommonJS specification defines it deep equality as:

1.  All identical values are equivalent, as determined by <code>=&lt;/code&gt;
    \# If the expected value is a &lt;code&gt;Date&lt;/code&gt; object, the actual value is equivalent if it is also a &lt;code&gt;Date&lt;/code&gt; object that refers to the same time
    \# For pairs that do not both pass &lt;code&gt;typeof value  "object"</code>, equivalence is determined by <code>==</code>
2.  For all other <code>Object</code> pairs, including <code>Array</code> objects, equivalence is determined by having the **same number of owned properties** (as verified with <code>Object.prototype.hasOwnProperty.call</code>), the same set of keys (although not necessarily the same order), equivalent values for every corresponding key, and an identical "prototype" property. Note: this accounts for both named and indexed properties on an <code>Array</code>.

### Research

Let's take a look at some equality code in the wild from an arbitrary JavaScript test framework, [Jasmine](https://github.com/pivotal/jasmine/):

{% highlight javascript %}
jasmine.Env.prototype.equals_ = function(a, b, mismatchKeys, mismatchValues) {
  mismatchKeys = mismatchKeys || [];
  mismatchValues = mismatchValues || [];

  for (var i = 0; i < this.equalityTesters_.length; i++) {
    var equalityTester = this.equalityTesters_[i];
    var result = equalityTester(a, b, this, mismatchKeys, mismatchValues);
    if (result !== jasmine.undefined) return result;
  }

  if (a === b) return true;

  if (a === jasmine.undefined || a === null || b === jasmine.undefined || b === null) {
    return (a == jasmine.undefined && b == jasmine.undefined);
  }

  if (jasmine.isDomNode(a) && jasmine.isDomNode(b)) {
    return a === b;
  }

  if (a instanceof Date && b instanceof Date) {
    return a.getTime() == b.getTime();
  }

  if (a instanceof jasmine.Matchers.Any) {
    return a.matches(b);
  }

  if (b instanceof jasmine.Matchers.Any) {
    return b.matches(a);
  }

  if (jasmine.isString_(a) && jasmine.isString_(b)) {
    return (a == b);
  }

  if (jasmine.isNumber_(a) && jasmine.isNumber_(b)) {
    return (a == b);
  }

  if (typeof a === "object" && typeof b === "object") {
    return this.compareObjects_(a, b, mismatchKeys, mismatchValues);
  }

  //Straight check
  return (a === b);
};
{% endhighlight %}

There are some commonalities between this code and the CommonJS specification. For example, <code>Date</code> comparison using <code>getTime</code>. It deviates slightly by having specific handling for DOM nodes, and the type checks for strings or numbers.

Here's what Node does in its assert module. The original source even has lines from the specification pasted in:

{% highlight javascript %}
function _deepEqual(actual, expected) {
  // 7.1. All identical values are equivalent, as determined by ===.
  if (actual === expected) {
    return true;

  } else if (Buffer.isBuffer(actual) && Buffer.isBuffer(expected)) {
    if (actual.length != expected.length) return false;

    for (var i = 0; i < actual.length; i++) {
      if (actual[i] !== expected[i]) return false;
    }

    return true;

  // 7.2. If the expected value is a Date object, the actual value is
  // equivalent if it is also a Date object that refers to the same time.
  } else if (actual instanceof Date && expected instanceof Date) {
    return actual.getTime() === expected.getTime();

  // 7.3. Other pairs that do not both pass typeof value == "object",
  // equivalence is determined by ==.
  } else if (typeof actual != 'object' && typeof expected != 'object') {
    return actual == expected;

  // 7.4. For all other Object pairs, including Array objects, equivalence is
  // determined by having the same number of owned properties (as verified
  // with Object.prototype.hasOwnProperty.call), the same set of keys
  // (although not necessarily the same order), equivalent values for every
  // corresponding key, and an identical "prototype" property. Note: this
  // accounts for both named and indexed properties on Arrays.
  } else {
    return objEquiv(actual, expected);
  }
}

function isUndefinedOrNull (value) {
  return value === null || value === undefined;
}

function isArguments (object) {
  return Object.prototype.toString.call(object) == '[object Arguments]';
}

function objEquiv (a, b) {
  if (isUndefinedOrNull(a) || isUndefinedOrNull(b))
    return false;
  // an identical "prototype" property.
  if (a.prototype !== b.prototype) return false;
  //~~~I've managed to break Object.keys through screwy arguments passing.
  //   Converting to array solves the problem.
  if (isArguments(a)) {
    if (!isArguments(b)) {
      return false;
    }
    a = pSlice.call(a);
    b = pSlice.call(b);
    return _deepEqual(a, b);
  }
  try{
    var ka = Object.keys(a),
      kb = Object.keys(b),
      key, i;
  } catch (e) {//happens when one is a string literal and the other isn't
    return false;
  }
  // having the same number of owned properties (keys incorporates hasOwnProperty)
  if (ka.length != kb.length)
    return false;
  //the same set of keys (although not necessarily the same order),
  ka.sort();
  kb.sort();
  //~~~cheap key test
  for (i = ka.length - 1; i >= 0; i--) {
    if (ka[i] != kb[i])
      return false;
  }
  //equivalent values for every corresponding key, and
  //~~~possibly expensive deep test
  for (i = ka.length - 1; i >= 0; i--) {
    key = ka[i];
    if (!_deepEqual(a[key], b[key] ))
       return false;
  }
  return true;
}
{% endhighlight %}

Node's code includes a few key techniques that are worth remembering:

-   A strict equality check is performed up front
-   <code>getTime</code> is used on the dates again
-   <code>Array.prototype.slice.call</code> is used to transform function arguments into arrays, so they can be tested like arrays
-   Key equivalence is tested to find differences in keys early before having to resort to another deep test on nested objects
-   <code>\_deepEqual</code> is called recursively for nested objects

If you take a look at the [Turing Test repository](https://github.com/alexyoung/turing-test.js), I've basically used Node's implementation with a few modifications.

### Conclusion

This is actually the main body of assertions as defined by the CommonJS specification. There's one more, <code>assert.throws</code>, which I'll define next week, along with a look at testing the tests. If you'd like to look at the code for this tutorial, it's in [commit 9a4c33e79cbeb0bf2a71](https://github.com/alexyoung/turing-test.js/commit/9a4c33e79cbeb0bf2a719bc155590bdb14c7390b) on GitHub.

### References

-   [CommonJS Unit Test 1.0](http://wiki.commonjs.org/wiki/Unit_Testing/1.0)
-   [Node's assert.js](https://github.com/ry/node/blob/master/lib/assert.js)
-   [Jasmine](https://github.com/pivotal/jasmine/)
