---
layout: post
title:  "How to setup testing for new iOS project"
---

## Why should we test?
* **Help minimize bugs**:
  * Because all edge cases are covered in the test suite already
  * Tests help spot regression bugs more quickly. Regression means that you're breaking the old behavior with your new code.

* **Help to document your code**:
  * Have a look at this piece of Swift code:

{% highlight swift %}
func testCreateNewNote() {
  visitNotesScreen()
  clickOn("Create new note")
  fillInNewNoteForm()
  clickOn("Create")
  expectToSeeNewNoteCreatedSuccessfully()
}
{% endhighlight %}

  * Can you guess what story it tells? I bet you will. Tests when written cleanly would read just like English, you won't need any additional documents other than the test suite itself

* **Give you confidence when making changes to the code**:
  * Tests are especially useful when you're working on a large codebase along with several other developers. You don't need to worry about breaking someone else's code (or your own). It's easy to make changes as long as the tests still pass

* **Give you confidence when showing the project to your boss, or your clients**:
  * Project with high test coverage (>90%) tends to have better quality. It will make your boss/clients happy and certainly earn you more money :)

Alright! If you're interested in writing your first tests on iOS, here's a few simple steps that would get you up and running pretty quick:

## Step 1: Create new iOS project
* Create new project
  * Open `Xcode`
  * Choose `Create a new Xcode project`
  * Choose template `iOS` > `Application` > `Single View Application` > `Next`
  * Fill in your project information. Make sure the `Include Unit Tests` option is ticked
  * Choose a path for the project
  * `Xcode` will open the project automatically for you
* Try to `Run` (⌘ + R). It should compile and run smoothly in the simulator

## Step 2: Setup cocoapods
In case you haven't installed [cocoapods](https://cocoapods.org/), you should do it now

* Open `Terminal` or `iTerm` and type in this command

{% highlight shell %}
gem install cocoapods
{% endhighlight %}

* Go to the project you just created

{% highlight shell %}
cd path/to/project
# for example:
# cd ~/Documents/MyAwesomeProject
{% endhighlight %}

* Initialize `cocoapods`

{% highlight shell %}
# this will generate a file called `Podfile` in your project directory
pod init

# open it in your default text editor
open Podfile
{% endhighlight %}

* If everything goes well, you should see some content like this

{% highlight ruby %}
# Uncomment this line to define a global platform for your project
# platform :ios, '9.0'

target 'SetupTestingForiOSProject' do
  # Comment this line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for SetupTestingForiOSProject

  target 'SetupTestingForiOSProjectTests' do
    inherit! :search_paths
    # Pods for testing
  end

end
{% endhighlight %}

## Step 3: Setup unit tests
install pod
for Swift: Quick/Nimble
for ObjC: Specta/Expecta
write your first unit test and run it
for Swift
for ObjC
see log for failed test

## Step 4: Setup UI tests
install pod
KIF
write your first UI test and run it

## Step 5: Create testing schemes
run all tests
run unit tests only
run UI tests only

conclusion
