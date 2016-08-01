---
layout: post
title:  'How to write unit tests in iOS Part 2: Behaviour-Driven Development (BDD)'
categories: ios testing
tags: ios swift testing unittest
---

## Remind about previous post

## Talk about some problems with XCTestCase

## Introduce a better way of writing tests

First, talk about way in pseudo code step by step
{% highlight abc %}
describe a Gun
{% endhighlight %}
{% highlight abc %}
describe a Gun
  shoot
  reload
{% endhighlight %}
{% highlight abc %}
describe a Gun
  shoot
    when has bullets
    when has no bullet
  reload
{% endhighlight %}
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

## Practice writing tests for a calculator class

TDD step by step to create a calculator with these methods:

* add/subtract/multiply/divide
* factorial
* power

## Wrap up
