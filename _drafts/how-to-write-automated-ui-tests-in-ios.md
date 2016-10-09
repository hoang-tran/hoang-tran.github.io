---
layout: post
title:  'How to write automated UI tests in iOS'
categories: ios development
tags: ios ui tests automated kif
---

You probably have heard about automated tests before. People talk about it a lot these days, especially when the topic is about software quality.

They say that if you don't write any tests for your project, you're in big trouble. It may not affect you at the moment. But in the long run, it would become a huge technical debt.

That's true.

Project with no tests is impossibile to maintain when it gets big and there are multiple developers involved. As you change something in the code, things start to break. You don't even know that it breaks until your boss comes to your desk and starts yelling. You know that feeling, right?

So, it's better to know a thing or two about testing so that you can improve your project quality and also improve yourself as a software engineer.

There are 2 types of automated tests in iOS:

* Unit test:
  * test a specific case in a class.
  * make sure that the class works independently on its own.
* UI test:
  * is also called Integration test.
  * test user interactions with the app.
  * make sure that all classes fit well together.

Both are equally important.

If you only write unit tests and completely ignore UI tests, you'll be trapped in this situation:

![2 unit tests, 0 integration test](/images/how-to-write-automated-ui-tests-in-ios/unittest-integrationtest.gif)

As you can see, each window works fine on its own. But when put together, bad things happen. You can see how mad that man is. 😉

UI tests are simple. Even simpler than unit tests.

Today we're gonna learn to write some very basic UI tests and yet still cover a fully-functional app from start to finish.

# What app will we be working on?

It's a simple note-taking app with the following features:

* Login with username and password.
* View list of notes.
* Create new note.
* Update existing note.
* Delete a note.

Here's a gif that runs through all functionalities of the app:

![simple note taking app in ios](/images/how-to-write-automated-ui-tests-in-ios/note-app.gif)

Look like a piece of cake, right?

However, our job today is not about how to build such app. What we're gonna do instead is learning to write automated UI tests for it, for all screens and functionalities. That's the exciting part.

# But what does automated UI tests look like?

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
* at Home screen, it tests scenario when:
  * adding a new note.
  * deleting notes.
  * editting an existing note.

I don't know about you but I have to admit that **it looks funny**.

Firing up the UI tests and watch it flies through everything is a fun experience. So next time when someone asks you to show your app, just run the UI tests. He'll be amazed and impressed at the same time. 😜

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
func testWrongUsernameOrPassword() {
  fillInUsername()
  fillInWrongPassword()
  tapButton("Login")
  expectToSeeAlert("Username or password is incorrect")
  tapButton("OK")
}
{% endhighlight %}

  This is a UI test for scenario when user tries to login with wrong credentials (username or password).

  Actually you don't need me to tell you what that is. It's obvious.

  You'll quickly understand what the app can do just by reading the tests. It's the best documentation.

* **Provides a visual run-through of your app**:

  As I mentioned earlier, this is simply fun. It's a nice thing to have and you're gonna love it.

  Seriously, you're gonna love it for sure.

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

In UI tests, we care about user interactions much more than the underlying code structure. We focus heavily on what a user can do and what he'll see on the screen. All the background stuffs can be ignored.

## How do we express UI test in Swift?

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

Expect a view (with accessibility label of *"my view"*) to appear on the screen:

{% highlight swift %}
tester().waitForViewWithAccessibilityLabel("my view")
{% endhighlight %}

Expect a view (with accessibility label of *"my view"*) to disappear from the screen:

{% highlight swift %}
tester().waitForAbsenceOfViewWithAccessibilityLabel("my view")
{% endhighlight %}

Get a reference to a view with accessibility label of *"my view"*:

{% highlight swift %}
let myView = tester().waitForViewWithAccessibilityLabel("my view") as! UIView
{% endhighlight %}

Swipe left on view with accessibility label of *"my view"*:

{% highlight swift %}
tester().swipeViewWithAccessibilityLabel("my view", inDirection: .Left)
{% endhighlight %}

