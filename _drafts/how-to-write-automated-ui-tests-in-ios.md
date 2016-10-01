---
layout: post
title:  'How to write automated UI tests in iOS'
categories: ios development
tags: ios ui tests automated kif
---

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

## Accessibility Label:

### How to set it?

1. From code:

2. From Storyboard:

### How to inspect it?

# Let's do it!

## Step 1: Import KIF using cocoapods

## Step 2: Create KIF helper for Swift

## Step 3: Test the first screen

## Step 4: Move step definitions out into extension

## Step 5: Test the second screen

## Step 6: Move common steps into base class

# Some common scenarios you may encounter:

## Inspect properties of a control:

## Working with TabBar:

## Swiping around:

## Dealing with alert and action sheet:

## Pull to refresh:

## Working with textfields:

## Working with UITableView and UICollectionView:

## Wait for sometime:

## Working with photos and gallery:

## Dealing with network requests:

## Dealing with permissions:

# Wrap up
