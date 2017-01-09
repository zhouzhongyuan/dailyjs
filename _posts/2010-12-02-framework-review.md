---
layout: post
title: "Let's Make a Framework: Free eBook"
author: Alex Young
categories: 
- frameworks
- tutorials
- lmaf
---

### eBook

I've collected and edited the *Let's Make a Framework* articles into a book that suitable for e-readers. Consider this a Christmas present!

-   [build-a-javascript-framework.pdf](/files/build-a-javascript-framework.pdf)
-   [build-a-javascript-framework.epub](/files/build-a-javascript-framework.epub)
-   [build-a-javascript-framework.mobi](/files/build-a-javascript-framework.mobi) (Kindle)

**Note**: Remember that this book is based on progress up to [commit 09d2c3](https://github.com/alexyoung/turing.js/commit/09d2c387c9a6b1a532c1044f6f1c91c0e5472865). As the framework changes the book might refer to obsolete parts of the framework, so keep this in mind if referring to the latest version of turing.js. Older commits are available in [turing.js's history](https://github.com/alexyoung/turing.js/commits/master/) from GitHub.

### Recap

Last week we finished building a CommonJS-based test framework. Next we need to replace the framework's tests with tests written using this framework. First I'd like to review the progress we've made so far on the project.

If you're new to the series or feeling lost after the last few parts, here's a summary of every part so far. Feel free to dive in at any point that interests you.

1.  Introduction
    1.  [Introduction to the Series](http://dailyjs.com/2010/02/18/framework/)
    2.  [Library Architecture](http://dailyjs.com/2010/02/25/djscript-part-1-structure/)
2.  Object Oriented JavaScript
    1.  [Classes, Inheritance, Extend](http://dailyjs.com/2010/03/04/framework-part-2-oo/)
    2.  [Class Creation In Depth](http://dailyjs.com/2010/03/11/framework-part-3-oo/)
3.  Functional Programming
    1.  [Introduction, Iterators, Performance Concerns](http://dailyjs.com/2010/03/18/framework-part-4-functional/)
    2.  [More Functional Methods](http://dailyjs.com/2010/03/25/framework-part-5/)
4.  Selector Engine
    1.  [DOM History, Browser Support, Performance, Selector Engines](http://dailyjs.com/2010/04/01/framework-part-6/)
    2.  [Parsing, Tokenizer, Scanner](http://dailyjs.com/2010/04/08/framework-part-7-selectors/)
    3.  [Searching with Tokens, Selector Engine API, Tests](http://dailyjs.com/2010/04/15/framework-part-8/)
    4.  [onReady](http://dailyjs.com/2010/08/12/framework-part-25/)
5.  Events
    1.  [Introduction, Using Events, Implementations in the Wild](http://dailyjs.com/2010/04/22/framework-part-9/)
    2.  [Event Registration, W3C Events, Microsoft Events, API Design, Tests](http://dailyjs.com/2010/04/29/framework-part-10/)
    3.  [Stopping Events, Browser Fixes](http://dailyjs.com/2010/05/06/framework-part-11/)
6.  Aliasing and Packing
    1.  [jQuery-Style Aliasing, Packing and Minification](http://dailyjs.com/2010/05/13/framework-part-12/)
7.  Ajax
    1.  [Introduction, History, Request Objects, Sending Requests, Popular APIs](http://dailyjs.com/2010/05/20/framework-part-13/)
    2.  [Cross-Domain Requests, Implementations in the Wild, API Design](http://dailyjs.com/2010/05/27/framework-part-14/)
8.  Animations
    1.  [Introduction, Popular Frameworks, Queues and Events, Animation Basics](http://dailyjs.com/2010/06/03/framework-part-15/)
    2.  [Time-Based Animation, Animating Properties, Parsing Style Values, API](http://dailyjs.com/2010/06/10/framework-part-16/)
    3.  [Easing, Writing Easing Functions](http://dailyjs.com/2010/06/17/framework-part-17/)
    4.  [Animation Helpers, Fade, IE Support](http://dailyjs.com/2010/06/24/framework-part-18/)
    5.  [Colour Support, Transformations, Highlight Helper](http://dailyjs.com/2010/07/01/framework-part-19/)
    6.  [Movement Helper, Chained API](http://dailyjs.com/2010/07/08/framework-part-20/)
    7.  [CSS3 Specifications, CSS3 Transitions, Transforms, Animations, and Hardware Acceleration](http://dailyjs.com/2010/07/15/framework-part-21/)
    8.  [CSS3 Feature Detection](http://dailyjs.com/2010/07/22/framework-part-22/)
9.  Touch
    1.  [Supporting Touchscreen Devices, Orientation](http://dailyjs.com/2010/07/29/framework-part-23/)
    2.  [Events, State](http://dailyjs.com/2010/08/05/framework-part-24/)
10. Chained APIs
    1.  [Namespaces and Chaining, API Design, fakeQuery Example](http://dailyjs.com/2010/08/26/framework-part-27/)
    2.  [Tests, Integration with Existing Library](http://dailyjs.com/2010/09/02/framework-part-28/)
    3.  [Chained Events](http://dailyjs.com/2010/09/09/framework-part-29/)
    4.  [Event Delegation](http://dailyjs.com/2010/09/16/framework-part-30/)
    5.  [Event Delegation Part 2](http://dailyjs.com/2010/09/23/framework-part-31/)
11. Test Framework
    1.  [Introduction, CommonJS](http://dailyjs.com/2010/11/04/framework-part-37/)
    2.  [Assertions, In the Wild](http://dailyjs.com/2010/11/11/framework-part-38/)
    3.  [Testing, Exceptions](http://dailyjs.com/2010/11/18/framework-part-39/)
    4.  [Test Runner and Test Output](http://dailyjs.com/2010/11/25/framework-part-40/)

### Next

-   Rewriting the tests to work with turing-test.js
-   Revised packaging solution, JsLint builds
-   CSS API
-   Pseudo-selectors
-   Better DOM manipulation
-   Bug fixes and browser support
