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

  If there is no tests, it's dangerous to make even a slightest change. One has to be very careful and do a lot of manual testing to gain back his confidence.

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

Actually it's a way to address UI components on the screen.

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

## Test the login screen:

There are 4 scenarios in the login screen:

### Scenario 1: Empty username and password.

In this case, the user should see an alert telling him that *"Username cannot be empty"*.

Before writing UI test, you should take some time to describe the entire scenario first. It would help to visualize the whole picture and organize your code better.

The recommended way is to use the Gherkin format:

{% highlight gherkin %}
Scenario: Put your scenario name here
  Given some precondition
  When I do something
  Then I expect something to happen
  ...
{% endhighlight %}

In our case, the scenario would look like this:

{% highlight gherkin %}
Scenario: Empty username and password
  Given I clear out the username and password fields
  When I tap "Login" button
  Then I expect to see alert "Username cannot be empty"
{% endhighlight %}

Let's translate it into Swift.

Open *LoginTests.swift* and write the first test:

{% highlight swift %}
func testEmptyUsernameAndPassword() {

}
{% endhighlight %}

Then we perform the first step: clear out both fields.

{% highlight swift %}
func testEmptyUsernameAndPassword() {
  clearOutUsernameAndPasswordFields()
}
{% endhighlight %}

Although this **clearOutUsernameAndPasswordFields** method is not defined yet, you don't have to worry about it. Just write what we want first. We'll fix the compile errors later.

Next step is tapping the "Login" button:

{% highlight swift %}
func testEmptyUsernameAndPassword() {
  clearOutUsernameAndPasswordFields()
  tapButton("Login")
}
{% endhighlight %}

Again, this **tapButton** method is also undefined. We just put it there to structure the test.

Then we do the same thing with the remaining steps:

{% highlight swift %}
func testEmptyUsernameAndPassword() {
  clearOutUsernameAndPasswordFields()
  tapButton("Login")
  expectToSeeAlert("Username cannot be empty")
  tapButton("OK")
}
{% endhighlight %}

Now we have the whole scenario written down in Swift. It's time to fill in the definition for each method.

To clear the text field, we use the KIF method *clearTextFromViewWithAccessibilityLabel*, which is quite self-explanatory. So the **clearOutUsernameAndPasswordFields** would be:

{% highlight swift %}
func clearOutUsernameAndPasswordFields() {
  tester().clearTextFromViewWithAccessibilityLabel("Login - Username")
  tester().clearTextFromViewWithAccessibilityLabel("Login - Password")
}
{% endhighlight %}

The **tapButton** method:

{% highlight swift %}
func tapButton(buttonName: String) {
  tester().tapViewWithAccessibilityLabel(buttonName)
}
{% endhighlight %}

And the **expectToSeeAlert** method:

{% highlight swift %}
func expectToSeeAlert(text: String) {
  tester().waitForViewWithAccessibilityLabel(text)
}
{% endhighlight %}

This is the *LoginTests.swift* at this point:

{% highlight swift %}
import KIF

class LoginTests : KIFTestCase {

  func testEmptyUsernameAndPassword() {
    clearOutUsernameAndPasswordFields()
    tapButton("Login")
    expectToSeeAlert("Username cannot be empty")
    tapButton("OK")
  }

  func clearOutUsernameAndPasswordFields() {
    tester().clearTextFromViewWithAccessibilityLabel("Login - Username")
    tester().clearTextFromViewWithAccessibilityLabel("Login - Password")
  }

  func tapButton(buttonName: String) {
    tester().tapViewWithAccessibilityLabel(buttonName)
  }

  func expectToSeeAlert(text: String) {
    tester().waitForViewWithAccessibilityLabel(text)
  }

}
{% endhighlight %}

Now run your test by pressing Cmd + U.

The simulator will pop up and run over your steps auto-magically.

![first ui test](/images/how-to-write-automated-ui-tests-in-ios/first-ui-test.gif)

The test should pass. (since all functionalities are already implemented)

Let's do a litte refactoring here. Create a new file called *LoginSteps.swift* and move all step methods there.

{% highlight swift %}
extension LoginTests {

