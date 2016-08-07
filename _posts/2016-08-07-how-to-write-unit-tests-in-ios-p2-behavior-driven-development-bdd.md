---
layout: post
title:  'How to write unit tests in iOS Part 2: Behavior-driven Development (BDD)'
meta_description: Learn how to write elegant unit tests in iOS using Behavior-driven Development (BDD)
categories: ios testing
tags: ios swift testing unittest test bdd
---

In [part 1](/ios/testing/2016/07/31/how-to-write-unit-tests-in-ios-p1-xctestcase), we discovered the basic usage of `XCTestCase` and wrote some simple unit tests for the Gun class using Swift. For a quick recap, this is what we got so far:

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

As you can see, each case in the method has its own unit test. They are arranged with this structure:

{% highlight abc %}
test method A case 1
test method A case 2
test method B
test method C case 1
test method C case 2 sub-case 1
test method C case 2 sub-case 2
...
{% endhighlight %}

This is the traditional way of writing unit tests in which you list out all the test cases linearly. Each test case is separated and does not relate to each other.

Many projects still prefer this approach because it's very simple and straightforward.

## But is that the best we can do?

Let's look back at our example and further develop it. Currently we have 2 cases for the `shoot` method. (in pseudo code)

{% highlight abc %}
test Gun can shoot if it has bullets
test Gun cannot shoot if it has no bullet
{% endhighlight %}

Let's make it a little bit more complicated. Our Gun now can only shoot if it is `unlocked`. That would require some changes in the test cases like so:

{% highlight abc %}
test Gun can shoot if it has bullets and it is unlocked
test Gun cannnot shoot if it has bullets but it is locked
test Gun cannot shoot if it has no bullet
{% endhighlight %}

And we need another 2 methods for locking/unlocking the Gun:

{% highlight abc %}
test Gun can shoot if it has bullets and it is unlocked
test Gun cannnot shoot if it has bullets but it is locked
test Gun cannot shoot if it has no bullet
test Gun lock will make it locked
test Gun unlock will make it unlocked
{% endhighlight %}

Now when we look at it, we see a line-by-line list with various conditions and scenarios. It's easy to get lost.

You can't tell what the Gun does just by taking a quick glance anymore. You have to stop and read each line carefully, which is not very ideal.

Wouldn't it be great if we could group all the common parts among test cases?

Actually there is such way. It's called `Behavior-driven Development`, or `BDD` for short.

The idea is: you describe how your class would **behave** from the top down to the bottom. You describe the `big picture` first, then go down to the nitty gritty.

Here is the `BDD` structure:

{% highlight abc %}
describe class Y
  describe method A
    case 1
    case 2

  describe method B

  describe method C
    case 1
    case 2
      sub-case 1
      sub-case 2
  ...
{% endhighlight %}

As you can see, there is no duplicated parts anymore. Tests are grouped by methods and cases. You can easily skim and still get the whole picture about what the class can do.

## Let's try it with our Gun example

We will go from the very top (`class`) and down one level at a time.

First, we describe a Gun.

{% highlight abc %}
describe a Gun
{% endhighlight %}

A Gun has 3 methods: `shoot`, `lock` and `unlock`.

{% highlight abc %}
describe a Gun
  describe shoot
  describe lock
  describe unlock
{% endhighlight %}

The `shoot` method has 2 cases: `has bullets` and `has no bullet`.

The `lock` and `unlock` methods each only has one case.

{% highlight abc %}
describe a Gun
  describe shoot
    when has bullets
    when has no bullet

  describe lock
  describe unlock
{% endhighlight %}

In the `has bullets` case of the `shoot` method, we have 2 sub-cases: `is unlocked` and `is locked`.

{% highlight abc %}
describe a Gun
  describe shoot
    when has bullets
      when is unlocked
      when is locked

    when has no bullet

  describe lock
  describe unlock
{% endhighlight %}

And each case will have different behavior.

