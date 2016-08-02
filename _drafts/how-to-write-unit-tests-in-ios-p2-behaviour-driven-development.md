---
layout: post
title:  'How to write unit tests in iOS Part 2: Behaviour-Driven Development (BDD)'
categories: ios testing
tags: ios swift testing unittest
---

In [part 1](/ios/testing/2016/07/31/how-to-write-unit-tests-in-ios-p1-xctestcase), we discovered the basic usage of `XCTestCase` and wrote some simple unit tests using Swift.



## Talk about some problems with XCTestCase

## Introduce a better way of writing tests

How about we take another approach?

First, we describe a Gun.

{% highlight abc %}
describe a Gun
{% endhighlight %}

A Gun has 2 methods: `shoot` and `reload`.

{% highlight abc %}
describe a Gun
  shoot
  reload
{% endhighlight %}

The `shoot` method has 2 cases. The `reload` method only has one case so we don't list it out.

{% highlight abc %}
describe a Gun
  shoot
    when has bullets
    when has no bullet
  reload
{% endhighlight %}

And each case will have different behaviour.

{% highlight abc %}
describe a Gun
  shoot
    when has bullets
      it can shoot
    when has no bullet
      it can not shoot
  reload
    it reloads the gun with full bullets
{% endhighlight %}

From looking at this description, you can easily see the whole picture about what a Gun can do. We go from higher level to lower one, from class to methods and from method to its individual cases.

We focus on describing the **behaviour** of a Gun rather than listing out a bunch of cases linearly.

## Practice writing tests for a calculator class

TDD step by step to create a calculator with these methods:

* add/subtract/multiply/divide
* factorial
* power

## Wrap up