  func clearOutUsernameAndPasswordFields() {
    tester().clearTextFromViewWithAccessibilityLabel("Login - Username")
    tester().clearTextFromViewWithAccessibilityLabel("Login - Password")
  }

  func tapButton(buttonName: String) {
    tester().tapViewWithAccessibilityLabel(buttonName)
  }

  func expectToSeeAlert(text: String) {
    tester().waitForViewWithAccessibilityLabel(text)
  }

}
{% endhighlight %}

Then the *LoginTests.swift* would look fairly short and sweet.

{% highlight swift %}
import KIF

class LoginTests : KIFTestCase {

  func testEmptyUsernameAndPassword() {
    clearOutUsernameAndPasswordFields()
    tapButton("Login")
    expectToSeeAlert("Username cannot be empty")
    tapButton("OK")
  }

}
{% endhighlight %}

### Scenario 2: Empty password.

Again, we will start with the scenario design first:

{% highlight gherkin %}
Scenario: Empty password
  Given I clear out the username and password fields
  When I fill in username
  And I tap "Login" button
  Then I expect to see alert "Password cannot be empty"
{% endhighlight %}

Then translate it:

{% highlight swift %}
func testEmptyPassword() {
  clearOutUsernameAndPasswordFields()
  fillInUsername()
  tapButton("Login")
  expectToSeeAlert("Password cannot be empty")
  tapButton("OK")
}
{% endhighlight %}

The **fillInUsername** method is also very straightforward.

{% highlight swift %}
func fillInUsername() {
  tester().enterText("username", intoViewWithAccessibilityLabel: "Login - Username")
}
{% endhighlight %}

Remember to put the step method in *LoginSteps.swift* instead of *LoginTests.swift*. Always keep the test clean.

Run the test. Make sure it passes.

Notice that the 2 tests have a same common step (*clearOutUsernameAndPasswordFields*). We will move it to the **beforeEach** method.

{% highlight swift %}
class LoginTests : KIFTestCase {

  override func beforeEach() {
    clearOutUsernameAndPasswordFields()
  }

  func testEmptyUsernameAndPassword() {
    tapButton("Login")
    expectToSeeAlert("Username cannot be empty")
    tapButton("OK")
  }

  func testEmptyPassword() {
    fillInUsername()
    tapButton("Login")
    expectToSeeAlert("Password cannot be empty")
    tapButton("OK")
  }

}
{% endhighlight %}

Now that we're quite familiar with writing UI tests. Let's move on more quickly.

### Scenario 3: Wrong username or password

The scenario design:

{% highlight gherkin %}
Scenario: Wrong username or password
  Given I clear out the username and password fields
  When I fill in username
  And I fill in wrong password
  And I tap "Login" button
  Then I expect to see alert "Username or password is incorrect"
{% endhighlight %}

The implementation:

{% highlight swift %}
func testWrongUsernameOrPassword() {
  fillInUsername()
  fillInWrongPassword()
  tapButton("Login")
  expectToSeeAlert("Username or password is incorrect")
  tapButton("OK")
}
{% endhighlight %}

Note that the first step (*clearOutUsernameAndPasswordFields*) is already in the **beforeEach** method so we don't need to call it here anymore.

The **fillInWrongPassword** method:

{% highlight swift %}
func fillInWrongPassword() {
  tester().enterText("wrongPassword", intoViewWithAccessibilityLabel: "Login - Password")
}
{% endhighlight %}

### Scenario 4: Correct username and password

The scenario design:

{% highlight gherkin %}
Scenario: Correct username or password
  Given I clear out the username and password fields
  When I fill in username
  And I fill in correct password
  And I tap "Login" button
  Then I expect to go to home screen
{% endhighlight %}

The implementation:

{% highlight swift %}
func testWrongUsernameOrPassword() {
  fillInUsername()
  fillInCorrectPassword()
  tapButton("Login")
  expectToSeeAlert("Username or password is incorrect")
  tapButton("OK")
}
{% endhighlight %}

The **fillInCorrectPassword** method:

{% highlight swift %}
func fillInCorrectPassword() {
  tester().enterText("correctPassword", intoViewWithAccessibilityLabel: "Login - Password")
}
{% endhighlight %}

This is the correct password because I hard-coded it. (a combination of "username" and "correctPassword") 😜

## Test the home screen:

# Wrap up