{% highlight abc %}
describe a Gun
  describe shoot
    when has bullets
      when is unlocked
        it can shoot

      when is locked
        it cannot shoot

    when has no bullet
      it cannot shoot

  describe lock
   it locks the Gun

  describe unlock
   it unlocks the Gun
{% endhighlight %}

Again, from looking at this description, you quickly understand what a Gun can do in various cases. This saves you a lot of time and headache when reading tests.

## Translating into Swift

First, follow this [guide](/ios/testing/2016/07/24/how-to-setup-testing-for-new-ios-project/#step-3-setup-unit-tests) (from step 1 to step 3) to setup `cocoapods` and add the [Quick](https://github.com/Quick/Quick) pod.

Add a new file called `GunSpec.swift` to your test target. Notice that unit tests written using `BDD` style should have the suffix `Spec` instead of `Tests` as the naming convention.

Your `GunSpec` class must subclass from `QuickSpec` and override the `spec` method.

{% highlight swift %}
import Quick

class GunSpec : QuickSpec {
  override func spec() {
    super.spec()
  }
}
{% endhighlight %}

Now we're going to describe the Gun's behaviors again. But this time, we do it with Swift.

First, we describe a Gun.

(Actually the line `class GunSpec : QuickSpec` already tells this. We don't need to write any more code)

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

The `shoot` method has 2 cases: `has bullets` and `has no bullet`.

The `lock` and `unlock` methods each only has one case. (Note that when translate to Swift code, we don't use the word `when`. We use `context` instead)

{% highlight swift %}
override func spec() {
  super.spec()

  describe("shoot") {
    context("has bullets") {

    }

    context("has no bullet") {

    }
  }

  describe("lock") {

  }

  describe("unlock") {

  }
}
{% endhighlight %}

In the context `has bullets` of the `shoot` method, we have 2 sub-cases: `is unlocked` and `is locked`

{% highlight swift %}
override func spec() {
  super.spec()

  describe("shoot") {
    context("has bullets") {
      context("is unlocked") {

      }

      context("is locked") {

      }
    }

    context("has no bullets") {

    }
  }

  describe("lock") {

  }

  describe("unlock") {

  }
}
{% endhighlight %}

And each case will have different behavior.

{% highlight swift %}
override func spec() {
  super.spec()

  describe("shoot") {
    context("has bullets") {
      context("is unlocked") {
        it("can shoot") {

        }
      }

      context("is locked") {
        it("cannot shoot") {

        }
      }
    }

    context("has no bullet") {
      it("cannot shoot") {

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

At this point, we are done describing the Gun with all funtionalities. It's time to write unit tests for each case. Let's start with the first one.

{% highlight swift %}
//...
describe("shoot") {
  context("has bullets") {
    context("is unlocked") {
      it("can shoot") {

      }
    }

    //...
{% endhighlight %}

We're going to write the 3 steps for this test just like we did in the previous post, which are: `setup`, `execution` and `expectation`

{% highlight swift %}
//...
describe("shoot") {
  context("has bullets") {
    context("is unlocked") {
      it("can shoot") {
        // 1. Setup: create a gun with 1 bullet
        let gun = Gun(bullets: 1)
        // make sure it is unlocked
        gun.isLocked = false
        // 2. Execution: call the shoot method
        gun.shoot()
        // 3. Expectation: expect the gun to be out of bullet
        XCTAssertTrue(gun.bullets == 0, "expect Gun to be out of bullet")
      }
    }

    //...
{% endhighlight %}

Now when we run the test (⌘ + U), it will fail because we don't have the property `isLocked` in the Gun class yet. Open your `Gun.swift` file and add this:

(If you forget where the `Gun.swift` is, please read [part 1](/ios/testing/2016/07/31/how-to-write-unit-tests-in-ios-p1-xctestcase) again)

{% highlight swift %}
class Gun : NSObject {

  var bullets = 0
  // add isLocked property here
  var isLocked = true

  //...
}
{% endhighlight %}

Run the test again. This time it should pass.

Let's move on to the next case. This is the case where the Gun has bullets but it is locked and cannot shoot. Therefore, we expect the number of bullets to remain the same.

{% highlight swift %}
//...
describe("shoot") {
  context("has bullets") {
    //...

    context("is locked") {
      it("cannot shoot") {
        let gun = Gun(bullets: 1)
        // the gun is locked
        gun.isLocked = true
        gun.shoot()
        XCTAssertTrue(gun.bullets == 1, "expect Gun to not shoot anything")
      }
    }

    //...
{% endhighlight %}

This test will fail. We haven't implemented anything related to the `isLocked` logic in the Gun class yet. We will do it now:

{% highlight swift %}
class Gun : NSObject {
  //...

  func shoot() {
    if self.bullets > 0 {
      if !isLocked {
        self.bullets -= 1
      }
    }
  }

  //...
}
{% endhighlight %}

Rerun the tests to make sure they all pass.

Now if we continue down the road with the remaining cases, we will have the full `Spec` for the Gun:

{% highlight swift %}
class GunSpec : QuickSpec {
  override func spec() {
    super.spec()

    describe("shoot") {
      context("has bullets") {
        context("is unlocked") {
          it("can shoot") {
            let gun = Gun(bullets: 1)
            gun.isLocked = false
            gun.shoot()
            XCTAssertTrue(gun.bullets == 0, "expect Gun to be out of bullet")
          }
        }

        context("is locked") {
          it("cannot shoot") {
            let gun = Gun(bullets: 1)
            gun.isLocked = true
            gun.shoot()
            XCTAssertTrue(gun.bullets == 1, "expect Gun to not shoot anything")
          }
        }
      }

      context("has no bullet") {
        it("cannot shoot") {
            let gun = Gun(bullets: 0)
            gun.shoot()
            XCTAssertTrue(gun.bullets == 0, "expect Gun to not shoot anything")
        }
      }
    }

    describe("lock") {
      it("locks the Gun") {
        let gun = Gun(bullets: 2)
        gun.lock()
        XCTAssertTrue(gun.isLocked, "expect Gun to be locked")
      }
    }

    describe("unlock") {
      it("unlocks the Gun") {
        let gun = Gun(bullets: 2)
        gun.unlock()
        // notice that we use assert False here
        XCTAssertFalse(gun.isLocked, "expect Gun to be unlocked")
      }
    }

  }
}
{% endhighlight %}

And the full Gun class with 2 additional methods: `lock` and `unlock`.

{% highlight swift %}
class Gun : NSObject {

  var bullets = 0
  var isLocked = true

  init(bullets: Int) {
    self.bullets = bullets
  }

  func shoot() {
    if self.bullets > 0 {
      if !isLocked {
        self.bullets -= 1
      }
    }
  }

  func lock() {
    isLocked = true
  }

  func unlock() {
    isLocked = false
  }
}
{% endhighlight %}

Although there is still some duplicated code in `GunSpec.swift`, you can refactor it out pretty easily using the `beforeEach` block as stated in [Quick's documentation](https://github.com/Quick/Quick/blob/master/Documentation/en-us/QuickExamplesAndGroups.md#sharing-setupteardown-code-using-beforeeach-and-aftereach).

The important thing is you know the **flow**. You know how to do things the `BDD` way.

## Wrap up

Today we discussed about the problem with traditional way of writing unit tests where we list things out linearly. We then introduced the `BDD` approach of designing tests from the top down so that it's easier to read and navigate through cases.

Now I don't say that you should dump the traditional way and only stick with `BDD`. If you're just starting out, learn how to write a correct unit test first. Organizing your tests comes later.

The idea is that you get to know what options are available and which one to choose for your next project.

In the next article, I'll be talking about `Test-driven Development` and how we can apply it to build a `Calculator` class from the ground up.

Stay tuned.
