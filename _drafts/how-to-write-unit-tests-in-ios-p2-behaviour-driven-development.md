---
layout: post
title:  'How to write unit tests in iOS Part 2: Behaviour Driven Development (BDD)'
categories: ios testing
tags: ios objc swift testing unittest
---

## The better way

Before we begin to write unit tests in iOS, make sure you [have the project setup properly for testing](/ios/testing/2016/07/24/how-to-setup-testing-for-new-ios-project) first

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
