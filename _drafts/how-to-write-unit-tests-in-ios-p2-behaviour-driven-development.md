---
layout: post
title:  'How to write unit tests in iOS Part 2: Behaviour-Driven Development (BDD)'
categories: ios testing
tags: ios swift testing unittest test
---

In [part 1](/ios/testing/2016/07/31/how-to-write-unit-tests-in-ios-p1-xctestcase), we discovered the basic usage of `XCTestCase` and wrote some simple unit tests using Swift. For a quick recap, this is what we got so far:

{% highlight swift %}
class MyNativeTests: XCTestCase {

  func testGunCanShootIfItHasBullets() {
    let gun = Gun(bullets: 1)
    gun.shoot()
    XCTAssertTrue(gun.bullets == 0, "expect Gun to be out of bullet")
  }

  func testGunCannotShootIfItHasNoBullet() {
    let gun = Gun(bullets: 0)
    gun.shoot()
    XCTAssertTrue(gun.bullets == 0, "expect the number of bullets to remain the same")
  }

}
{% endhighlight %}

Some text here

{% highlight abc %}
Gun can shoot if it has bullets
Gun cannot shoot if it has no bullet
{% endhighlight %}

Let's make it a little bit more complicated. Our Gun now can only shoot if it is not `locked`. That would require some changes in the test cases like so:

{% highlight abc %}
Gun cannot shoot if it is locked
Gun can shoot if it is unlocked and it has bullets
Gun cannot shoot if it is unlocked but it has no bullet
{% endhighlight %}

And we need another 2 methods for locking/unlocking the Gun:

{% highlight abc %}
Gun cannot shoot if it is locked
Gun can shoot if it is unlocked and it has bullets
Gun cannot shoot if it is unlocked but it has no bullet
Gun lock will make it locked
Gun unlock will make it unlocked
{% endhighlight %}

This could go on and on as our test cases grow. When we look at them, we see a line-by-line list with various conditions and scenarios. It's easy to get lost.

You can't tell what the Gun does just by taking a quick glance anymore. You have to stop and read each line carefully, which is such a waste of time.

What if we can have a better way around this?

Actually there is. Let's try it.

First, we describe a Gun.

{% highlight abc %}
describe a Gun
{% endhighlight %}

A Gun has 3 methods: `shoot`, `lock` and `unlock`.

<script src="https://gist.github.com/hoang-tran/86d270b5bd1dc6bbd5f037a76cea9fb9.js"></script>

The `shoot` method has 2 cases. The `lock` and `unlock` methods each only has one case.

<script src="https://gist.github.com/hoang-tran/85565180d49561edd74eca28329d8720.js"></script>

In the `when is unlocked` case of the `shoot` method, we have 2 sub-cases:

<script src="https://gist.github.com/hoang-tran/566056f7e04300a6803c24542534d590.js"></script>

And each case will have different behaviour.

<script src="https://gist.github.com/hoang-tran/bede5b96cf0b9cb7db907b0b4bb5f618.js"></script>

From looking at this description, you can easily see the whole picture about what a Gun can do. We go from higher level to lower one, from class to methods and from method to its individual cases.

We focus on describing the **behaviour** of a Gun rather than listing out a bunch of cases linearly.

Let's apply this approach to our iOS app.

First, you should follow this [guide](/ios/testing/2016/07/24/how-to-setup-testing-for-new-ios-project/#step-3-setup-unit-tests) (from step 1 to step 3) to setup `cocoapods` and add the necessary pods ([Quick](https://github.com/Quick/Quick) and [Nimble](https://github.com/Quick/Nimble)).

Add a new file called `GunSpec.swift` to your test target. Notice that unit tests written using BDD style should have the suffix `Spec` instead of `Tests` as the naming convention.

Your `GunSpec` class must subclass from `QuickSpec` and override the `spec` method.

{% highlight swift %}
import Quick

class GunSpec : QuickSpec {
  override func spec() {
    super.spec()
  }
}
{% endhighlight %}

Now we're going to describe the Gun's behaviours once again. But this time, we do it with Swift.

First, we describe a Gun. Actually the line `class GunSpec : QuickSpec` already tells this. We don't need to write any more code.

A Gun has 3 methods: `shoot`, `lock` and `unlock`.

{% highlight swift %}
override func spec() {
  super.spec()

  describe("shoot") {

  }

  describe("lock") {

  }

  describe("unlock") {

  }
}
{% endhighlight %}

The `shoot` method has 2 cases. The `lock` and `unlock` methods each only has one case.
(Note that when translate to Swift code, we don't use the word `when`. We use `context` instead)

{% highlight swift %}
override func spec() {
  super.spec()

  describe("shoot") {
    context("is unlocked") {

    }

    context("is locked") {

    }
  }

  describe("lock") {

  }

  describe("unlock") {

  }
}
{% endhighlight %}

In the context `is unlocked` of the `shoot` method, we have 2 sub-cases:

{% highlight swift %}
override func spec() {
  super.spec()

  describe("shoot") {
    context("is unlocked") {
      context("has bullets") {

      }

      context("has no bullet") {

      }
    }

    context("is locked") {

    }
  }

  describe("lock") {

  }

  describe("unlock") {

  }
}
{% endhighlight %}

And each case will have different behaviour.

{% highlight swift %}
override func spec() {
  super.spec()

  describe("shoot") {
    context("is unlocked") {
      context("has bullets") {
        it("can shoot") {

        }
      }

      context("has no bullet") {
        it("can not shoot") {

        }
      }
    }

    context("is locked") {
      it("can not shoot") {

      }
    }
  }

  describe("lock") {
    it("locks the Gun") {

    }
  }

  describe("unlock") {
    it("unlocks the Gun") {

    }
  }
}
{% endhighlight %}

## Wrap up

something something
