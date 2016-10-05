---
layout: post
title:  'How to write automated UI tests in iOS'
categories: ios development
tags: ios ui tests automated kif
---

You probably have heard about automated tests before.

There are 2 types of them in iOS:

* Unit test:
  * test a specific case in a class.
  * make sure that the class works independently on its own.
* UI test:
  * is also called Integration test.
  * test user interactions with the app.
  * make sure that all the pieces fit well together.

Both are equally important.

If you only write unit tests, you can only guarantee that each class/component is good by itself. But in reality, it usually breaks when put together with other parts. It would become like this:

![2 unit tests, 0 integration test](/images/how-to-write-automated-ui-tests-in-ios/unittest-integrationtest.gif)

The reverse is also true. Writing only UI tests is not recommended either. Many edge cases can be spotted early using uint test

# Our sample app:

It's a simple note-taking app with the following features:

* Login.
* View list of notes.
* Create new note.
* Update existing note.
* Delete a note.

Here's a gif that runs through all functionalities of the app:

![simple note taking app in ios](/images/how-to-write-automated-ui-tests-in-ios/note-app.gif)

However, our job today is not about how to build such app. What we're gonna do instead is learning how to write automated UI tests for it. That's the exciting part.

# But what does automated UI tests look like anyway?

Let's have a look at this video:

<iframe width="840" height="472" src="https://www.youtube.com/embed/HEGl3Lj4SLE" frameborder="0" allowfullscreen></iframe>

As soon as the test suite is launched, it simulates all kinds of user interactions:

* filling in a text field.
* tapping a button.
* swiping around.

It travels through each screen and test all scenarios in sequence:

* at Login screen, it tests scenario when:
  * entering correct username and password.
  * password field is empty.
  * username field is empty.
  * entering incorrect username or password.
* at Note screen, it tests scenario when:
  * adding a new note.
  * deleting notes.
  * editting an existing note.

**It looks funny**.

Firing up the UI tests and watch it flies through everything is a fun experience.

So the next time someone asks you to show the app, just run the UI tests. He'll be amazed and impressed at the same time. 😜

# Why should we write automated UI tests?

There are some key benefits here:

* **Avoid regression bugs and minimize time spent on manual testing**:

  Actually this is the benefit of automated tests in general.

  When you make some changes in the code, the tests are there to make sure that you don't break anything (regression). And that your app still works as expected.

  If there is no tests, it's dangerous to even make a slightest change. One has to be very careful and do a lot of manual testing to gain back his confidence.

  If there is no tests, we will need the whole QC team to sit together trying to break the app and looking for bugs. Now we don't anymore. UI tests will take care of that pretty nicely.

  I'm not saying that we should totally rely on automated tests. There is always something that tests can't cover. We still need to test the app manually ourselves. However, the time we spend on it is kept to a minimum.

* **Helps test view controllers**:

  Given the design of UIKit framework where everything is tightly coupled (the window, the view, the controller, the app lifecycle), that makes it very hard to write unit tests for view controllers even for the simplest scenario.

  It's still possible but we would have to mock and stub things around, simulate events and stuffs, which turns out to be a ton of work and doesn't prove useful at all.

  Why fight the system when we can ride with it?

  UI test puts us in the role of a user. A user doesn't care about any logic underneath. All he wants to do is tapping buttons and expect to see something appear on the screen. That's it.

* **Helps to document your code**:

  Look at this:

{% highlight swift %}
func testLoginWithWrongCredentials() {
  fillInWrongUsername()
  fillInWrongPassword()
  tapButton("Login")
  expectToSeeAlert("Username or password is incorrect")
  tapButton("OK")
}
{% endhighlight %}

  This is a UI test for the scenario when user tries to login with the wrong credentials.

  Actually you don't need me to tell you what it is. It's obvious.

  If you organize your tests mindfully, you'll get this same effect. People will understand what the app can do just by reading the tests. It's the best documentation.

* **Provides a visual run-through of your app**:

  As I mentioned earlier, this is simply fun.

  It's a nice thing to have and you're gonna love it.

Alright! I hope that gets you a little bit more excited about UI testings. Let's dive deeper.

# Understand UI testing:

A typical UI test might look like this (in pseudo code):

{% highlight abc %}
fill in a text field
tap a button
expect to go to next screen
{% endhighlight %}

As you can see, UI test is composed of actions that a normal user can take. It's like a step-by-step guide with some expectations along the way. The common format is:

{% highlight abc %}
do something
expect something to happen
{% endhighlight %}

For example:

{% highlight abc %}
tap on a table cell
expect it to expand
tap on it again
expect it to collapse
{% endhighlight %}

Another example:

{% highlight abc %}
pull to refresh
expect to see new contents on top
swipe to delete a row
expect the row to be deleted from the table view
{% endhighlight %}

Simple, right?

## But how do we express UI test in Swift?

We're gonna need a framework called [KIF](https://github.com/kif-framework/KIF). It offers a whole lot of APIs to deal with UI interactions. For example:

Fill in *"some random text"* into a text field.

{% highlight swift %}
tester().enterText("some random text", intoViewWithAccessibilityLabel: "my text field")
{% endhighlight %}

Tap a button.

{% highlight swift %}
tester().tapViewWithAccessibilityLabel("my button")
{% endhighlight %}

You may wonder: What is this **accessibility label** thing?

Actually it's just a way to address UI components on the screen.

Or frankly, **accessibility label** is a name you give for each UIView to distinguish among them.

