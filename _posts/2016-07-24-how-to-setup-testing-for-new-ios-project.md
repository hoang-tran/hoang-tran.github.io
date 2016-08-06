---
layout: post
title:  How to setup testing for new iOS project
meta_description: Learn how to setup a testing environment on iOS, for both ObjC and Swift
categories: ios testing
tags: ios objc swift testing setup unittest uitest test
---

App (or any product in general), before getting released to users, needs to be tested carefully.

When talking about testings, you might think of someone who sits for hours playing around with the app, trying to hit all the buttons, swiping in all directions or even shaking the phone like crazy in hope of finding something unusual to report back to the developers.

For big app, it would take days for the tester team to go through all the functionalities. Even after that, some developers might come in and make changes to the code and that would restart the whole testing process all over again.

That's the reason why more and more companies are starting to write automation tests for their projects. The benefits are huge. To name a few, tests will:

**Provide instant feedback on where the code fails**:

* It only take a few seconds to several minutes (one hour at max) to run all of your automation tests.
* When tests fail, we will quickly know what's wrong just by looking at the tests log instead of having a raging customers/boss to yell at our face.
* Although we still need to do manually testing sometimes, automation tests still save you a lot of development time by providing extremely fast feedback loop.

**Give you confidence when making changes to the code**:

* Tests are especially useful when you're working on a large codebase along with several other developers. You don't need to worry about breaking someone else's code (or your own). It's easy to make changes as long as the tests still pass.

**Give you confidence when showing the project to your boss, or your clients**:

* Project with high test coverage (>90%) tends to have better quality. It will make your boss/clients happy and certainly earn you more money.

**Help to document your code**:

* Have a look at this piece of test:

{% highlight swift %}
func testCreateNewNote() {
  visitNotesScreen()
  clickOn("Create new note")
  fillInNewNoteForm()
  clickOn("Create")
  expectToSeeNewNoteCreatedSuccessfully()
}
{% endhighlight %}

* Can you guess what story it tells? I bet you will. Tests when written cleanly would read just like English, you won't need any additional documents other than the test suite itself.

Alright! If that get you excited about writing your first automation tests on iOS, here's a few simple steps that would get you up and running pretty quick:

## Step 1: Create new iOS project
Create new project:

* Open Xcode.
* Choose `Create a new Xcode project`.
* Choose template `iOS` > `Application` > `Single View Application` > `Next`.
* Fill in your project information. Make sure the `Include Unit Tests` option is ticked.
* Choose a path for the project.
* Xcode will open the project automatically for you.

Try to **Run** the project (`⌘ + R`). It should compile and run smoothly in the simulator.

For the rest of this tutorial, let's just assume that your project name is `MyAwesomeProject` and it is located at `~/Documents/MyAwesomeProject`.

## Step 2: Setup cocoapods
In case you haven't installed [cocoapods](https://cocoapods.org/), you should do it now.

Open `Terminal` or `iTerm` and type in this command.

{% highlight shell %}
gem install cocoapods
{% endhighlight %}

Go to the project you just created.

{% highlight shell %}
cd ~/Documents/MyAwesomeProject
{% endhighlight %}

Initialize `cocoapods`.

{% highlight shell %}
# this will generate a file called `Podfile` in your project directory
pod init

# open it in your default text editor
open Podfile
{% endhighlight %}

If everything goes well, you should see content like this:

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

Declare necessary pods for unit tests in your `Podfile`:

For `Swift`: we'll use:

