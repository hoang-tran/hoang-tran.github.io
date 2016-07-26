---
layout: post
title:  How to write unit tests in iOS
categories: ios testing
tags: ios objc swift testing unittest
---

## What is unit test?

`Unit test` is basically just a test at the `unit` level. Or to put simply, you test each `class` in isolation

A `unit test` usually has 4 steps:

### 1. Setup:
  * This is where you initialize the `class` you want to test

### 2. Execution:
  * Call method on the `class` or instance of the `class`

### 3. Expectation:
  * Set some expectations for the outcome

### 4. Cleanup:
  * Clean up anything that may affect other subsequent tests

Let's look at some example unit tests here (in pseudo code):

* Test the `shoot` method on `Gun` class:

  * Case 1: Gun CAN shoot if it has bullet(s)
<script src="https://gist.github.com/hoang-tran/4521afc417fd8e2b3c3cdd31d978723e.js"></script>
  * Case 2: Gun CANNOT shoot if it has no bullet
<script src="https://gist.github.com/hoang-tran/645b0c0647b4335e1c57899ed16a14dc.js"></script>

* Test the `reload` method on `Gun` class:
<script src="https://gist.github.com/hoang-tran/6c1f9bab9e85ca53dd30d452de298842.js"></script>

As you can see, for each case, we only test a very small part of the object. That's why it is called `unit` test.

A `class` may have multiple `methods`. Each `method` will have multiple unit tests to cover all of its cases

## So how does all of this translate into iOS

Before we begin to write unit tests in iOS, make sure you [have the project setup properly for testing](/ios/testing/2016/07/24/how-to-setup-testing-for-new-ios-project) first

## The better way

describe > context > it
beforeEach, afterEach
beforeAll, afterAll

## Practice writing tests for a calculator class

TDD step by step to create a calculator with these methods:
* add/subtract/multiply/divide
* factorial
* power

## What makes unit test hard?

* Dependencies

## Which part should we write unit tests for?

* What you **SHOULD** unit test:
  * model
  * network layer
  * database layer
  * utilities/helpers/extensions

* What you **SHOULD NOT** unit test:
  * controller
  * view
  * class that has lots of dependencies