When you say you wanna tap a button, you have to tell the framework which button to tap. Therefore, you assign it an **accessibility label** (let's say *"my button"*) and then tap it:

{% highlight swift %}
tester().tapViewWithAccessibilityLabel("my button")
{% endhighlight %}

Here's a couple more examples:

Expect a view to appear on the screen:

{% highlight swift %}
tester().waitForViewWithAccessibilityLabel("my view")
{% endhighlight %}

Expect a view to disappear from the screen:

{% highlight swift %}
tester().waitForAbsenceOfViewWithAccessibilityLabel("my view")
{% endhighlight %}

Delete the first cell from a UITableView:

{% highlight swift %}
let tableview = tester().waitForViewWithAccessibilityLabel("my tableview") as! UITableView

let firstRow = NSIndexPath(forRow: 0, inSection: 0)

tester().swipeRowAtIndexPath(firstRow, inTableView: tableView, inDirection: .Left)

tester().tapViewWithAccessibilityLabel("Delete")
{% endhighlight %}

## How do we assign accessibility label for UIView?

There are 2 ways that we can do:

1. Using Storyboard:

* Open Storyboard.
* Click on a view you wanna assign accessbility label.
* Choose the **Identity Inspector** tab.

![set accessibility label in storyboard - 1](/images/how-to-write-automated-ui-tests-in-ios/al-storyboard-1.jpg)

* Scroll down to the **Accessibility** section.
* Input the accessbility label you want into the **Label** field. (*"Login - Username"* in this case)

![set accessibility label in storyboard - 2](/images/how-to-write-automated-ui-tests-in-ios/al-storyboard-2.jpg)

For UITableView and UICollectionView, there won't be an **Accessibility** section available. I have no idea why Apple does that but we can workaround anyway:

![set accessibility label in storyboard - uitableview](/images/how-to-write-automated-ui-tests-in-ios/al-tableview.gif)

So basically what we do is we assign a key path that matches the **accessibilityLabel** property of UIView (UITableView inherits from UIView). Then at runtime, it will read value from key path and set its property accordingly.

2. Using code:

If you have a text field that is a reference from the Storyboard:

{% highlight swift %}
@IBOutlet weak var usernameTextField: UITextField!
{% endhighlight %}

Then you can:

{% highlight swift %}
usernameTextField.accessibilityLabel = "my username textfield"
{% endhighlight %}

Although it looks simpler when using code, it is still recommended to set your accessibility label directly from the Storyboard if possible. The best code is no code at all.

# How to setup UI testing?

## Step 1: Import KIF and Nimble using cocoapods

Add `pod 'KIF'` and `pod 'Nimble'` to your *Podfile*. Remember to put them in the test target instead.

{% highlight ruby %}
platform :ios, '9.0'

target 'SimpleNoteTakingApp' do
  use_frameworks!

  #...

  target 'SimpleNoteTakingAppTests' do
    inherit! :search_paths
    pod 'KIF'
    pod 'Nimble'
  end

end
{% endhighlight %}

Open Terminal and run:

{% highlight sh %}
pod install
{% endhighlight %}

## Step 2: Create KIF helper for Swift

Create a *KIF+Extensions.swift* file in your test target.

{% highlight swift %}
import KIF

extension XCTestCase {
  func tester(file : String = #file, _ line : Int = #line) -> KIFUITestActor {
    return KIFUITestActor(inFile: file, atLine: line, delegate: self)
  }

  func system(file : String = #file, _ line : Int = #line) -> KIFSystemTestActor {
    return KIFSystemTestActor(inFile: file, atLine: line, delegate: self)
  }
}

extension KIFTestActor {
  func tester(file : String = #file, _ line : Int = #line) -> KIFUITestActor {
    return KIFUITestActor(inFile: file, atLine: line, delegate: self)
  }

  func system(file : String = #file, _ line : Int = #line) -> KIFSystemTestActor {
    return KIFSystemTestActor(inFile: file, atLine: line, delegate: self)
  }
}
{% endhighlight %}

Create a briding header:

* Create a new temporary Objective-C file in your test target. You can name it to anything.

![](/images/how-to-setup-testing-for-new-ios-project/create-objc-file.png)

* Xcode will ask whether you also want to add a briding header. Hit **Create Bridging Header**.

![](/images/how-to-setup-testing-for-new-ios-project/add-bridging-header.png)

* Delete the temporary ObjC file we just created above. We don't need it anymore.

Import **KIF** from within the bridging header (*SimpleNoteTakingApp-Bridging-Header.h*).

{% highlight objc %}
#import <KIF/KIF.h>
{% endhighlight %}

Create our first UI test (also in the test target). Let's name it *LoginTests.swift*.

{% highlight swift %}
import KIF

class LoginTests : KIFTestCase {

  func testSomethingHere() {
    tester().tapViewWithAccessibilityLabel("hello")
  }

}
{% endhighlight %}

Please note that:

* your UI test must subclass from **KIFTestCase**.
* test methods must begin with the word **test**. For example: testA, testB, testLogin, testSomethingElse, ect...

Now let's run the test to see how it works (Cmd + U).

It should open up the iOS simulator and stop at the login screen. Wait for a couple of seconds. Then fail.

That's because we haven't had any view with accessibility label of **hello** yet. We're gonna fix that later. But for now, we have our first UI test up and running. That's cool.

# Let's start writing UI tests for our elegent note-taking app

## Step 3: Test the first screen

## Step 4: Move step definitions out into extension

## Step 5: Test the second screen

## Step 6: Move common steps into base class

# Wrap up
