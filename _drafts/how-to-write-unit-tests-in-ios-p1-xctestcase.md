---
layout: post
title:  'How to write unit tests in iOS Part 1: XCTestCase'
categories: ios testing
tags: ios objc swift testing unittest
---

## What is unit test?

Unit test is basically just a test at the `unit` level. Or to put simply, you test each `class` in isolation

A unit test usually has 4 steps:

1. **Setup**:
  * This is where you initialize the `class` you want to test

2. **Execution**:
  * Call method on the `class` or `instance of the class`

3. **Expectation**:
  * Set some expectations for the outcome

4. **Cleanup**: (optional)
  * Clean up anything that may affect other subsequent tests

Let's look at some example unit tests here (in pseudo code):

Test the `shoot` method on `Gun` class:

* Case 1: Gun CAN shoot if it has bullet(s)

{% highlight abc %}
1. Setup: create a gun with 1 bullet
2. Execution: shoot
3. Expectation: expect gun to be out of bullet
{% endhighlight %}

* Case 2: Gun CANNOT shoot if it has no bullet

{% highlight abc %}
1. Setup: create a gun with no bullet
2. Execution: shoot
3. Expectation: expect gun not to shoot anything
{% endhighlight %}

Test the `reload` method on `Gun` class:

{% highlight abc %}
1. Setup: create a gun with no bullet
2. Execution: reload the gun
3. Expectation: expect the gun to be full of bullets
{% endhighlight %}

As you can see, for each case, we only test a very small part of the object. That's why it is called `unit` test.

A `class` may have multiple `methods`. Each `method` will have multiple unit tests to cover all of its cases

## So how does all of this translate into iOS?

Xcode does provide a way to write unit tests natively through the `XCTestCase` class. It provides a variety of assertion methods to help us describe our `expectation` for the test. Let's dive in and see what it looks like.

First, add a new file to your test target. (if your project name is `MyAwesomeProject`, then your test target's name would be `MyAwesomeProjectTests`)

![](/images/how-to-write-unit-tests-in-ios/add-new-file.png)

Choose the `Unit Test Case Class` template > `Next`

![](/images/how-to-write-unit-tests-in-ios/choose-unit-test.png)

Make sure you subclass from `XCTestCase`.

![](/images/how-to-write-unit-tests-in-ios/name-unit-test.png)

Originally, your class will look like this:

<script src="https://gist.github.com/hoang-tran/5e8c1ad8b18ffaa2d9fa261e7007fcdf.js"></script>

For simplicity, we'll remove everything and just put our single test there. Notice that your test method **MUST** begin with the prefix `test`. (Ex: `testSomething`, `testFirst`, `testTheWorld`)

<script src="https://gist.github.com/hoang-tran/d08342a5912a9cabc59662d520318237.js"></script>

Let's write our first `expectation`.

{% highlight swift %}
func testSomething() {
  XCTAssert(1 + 1 == 2)
}
{% endhighlight %}

This reads as `expect the boolean expression (1 + 1 == 2) to be true` or `expect 1 + 1 to equal to 2`. When you run this test (`⌘ + U`), it should pass. But when you change it to:

{% highlight swift %}
func testSomething() {
  XCTAssert(1 + 1 == 3)
}
{% endhighlight %}

This will fail.

![](/images/how-to-write-unit-tests-in-ios/test-fail.png)

When the test fails, we know exactly which line it is located but we don't know why it failed, we only see a general statement `XCTAssertTrue failed`, which is not very useful. Let's fix that by adding another argument for error message.

{% highlight swift %}
func testSomething() {
  XCTAssert(1 + 1 == 3, "1 + 1 should equal to 2")
}
{% endhighlight %}

Now it looks better when it fails.

![](/images/how-to-write-unit-tests-in-ios/test-fail-reason.png)

Besides `XCTAssert`, there are some other methods you can use for assertion, such as:

* `XCTAssertTrue(booleanExpression, errorMessage)`: **pass** when `booleanExpression` is true, otherwise **fail** with `errorMessage`. This one is equipvalent to `XCTAssert`.

* `XCTAssertFalse(booleanExpression, errorMessage)`: **pass** when `booleanExpression` is false, otherwise **fail** with `errorMessage`.

* `XCTAssertNil(object, errorMessage)`: **pass** when `object` is nil, otherwise **fail** with `errorMessage`.

* `XCTAssertEqual(object1, object2, errorMessage)`: **pass** when `object1` is equal to `object2`, otherwise **fail** with `errorMessage`.

* `XCTAssertThrowsError(expression, errorMessage, errorHandler)`: **pass** when `expression` throws exception while evaluated, otherwise **fail** with `errorMessage`. We can also use the `errorHandler` to assert for the exception thrown.
  For example:

{% highlight swift %}
enum Error: ErrorType {
  case SomeExpectedError
  case SomeUnexpectedError
}

func functionThatThrows() throws {
    throw Error.SomeExpectedError
}

XCTAssertThrowsError(try functionThatThrows(), "some message") { (error) in
  XCTAssertEqual(error as? Error, Error.SomeExpectedError)
}
{% endhighlight %}

This is just some methods that I use often. For the full list, please refer to Apple documenation.

Let's write real unit tests for the `Gun` example we talked about earlier.

Test the `shoot` method on `Gun` class:

* Case 1: Gun CAN shoot if it has bullet(s).

{% highlight swift %}
func testGunCanShootIfItHasBullets() {
  // 1. Setup: create a gun with 1 bullet
  let gun = Gun(bullets: 1)
  // 2. Execution: shoot
  gun.shoot()
  // 3. Expectation: expect Gun to be out of bullet
  XCTAssertTrue(gun.bullets == 0, "expect Gun to be out of bullet")
}
{% endhighlight %}

* Case 2: Gun CANNOT shoot if it has no bullet.

{% highlight swift %}
func testGunCannotShootIfItHasNoBullet() {
  // 1. Setup: create a gun with no bullet
  let gun = Gun(bullets: 0)
  // 2. Execution: shoot
  gun.shoot()
  // 3. Expectation: expect Gun to not shoot anything
  XCTAssertTrue(gun.bullets == 0, "expect the number of bullets to remain the same")
}
{% endhighlight %}

Both of these 2 tests will fail because we haven't implemented the `Gun` class yet. Let's go ahead and do it.

Add a new file called `Gun.swift` inside your main target (not inside the test target). We will do a very simple implementation here:

<script src="https://gist.github.com/hoang-tran/4987fa39dd28f6eb89f4d3b52c985623.js"></script>

Go to the test class and import your main target. Without this step, all of your classes in the main target won't be visible to the tests.

{% highlight swift %}
// add this line to the top of the file
@testable import MyAwesomeProject
{% endhighlight %}

The `@testable` means that you don't need to declare your classes and methods as `public` anymore. All of them will be visible in the test target eventually.

Now when you run the tests (`⌘ + U`), they should all pass. Here is the test class at this point:

<script src="https://gist.github.com/hoang-tran/901149f12321a47569519defae21be1e.js"></script>

There is still the `reload` method to test but I guess you can easily do it on your own since it's quite similar to what we just did.

### Wrap up

Something someting