For full list of supported UI interactions in KIF, you can read [here](https://github.com/kif-framework/KIF/blob/master/Classes/KIFUITestActor.h).

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

For UITableView and UICollectionView, there won't be an **Accessibility** section available. I have no idea why Apple does that. However, we can workaround anyway:

![set accessibility label in storyboard - uitableview](/images/how-to-write-automated-ui-tests-in-ios/al-tableview.gif)

So basically what we do is we assign a key path that matches the **accessibilityLabel** property of UITableView. Then at runtime, it will read value from key path and set its property accordingly.

Another thing to note is that: for UIButton or UILabel, it has a default accessibility label which is equal to its **text** property. Let's say you have a button with the text *"click me"*, then its accessibility label is also *"click me"*. You don't need to set it again.

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

# Prepare the project:

First, download the project here.

Run it. Make sure it compiles fine.

Play around with the app to get a sense of what we're gonna do next.

# How to setup KIF for UI testing?

## Step 1: Import KIF and Nimble using cocoapods.

Add `pod 'KIF'` and `pod 'Nimble'` to your *Podfile*. Remember to put them in the test target.

Nimble is a framework to help you better express your expectation in test. I once wrote an article about it [here](http://hoangtran.me/ios/testing/2016/08/09/write-better-unit-test-assertion-with-nimble/).

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

## Step 2: Create KIF helper for Swift.

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

## Step 3: Create a briding header.

Create a new temporary Objective-C file in your test target. You can name it to anything.

![](/images/how-to-setup-testing-for-new-ios-project/create-objc-file.png)

Xcode will ask whether you also want to add a briding header. Hit **Create Bridging Header**.

![](/images/how-to-setup-testing-for-new-ios-project/add-bridging-header.png)

Delete the temporary ObjC file we just created above. We don't need it anymore.

Import **KIF** from within the bridging header (*SimpleNoteTakingApp-Bridging-Header.h*).

{% highlight objc %}
#import <KIF/KIF.h>
{% endhighlight %}

## Step 4: Create our first UI test.

Create a new file in the test target and name it to `LoginTests.swift`.

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

That's because we haven't had any view with accessibility label of **hello** yet. We're gonna fix that later. But for now, our first UI test is up and running. It's cool.

# Let's start writing UI tests for our elegent note-taking app

## Test the login screen:

There are 4 scenarios in the login screen:

### Scenario 1: Empty username and password.

In this case, the user should see an alert telling him that *"Username cannot be empty"*.

Now before we move on to the nitty gritty, I want you to take some time and design the scenario first: what step will you perform? And what do you expect out of it?

* First, we would like our username and password fields to be cleared out.
* Then we tap the Login button.
* And we expect to see an alert that says "Username cannot be empty".

Actually there's a better way to express these steps, using Gherkin format. Here's the basic structure of a scenario:

{% highlight gherkin %}
Scenario: Your scenario name
  Given some pre-condition
  When I do something
  Then I expect something to happen
  ...
{% endhighlight %}

In our case, the scenario would become:

{% highlight gherkin %}
Scenario: Empty username and password
  Given I clear out the username and password fields
  When I tap "Login" button
  Then I expect to see alert "Username cannot be empty"
{% endhighlight %}

Although this is just something we draft on a paper, or directly in our mind, it's very close to the human language. Everyone should be able to read and understand.

Let's translate it into Swift.

Open `LoginTests.swift` and write our first test. Again, test method name must begin with prefix **test**.

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

To clear the text field, we use the KIF method **clearTextFromViewWithAccessibilityLabel**, which is quite self-explanatory. So the **clearOutUsernameAndPasswordFields** would be:

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

This is the `LoginTests.swift` at this point:

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

Let's do a litte refactoring here. Create a new file called `LoginSteps.swift` and move all step methods there.

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

Then the `LoginTests.swift` would look fairly short and sweet.

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
  tester().enterText("appcoda", intoViewWithAccessibilityLabel: "Login - Username")
}
{% endhighlight %}

Remember to put the step method in `LoginSteps.swift` instead of `LoginTests.swift`. Always keep the test clean.

Run the test. Make sure it passes.

Notice that the 2 tests now have a same common step (**clearOutUsernameAndPasswordFields**). We will move it to the **beforeEach** method. That's where you put things you wanna execute first before each test runs.

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

Note that the first step (**clearOutUsernameAndPasswordFields**) is already in the **beforeEach** method so we don't need to call it here anymore.

The **fillInWrongPassword** method:

{% highlight swift %}
func fillInWrongPassword() {
  tester().enterText("wrongPassword", intoViewWithAccessibilityLabel: "Login - Password")
}
{% endhighlight %}

### Scenario 4: Correct username and password

The scenario design:

{% highlight gherkin %}
Scenario: Correct username and password
  Given I clear out the username and password fields
  When I fill in username
  And I fill in correct password
  And I tap "Login" button
  Then I expect to go to home screen
{% endhighlight %}

The implementation:

{% highlight swift %}
func testCorrectUsernameAndPassword() {
  fillInUsername()
  fillInCorrectPassword()
  tapButton("Login")
  expectToGoToHomeScreen()
}
{% endhighlight %}

The **fillInCorrectPassword** method:

{% highlight swift %}
func fillInCorrectPassword() {
  tester().enterText("correctPassword", intoViewWithAccessibilityLabel: "Login - Password")
}
{% endhighlight %}

This is the correct password because I hard-coded it. (a combination of "appcoda" and "correctPassword") 😜

About the **expectToGoToHomeScreen**, how do we know if we've already transitioned into another screen?

Well, we do it by:

* expect the UI elements in the login screen to disappear.
* expect to see UI elements of the Home screen.

{% highlight swift %}
func expectToGoToHomeScreen() {
  // expect login screen to disappear
  tester().waitForAbsenceOfViewWithAccessibilityLabel("Login - Username")
  tester().waitForAbsenceOfViewWithAccessibilityLabel("Login - Password")
  tester().waitForAbsenceOfViewWithAccessibilityLabel("Login")

  // expect to see Home screen
  tester().waitForViewWithAccessibilityLabel("No notes")
  tester().waitForViewWithAccessibilityLabel("Add note")
}
{% endhighlight %}

## Test the home screen:

Create file `HomeTests.swift`.

{% highlight swift %}
import KIF

class HomeTests: KIFTestCase {

}
{% endhighlight %}

And its corresponding `HomeSteps.swift`.

{% highlight swift %}
extension HomeTests {

}
{% endhighlight %}

Now that we have more than 1 test class (`LoginTests` and `HomeTests`). There will be common step methods that we're gonna reuse between them. Let's create a base class called `BaseUITests.swift`.

{% highlight swift %}
class BaseUITests: KIFTestCase {

}
{% endhighlight %}

Then make *LoginTests* and *HomeTest* inherit from it.

{% highlight swift %}
// in LoginTests.swift
class LoginTests: BaseUITests { ... }

// in HomeTests.swift
class HomeTests: BaseUITests { ... }
{% endhighlight %}

Create another file called `CommonSteps.swift`. Move all common step methods there:

{% highlight swift %}
extension BaseUITests {
  // move common steps here
}
{% endhighlight %}

So whenever you write a step method, be sure to put it into the right place:

* steps that are widely used among test classes should belong to `CommonSteps.swift`.
* steps that are only used in a specific screen should go to its corresponding step file. (Ex: `LoginSteps.swift` or `HomeSteps.swift`)

{% highlight swift %}
// in CommonSteps.swift
extension BaseUITests {
  // common steps
}

// in LoginSteps.swift
extension LoginTests {
  // step specific for Login screen
}

// in HomeSteps.swift
extension HomeTests {
  // step specific for Home screen
}
{% endhighlight %}

The Home screen is where we create/edit/delete our notes so there're a lot of database interactions happening.

We don't wanna put a bunch of test records into our production database every time we run the UI tests. Instead, we will create a test database and use it in our testing environment.

Since I'm using Realm as the database layer. It only takes me 1 line of code to setup a test database:

{% highlight swift %}
func useTestDatabase() {
  Realm.Configuration.defaultConfiguration.inMemoryIdentifier = "put any name here"
}
{% endhighlight %}

This will create a Realm database in memory and it only exists for as long as the tests are still running.

Your project may use a different database technology (CoreData, FMDB, SQLite) but the idea is the same. You create a test database file and direct all test records into it. Your main database file is safe.

We're gonna put this database setup into the **beforeAll** block so that it is executed only once.

{% highlight swift %}
class HomeTests: KIFTestCase {

  override func beforeAll() {
    useTestDatabase()
  }

}
{% endhighlight %}

Now we're ready for the 4 scenarios of the Home screen.

### Scenario 1: When there's no notes, display label "No notes"

Since Home is not the initial screen. We must have a step to go to Home before we can do anything else.

{% highlight gherkin %}
Scenario: If there's no notes, display label "No notes"
  Given I have no notes
  When I visit home screen
  Then I expect to see label "No notes"
  And I expect not to see the note list
{% endhighlight %}

The implementation:

{% highlight swift %}
func testNoNotes() {
  haveNoNotes()
  visitHomeScreen()
  expectToSeeLabel("No notes")
  expectNotToSeeNoteList()
}
{% endhighlight %}

For the **haveNoNotes**, we have to delete all records from database:

{% highlight swift %}
func haveNoNotes() {
  let realm = try! Realm()
  try! realm.write {
    realm.deleteAll()
  }
}
{% endhighlight %}

For **visitHomeScreen**, it's a little bit tricky because after the previous test finishes, you may not know which screen you're on. You may be at the Login screen, Home screen, or any other screen.

It's hard to go anywhere when you have no idea where you are at the moment, right?

However, if we put ourselves at the initial (Login) screen, there is always at least 1 way to reach every screens in the app.

Therefore, the solution is that no matter where you are, go back to the initial screen first, then you can proceed to other screens very easily.

But how to do that?

The first thing to do is grabbing a reference to the root view controller:

{% highlight swift %}
let rootViewController = UIApplication.sharedApplication().keyWindow?.rootViewController
{% endhighlight %}

Then depends on your app architecture, we will proceed in different ways. In my case, it's a navigation controller on top of everything. So all I need to do is popping back to first controller in the navigation stack:

{% highlight swift %}
func backToRoot() {
  if let rootViewController = UIApplication.sharedApplication().keyWindow?.rootViewController as? UINavigationController {
    rootViewController.popToRootViewControllerAnimated(false)
  }
}
{% endhighlight %}

With this in place, we'll put it in the **beforeEach** method so every time a test gets executed, it has to go back to root first.

After that, it's recommended to clear out the database to ensure a fresh start for subsequent tests. We're gonna move the **haveNoNotes** step into the **beforeEach** method too.

{% highlight swift %}
class HomeTests: KIFTestCase {

  override func beforeAll() {
    useTestDatabase()
  }

  override func beforeEach() {
    backToRoot()
    haveNoNotes()
  }

  func testNoNotes() {
    visitHomeScreen()
    expectToSeeLabel("No notes")
    expectNotToSeeNoteList()
  }

}
{% endhighlight %}

The **visitHomeScreen** method:

{% highlight swift %}
func visitHomeScreen() {
  fillInUsername()
  fillInCorrectPassword()
  tapButton("Login")
}
{% endhighlight %}

The **expectToSeeLabel** method:

{% highlight swift %}
func expectToSeeLabel(label: String) {
  tester().waitForViewWithAccessibilityLabel(text)
}
{% endhighlight %}

The **expectNotToSeeNoteList** method:

{% highlight swift %}
func expectNotToSeeNoteList() {
  tester().waitForAbsenceOfViewWithAccessibilityLabel("Note - Table View")
}
{% endhighlight %}

### Scenario 2: Create new note.

The scenario design:

{% highlight gherkin %}
Scenario: Create new note
  Given I have no notes
  When I visit home screen
  And I tap "Add note" button
  Then I expect the Create button to be disabled
  When I fill in the note title with "new note"
  Then I expect the Create button to be enabled
  And I fill in the note body with "new body"
  And I tap "Create" button
  Then I expect to see note with title "new note" and body "new body" at row 0
  And I expect the number of notes in list to be 1
{% endhighlight %}

There's little bit of logic here. The create button is only enabled if there's some text in the title field, otherwise it will be disabled.

The implementation:

{% highlight swift %}
func testCreateNewNote() {
  haveNoNotes()
  visitHomeScreen()
  tapButton("Add note")
  expectTheCreateButtonToBeDisabled()
  fillInNoteTitle("new note")
  expectTheCreateButtonToBeEnabled()
  fillInNoteBody("new body")
  tapButton("Create")
  expectToSeeNoteWithTitle("new note", body: "new body", atRow: 0)
  expectNumberOfNotesInListToEqual(1)
}
{% endhighlight %}

For the **expectTheCreateButtonToBeDisabled** method, we have to:

* First, get a reference to the Create button.
* Then assert its property using Nimble. (If you don't know what Nimble is, read [here](http://hoangtran.me/ios/testing/2016/08/09/write-better-unit-test-assertion-with-nimble/))

{% highlight swift %}
func expectTheCreateButtonToBeDisabled() {
  let createButton = tester().waitForViewWithAccessibilityLabel("Create") as! UIButton
  expect(createButton.enabled) == false
}
{% endhighlight %}

Same with **expectTheCreateButtonToBeEnabled**:

{% highlight swift %}
func expectTheCreateButtonToBeEnabled() {
  let createButton = tester().waitForViewWithAccessibilityLabel("Create") as! UIButton
  expect(createButton.enabled) == true
}
{% endhighlight %}

The **fillInNoteTitle** and **fillInNoteBody** are easy. We just need to fill in the field with some texts.

The **expectToSeeNoteWithTitle** method can be done using the same approach too.

* Get a reference to the cell.
* Assert for its property.

{% highlight swift %}
func expectToSeeNoteWithTitle(title: String, body: String, atRow row: NSInteger) {
  let indexPath = NSIndexPath(forRow: row, inSection: 0)
  let noteCell = tester().waitForCellAtIndexPath(indexPath, inTableViewWithAccessibilityIdentifier: "Note - TableView")
  expect(noteCell.textLabel?.text) == title
  expect(noteCell.detailTextLabel?.text) == body
}
{% endhighlight %}

The **expectNumberOfNotesInListToEqual** method:

{% highlight swift %}
func expectNumberOfNotesInListToEqual(count: Int) {
  let noteTableView = tester().waitForViewWithAccessibilityLabel("Note - TableView") as! UITableView
  expect(noteTableView.numberOfRowsInSection(0)) == count
}
{% endhighlight %}

### Scenario 3: Edit a note

The scenario design:

{% highlight gherkin %}
Scenario: Edit a note
  Given I have 3 notes
  When I visit home screen
  And I tap on note at row 1
  And I update note title to "updated note"
  And I update note body to "updated body"
  And I tap "Update" button
  Then I expect to see note with title "updated note" and body "updated body" at row 1
{% endhighlight %}

The implementation:

{% highlight swift %}
func editANote() {
  have3Notes()
  visitHomeScreen()
  tapOnNoteAtRow(1)
  updateNoteTitleTo("updated note")
  updateNoteBodyTo("updated body")
  tapButton("Update")
  expectToSeeNoteWithTitle("updated note", body: "updated body", atRow: 1)
}
{% endhighlight %}

The **have3Notes** method: we add 3 records to Realm database.

{% highlight swift %}
func have3Notes() {
  let realm = try! Realm()
  try! realm.write {
    for i in 0...2 {
      let note = Note()
      note.title = "title \(i)"
      note.body = "body \(i)"
      realm.add(note)
    }
  }
}
{% endhighlight %}

The **tapOnNoteAtRow** method:

{% highlight swift %}
func tapOnNoteAtRow(row: Int) {
  let indexPath = NSIndexPath(forRow: row, inSection: 0)
  tester().tapRowAtIndexPath(indexPath, inTableViewWithAccessibilityIdentifier: "Note - TableView")
}
{% endhighlight %}

### Scenario 4: Delete notes

The scenario design:

{% highlight gherkin %}
Scenario: Delete notes
  Given I have 3 notes
  When I visit home screen
  When I delete a note
  Then I expect the number of note in list to be 2
  When I delete a note
  Then I expect the number of note in list to be 1
  When I delete a note
  Then I expect to see label "No notes"
{% endhighlight %}

The implementation:

{% highlight swift %}
func deleteNotes() {
  have3Notes()
  visitHomeScreen()
  deleteANote()
  expectNumberOfNotesInListToEqual(2)
  deleteANote()
  expectNumberOfNotesInListToEqual(1)
  deleteANote()
  expectToSeeLabel("No notes")
}
{% endhighlight %}

The **deleteANote** method:

{% highlight swift %}
func deleteANote() {
  let noteTableView = tester().waitForViewWithAccessibilityLabel("Note - TableView") as! UITableView
  let indexPath = NSIndexPath(forRow: 0, inSection: 0)
  tester().swipeRowAtIndexPath(indexPath, inTableView: noteTableView, inDirection: .Left)
  tapButton("Delete")
}
{% endhighlight %}

# Wrap up

UI tests are easy to learn but yield a lot of benefits.

It might take a couple of days to familiarize yourself with KIF and how to work with accessibility labels. But then after that, you're unstoppable.

Your UI tests are going to cover all user scenarios across the app. Whenever you change something in the code, you're good to go as long as the tests are still passing.

The only downside I found is that it really take time to run the whole UI test suite as your app grows. For this note taking app, it take me around 80 seconds to run through everything, considering I'm using an old hackintosh with Intel Core i3.

For bigger real-world app, it may take much longer, from a couple of minutes to even hours. But the good news is we can delegate the test running task to another service called **Continuous Integration**. That deserves another blog post for itself.

You can download the full project (with all UI tests) here.

If you have any questions regarding UI tests, feel free to put a comment down below. I would love to hear more from you guys. Thanks and have a good day.
