---
layout: post
title:  'How to unit test your Network Layer in iOS'
categories: ios development
tags: ios network unittest rest api
---

## The approach:

## Before you start:

Make sure you're familiar with:

* [How to write unit tests in iOS Part 1: XCTestCase](/ios/testing/2016/07/31/how-to-write-unit-tests-in-ios-p1-xctestcase).
* [How to write unit tests in iOS Part 2: Behavior-driven Development (BDD)](/ios/testing/2016/08/07/how-to-write-unit-tests-in-ios-p2-behavior-driven-development-bdd).
* [Write better unit test assertions with Nimble](/ios/testing/2016/08/09/write-better-unit-test-assertion-with-nimble).

## Step : Setup project for unit testing

Open Podfile and add the following pods to your test target:

* **Quick**: a behavior-driven development (BDD) framework for Swift.
* **Nimble**: write test assertion that reads just like English.
* **Mockingjay**: stub network request in Swift.

{% highlight ruby %}
platform :ios, '9.0'

target 'MyAwesomeProject' do
  use_frameworks!

  target 'MyAwesomeProjectTests' do
    inherit! :search_paths
    pod 'Quick'
    pod 'Nimble'
    pod 'Mockingjay'
  end
end
{% endhighlight %}

Type this command into your Terminal to install all 3 pods.

{% highlight sh %}
pod install
{% endhighlight %}

Open the generated *.xcworkspace* file.

{% highlight sh %}
open MyAwesomeProject.xcworkspace
{% endhighlight %}

## Step : Create a new spec file for MyApiClient

Create a new file called `MyApiClientSpec.swift` in your test target.

{% highlight swift %}
import Quick
import Nimble

class MyApiClientSpec : QuickSpec {
  override func spec() {
    super.spec()

    describe("first test") {
      it("should pass") {
        expect(1).to(equal(1))
      }
    }
  }
}
{% endhighlight %}

Press **Cmd + U** in Xcode and make sure the test pass.

## Step: Write a test that make real network request

Modify the `MyApiClientSpec.swift` to:

{% highlight swift %}
override func spec() {
  super.spec()

  describe("first test") {
    it("should pass") {
      expect(1).to(equal(1))
    }
  }
}
{% endhighlight %}

## Step 2: Create stub response files

## Step 3: Actually write unit test

