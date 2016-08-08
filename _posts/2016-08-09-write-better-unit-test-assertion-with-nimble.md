---
layout: post
title:  'Write better unit test assertions with Nimble'
meta_description: Learn how to write better unit test assertions in iOS with the help of Nimble
categories: ios testing
tags: ios swift testing unittest test nimble assertion xctassert
---

Normally, when we write unit tests in iOS, we usually use the `XCTAssert` method to describe our expectation for the test. The syntax is very simple.

{% highlight swift %}
XCTAssert(1 + 1 == 2)

XCTAssertFalse(2 < 1)

XCTAssertEqual("hello", "hello")
{% endhighlight %}

However, when these tests fail, we have no idea **why**.

Xcode only shows us `XCTAssertTrue failed` or `XCTAssertFalse failed` without any explanation.

![](/images/write-better-unit-test-assertions-with-nimble/xctassert-fail.png)

Of course we can add a custom error message like this:

{% highlight swift %}
XCTAssert(1 == 2, "your custom error message here")
{% endhighlight %}

But this seems like an overkill.

Who's gonna write error messages for every assertions? It's gonna take ages.

Why don't we just focus on writing whatever we expect and stop worrying about error messsages?

## That's where Nimble comes to the rescue

With [Nimble](https://github.com/Quick/Nimble), you can write your expectation just like plain English:

{% highlight swift %}
expect(1 + 1).to(equal(2))
// or shorter version
expect(1 + 1) == 2

expect(2).to(beGreaterThan(1))
// or
expect(2) > 1

expect("hello").notTo(equal("holla"))
// or
expect("hello").toNot(equal("holla"))
// or
expect("hello") != "holla"
{% endhighlight %}

And when something fails, it shows exactly what went wrong:

![](/images/write-better-unit-test-assertions-with-nimble/nimble-fail.png)

If that is not clear enough (really?), you can still add your own error message:

{% highlight swift %}
expect(1).to(equal(2), description: "your custom error message here")
{% endhighlight %}

There are a whole lot of useful Nimble methods that you can use to better desribe your expectation:

* Type/Classes

{% highlight swift %}
// Passes if instance is an instance of aClass
expect(instance).to(beAnInstanceOf(aClass))

// Passes if instance is an instance of aClass or any of its subclasses
expect(instance).to(beAKindOf(aClass))
{% endhighlight %}

* Collections

{% highlight swift %}
// expect collection contains element(s)
expect(["dog", "cat", "monkey"]).to(contain("monkey", "dog"))

// expect empty collection
expect([]).to(beEmpty())

// expect collection elements count
expect([1, 5, 7]).to(haveCount(3))
{% endhighlight %}

* Strings

{% highlight swift %}
// expect string contains substring
expect("how are you").to(contain("are"))

// expect string begins with substring
expect("how are you").to(beginWith("how"))

// expect string ends with substring
expect("how are you").to(endWith("you"))
{% endhighlight %}

And so much more...

All available methods can be found on the [Nimble's documentation](https://github.com/Quick/Nimble). There's a section dedicated for [Asynchronous Expectations](https://github.com/Quick/Nimble#asynchronous-expectations) which is extremely useful when you want to write tests for network requests. Please read if you want to fully utilize Nimble's power.

## So how do we install Nimble?

Just like any other pod, all you need to do is adding one single line `pod 'Nimble'` to your Podfile. Note that since we only use Nimble for testing purpose, we want to add it to the test target only.


{% highlight ruby %}
platform :ios, '9.0'

# please replace `MyAwesomeProject` with your real project name
target 'MyAwesomeProject' do
  use_frameworks!

  target 'MyAwesomeProjectTests' do
    inherit! :search_paths
    pod 'Nimble'
  end
end
{% endhighlight %}

Finally open `Terminal` and type:
{% highlight sh %}
pod install
{% endhighlight %}

If you don't know how to setup `cocoapods` properly, you can read this [post](/ios/testing/2016/07/24/how-to-setup-testing-for-new-ios-project/#step-2-setup-cocoapods) for detailed steps.

## Wrap up

In this post, we discussed about some of `XCTAssert`'s shortcomings and how `Nimble` can fill that gap.

`Nimble` also provides an expressive way of writing your expectation so that it reads just like plain English. I highly recommend `Nimble` to anyone who is learning to write unit tests and want to write them beautifully.