* [Quick](https://github.com/Quick/Quick): a behavior-driven development framework for Swift.
* [Nimble](https://github.com/Quick/Nimble): to express the expected outcomes of Swift expressions.

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

For `ObjC`: since we don't want to use dynamic framework, we'll remove the `use_frameworks!` line. And the pods we need are:

* [Specta](https://github.com/specta/specta): A light-weight TDD / BDD framework for Objective-C & Cocoa.
* [Expecta](https://github.com/specta/expecta/): A Matcher Framework for Objective-C/Cocoa.

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

Install all the pods we just declared (type this into your terminal).

{% highlight shell %}
pod install
{% endhighlight %}

Open the generated `.xcworkspace` file.

{% highlight shell %}
open MyAwesomeProject.xcworkspace
{% endhighlight %}

Now we're going to write our first unit test.

For `Swift`:

* Create a new file in the test target and name it to something like `MyFirstSpec.swift`.
* Put this content into it.

{% highlight swift %}
import Quick
import Nimble

class MyFirstSpec : QuickSpec {
  override func spec() {
    super.spec()

    describe("first test") {
      it("should pass") {
        expect(1).to(equal(1))
      }
    }

    describe("second test") {
      it("should fail") {
        expect(1).to(equal(2))
      }
    }
  }
}
{% endhighlight %}

For `ObjC`:

* Create a new file in the test target and name it to something like `MyFirstSpec.m`.
* Put this content into it.

{% highlight objc %}
#import <Specta/Specta.h>
#import <Expecta/Expecta.h>

SpecBegin(MyFirstSpec)

describe(@"first test", ^{
  it(@"should pass", ^{
    expect(1).to.equal(1);
  });
});

describe(@"second test", ^{
  it(@"should fail", ^{
    expect(1).to.equal(2);
  });
});

SpecEnd
{% endhighlight %}

Run your tests by pressing `⌘ + U`.
It should fail at the appropriate line.

![](/images/how-to-setup-testing-for-new-ios-project/MyFirstSpec-fail.png)

At this point, although it's quite easy to extend and write more unit tests. I would recommend to check out the pods's `README` to get more understanding and learn how to use them effectively.

## Step 4: Setup UI tests
  For UI tests, we'll use the same pod for both `ObjC` and `Swift`. It is called [KIF](https://github.com/kif-framework/KIF): An iOS Functional Testing Framework.

Add pod `KIF` to your `Podfile`.

{% highlight ruby %}
#...
target 'MyAwesomeProjectTests' do
  #...
  pod 'KIF'
end
#...
{% endhighlight %}

Install pod by typing this command into the terminal.

{% highlight shell %}
pod install
{% endhighlight %}

Now we're going to write our first UI test.

For `Swift`:

* Create a new temporary `Objective-C File` to tell Xcode that we want to use `ObjC` along with `Swift`. We can name this file to anything and save it to anywhere as long as it is still in the test target folder. (for example: `something.m`)

![](/images/how-to-setup-testing-for-new-ios-project/create-objc-file.png)

* Xcode will then ask if we want to also add a bridging header. Proceed with `Create Bridging Header`.

![](/images/how-to-setup-testing-for-new-ios-project/add-bridging-header.png)

* Now that we have the bridging header added and correctly configured, we can safely remove the temporary `something.m` file. (Right click to file in `Xcode` > `Delete` > `Move to Trash`)
* Import `KIF` from within the bridging header (`MyAwesomeProjectTests-Bridging-Header.h`).

{% highlight objc %}
#import <KIF/KIF.h>
{% endhighlight %}

* Add a `KIF` extension to help smooth things up.

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

* Create a new file in the test target and name it to something like `HomeTests.swift`.

{% highlight swift %}
import KIF

class HomeTests : KIFTestCase {

  // test method name MUST have the prefix `test`
  // for example: testA, testB, testLogin, ect...
  func testHomeScreen() {
    visitHomeScreen()
    tapOn("button")
    expectToSee("Some random label")
  }

}

// steps definition
extension HomeTests {

  func visitHomeScreen() {
    // since we already at the home screen by default, we don't need to do anything specifically here
  }

  func tapOn(button: String) {
    tester().tapViewWithAccessibilityLabel(button)
  }

  func expectToSee(label: String) {
    tester().waitForViewWithAccessibilityLabel(label)
  }

}
{% endhighlight %}

For `ObjC`:

* Create 2 new files in the test target and name them to something like `HomeTests.h` and `HomeTests.m` respectively.

{% highlight objc %}
#import <KIF/KIF.h>

@interface HomeTests : KIFTestCase
@end
{% endhighlight %}

{% highlight objc %}
#import "HomeTests.h"

@implementation HomeTests

#pragma mark - tests

// test method name MUST have the prefix `test`
// for example: testA, testB, testLogin, ect...
- (void)testHomeScreen {
  [self visitHomeScreen];
  [self tapOn:@"Button"];
  [self expectToSee:@"Some random label"];
}

#pragma mark - steps definition

- (void)visitHomeScreen {
  // since we already at the home screen by default, we don't need to do anything specifically here
}

- (void)tapOn:(NSString*)button {
  [tester tapViewWithAccessibilityLabel:button];
}

- (void)expectToSee:(NSString*)something {
  [tester waitForViewWithAccessibilityLabel:something];
}

@end
{% endhighlight %}

Press `⌘ + U` to run all tests (including unit tests and UI tests). It should fail because we haven't implemented anything yet. (there's nothing on the screen)

## Wrap up

In this post, you learnt how to setup a testing environment for iOS, including all the necessary pods for both `ObjC` and `Swift`. You're also provided some basic test samples to use as a starting point. Once again, I highly recommend you to read more about those testing frameworks ([Quick](https://github.com/Quick/Quick), [Specta](https://github.com/specta/specta), [KIF](https://github.com/kif-framework/KIF)). It would help a lot on your testing journey.
