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

  For the rest of this tutorial, let's just assume that your project name is `MyAwesomeProject` and it is located at `~/Documents/MyAwesomeProject`

## Step 2: Setup cocoapods
In case you haven't installed [cocoapods](https://cocoapods.org/), you should do it now

* Open `Terminal` or `iTerm` and type in this command

{% highlight shell %}
gem install cocoapods
{% endhighlight %}

* Go to the project you just created

{% highlight shell %}
cd ~/Documents/MyAwesomeProject
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
* Declare necessary pods for unit tests in your `Podfile`
  * For `Swift`: we'll use:
    * [Quick](https://github.com/Quick/Quick): a behavior-driven development framework for Swift
    * [Nimble](https://github.com/Quick/Nimble): to express the expected outcomes of Swift expressions

{% highlight ruby %}
platform :ios, '9.0'

target 'MyAwesomeProject' do
  use_frameworks!

  target 'MyAwesomeProjectTests' do
    inherit! :search_paths
    pod 'Quick'
    pod 'Nimble'
  end
end
{% endhighlight %}

  * For `ObjC`: since we don't want to use dynamic framework, we'll remove the `use_frameworks!` line. And the pods we need are:
    * [Specta](https://github.com/specta/specta): A light-weight TDD / BDD framework for Objective-C & Cocoa
    * [Expecta](https://github.com/specta/expecta/): A Matcher Framework for Objective-C/Cocoa

{% highlight ruby %}
platform :ios, '9.0'

target 'MyAwesomeProject' do
  target 'MyAwesomeProjectTests' do
    inherit! :search_paths
    pod 'Specta'
    pod 'Expecta'
  end
end
{% endhighlight %}

* Install all the pods we just declared

{% highlight shell %}
pod install
{% endhighlight %}

* Open the generated `.xcworkspace` file

{% highlight shell %}
open MyAwesomeProject.xcworkspace
{% endhighlight %}

* Write your first unit test
  * For `Swift`:
    * Create a new file in the test target and name it to something like `MyFirstSpec.swift`
    * Copy the content of this `gist` and put into your newly created file

<script src="https://gist.github.com/hoang-tran/8b7c0966b8d68b0d150fe33abd30c292.js"></script>

  * For `ObjC`:
    * Create a new file in the test target and name it to something like `MyFirstSpec.m`
    * Copy the content of this `gist` and put into your newly created file

<script src="https://gist.github.com/hoang-tran/9ea175f0a52c4aec45b5ee2601edcc5f.js"></script>

* Run your tests by pressing ⌘ + U
* It should fail at the appropriate line

![](/images/how-to-setup-testing-for-new-ios-project/MyFirstSpec-fail.png)

* With this as a template, you can write more unit tests very easily

## Step 4: Setup UI tests
install pod
KIF
write your first UI test and run it

## Step 5: Create testing schemes
run all tests
run unit tests only
run UI tests only

conclusion
