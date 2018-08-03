---
layout: post
title: "The Three Things Theorem of Function Intention"
description: "Functions should cordinate, calculate, or control"
date: 2018-08-03
tags:
- code-design
- JavaScript
comments: true
share: true
---

<div style="display: grid; grid-template-columns: 330px auto">
  <p style="padding: 6px 20px 10px 0px">
    <a href="https://en.wikipedia.org/wiki/Orizuru" alt="orizuru: origami crane">
      <img src="/assets/images/three-things/orizuru-paper-cranes.jpg" alt="orizuru: origami crane"/>
    </a>
  </p>

  <p>Origami, the ancient art of paper folding, has gone through a <a href="http://www.pbs.org/independentlens/between-the-folds/history.html">modern mathematical renaissance</a>. The folding process was originally passed down between generations orally. Recently, though, artists have developed a way to document the steps to fold a shape with a diagram called a <a href="http://www.langorigami.com/crease-patterns">crease pattern</a>.</p>

  <p style="padding: 0px 20px 10px 0px">
    <a href="https://en.wikipedia.org/wiki/Mathematics_of_paper_folding" alt="mathematics of paper folding" style="">
      <img style="height: 320px; width: 320px;" src="/assets/images/three-things/two-colorability.png" alt="two colorability of bird base"/>
    </a>
  </p>

  <p>New theorems have emerged to determine whether a crease pattern can be folded. For example, a crease pattern must be <a href="https://en.wikipedia.org/wiki/Maekawa%27s_theorem">two-colorable</a>. Each region in the crease pattern can be colored with one of two colors in such a way that no two regions of the same color share a dividing crease.</p>

</div>

<blockquote style="margin-left: 20px"><p>The art of origami has been reduced to simple, elegant, binary theorems that represents its integrity.</p></blockquote>

## A Theorem Proving Code Intention

We can think of our source files as crease patterns used to diagram the intention of the systems we build. We can then define theorems to help us ensure that our diagrams, our source files, have integrity. Each function should have a clear intention, and to prove that, I'd like to introduce **The Three-Things Theorem of Function Intention**.

## A function should do one of these three things

### Coordinate

**Coordinate the actions of other small, well-named units by using a defined control flow mechanism.**

```javascript
function coordinator () {
  return callFirstFunction
    .then(res => callSecondFunction(res))
    .then(res => callThirdFunction(res))
}
```

A coordinating function is responsible for creating a pipeline of actions, for calling other well named functions. It manages interactions and that's it. Just like a good _people_ manager, a coordinating function becomes less effective when it's splitting time actually doing the work that should be a delegated task.

### Calculate

**Calculate a very specific value, state, or object in our system.**

```javascript
function simpleCalculator (val1, val2, val3) {
  return Math.pow(Math.abs(val1 + val2) * val3, 2)
}

function domainObjectCalculator (input) {
  return { output: _.get(input, 'foo.bar.baz')  }
}
```

Calculating a value doesn't just need to be limited to a mathematical equation. A calculating function can also be used for transitions in domain concepts, like when we "calculate" new objects in our system.

### Control

**Encapsulate and expose ways to control the flow of our application.**

```javascript
function until (condition, run) {
  return function method (subject) {
    if (!condition(subject)) {
      return method(run(subject))
    } else {
      return subject
    }
  }
}

function nComplete(n) {
  return ...
}

function calculateNewN(oldN) {
  return ...
}

until(nComplete, calculateNewN)(5)
```

We can build functions that perform the control flow through our applications. They can hold conditional logic, looping, and anything algorithmic. This allows our domain functions to focus on performing a specific task. `until` is an example of an fairly complex path through our application, but structuring it as a higher-order-function allows you to prove that it works with tests that aren't mixed up by _other_ complex domain logic. All the pieces can be tested in isolation, making them easier to develop, change, and understand.

## Lean More

<p><b>About this idea</b> in a talk about code design that <a href="https://twitter.com/samjonester">Sam</a> gave called <a href="http://blog.testdouble.com/posts/2018-05-02-javascript-is-too-convenient">JavaScript Is Too Convenient</a></p>
<p><b>About origami</b> from an amazing talk given by <a href="https://twitter.com/robbykraft">Robby Kraft</a> on <a href="https://www.thestrangeloop.com/2017/origami-software-from-scratch.html">software built for origami artists</a></p>


