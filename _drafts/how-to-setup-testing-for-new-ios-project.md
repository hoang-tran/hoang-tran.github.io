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

        ```swift
        func testCreateNewNote() {
          visitNotesScreen()
          clickOn("Create new note")
          fillInNewNoteForm()
          clickOn("Create")
          expectToSeeNewNoteCreatedSuccessfully()
        }
        ```

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

    ```shell
    gem install cocoapods
    ```

* Go to the project you just created

    ```shell
    cd ~/Documents/MyAwesomeProject
    ```

* Initialize `cocoapods`

    ```shell
    # this will generate a file called `Podfile` in your project directory
    pod init

    # open it in your default text editor
    open Podfile
    ```

* If everything goes well, you should see some content like this

    ```ruby
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
    ```

## Step 3: Setup unit tests

* Declare necessary pods for unit tests in your `Podfile`
    * For `Swift`: we'll use:
        * [Quick](https://github.com/Quick/Quick): a behavior-driven development framework for Swift
        * [Nimble](https://github.com/Quick/Nimble): to express the expected outcomes of Swift expressions

        ```ruby
        platform :ios, '9.0'

        target 'MyAwesomeProject' do
          use_frameworks!

          target 'MyAwesomeProjectTests' do
            inherit! :search_paths
            pod 'Quick'
            pod 'Nimble'
          end
        end
        ```

    * For `ObjC`: since we don't want to use dynamic framework, we'll remove the `use_frameworks!` line. And the pods we need are:
        * [Specta](https://github.com/specta/specta): A light-weight TDD / BDD framework for Objective-C & Cocoa
        * [Expecta](https://github.com/specta/expecta/): A Matcher Framework for Objective-C/Cocoa

        ```ruby
        platform :ios, '9.0'

        target 'MyAwesomeProject' do
          target 'MyAwesomeProjectTests' do
            inherit! :search_paths
            pod 'Specta'
            pod 'Expecta'
          end
        end
        ```

* Install all the pods we just declared (type this into your terminal)

    ```shell
    pod install
    ```

* Open the generated `.xcworkspace` file

    ```shell
    open MyAwesomeProject.xcworkspace
    ```

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
  For UI tests, we'll use the same pod for both `ObjC` and `Swift`. It is called [KIF](https://github.com/kif-framework/KIF): An iOS Functional Testing Framework

* Add pod `KIF` to your `Podfile`

    ```ruby
    #...
    target 'MyAwesomeProjectTests' do
      #...
      pod 'KIF'
    end
    #...
    ```

* Install pod by typing this command into the terminal

    ```shell
    pod install
    ```

* Write your first UI tests
    * For `Swift`:
        * Do some preparations beforehand (as stated in the [KIF pod](https://github.com/kif-framework/KIF#use-with-swift)):
            * Add a bridging header to your test target to do the static import:
            * Create a new temporary `Objective-C File` to tell `Xcode` that we want to use `ObjC` along with `Swift`. We can name this file to anything and save it to anywhere as long as it is still in the test target folder (for example: `something.m`)
![](/images/how-to-setup-testing-for-new-ios-project/create-objc-file.png)
            * `Xcode` will then ask if we want to also add a bridging header. Proceed with `Create Bridging Header`
![](/images/how-to-setup-testing-for-new-ios-project/add-bridging-header.png)
            * Now that we have the bridging header added and correctly configured, we can safely remove the temporary `something.m` file (Right click to file in `Xcode` > `Delete` > `Move to Trash`)
            * Import `KIF` from within the bridging header
<script src="https://gist.github.com/hoang-tran/0915730ed70e45b70e8abb92391b350d.js"></script>
        * Add a `KIF` extension to help smooth things up
<script src="https://gist.github.com/hoang-tran/c56fd62e18b824be3081d31a86c56860.js"></script>
        * Create a new file in the test target and name it to something like `HomeTests.swift`
<script src="https://gist.github.com/hoang-tran/014ca12f2d0d15fd985d965d639b9242.js"></script>
    * For `ObjC`:
        * Create 2 new files in the test target and name them to something like `HomeTests.h` and `HomeTests.m` respectively
<script src="https://gist.github.com/hoang-tran/725c2916bdd6a397dca23f8d97e48e01.js"></script>
<script src="https://gist.github.com/hoang-tran/e29080a787935fb2c53d8efc3bb7d777.js"></script>
* Press `⌘ + U` to run all tests (including unit tests and UI tests). It should fail for now because we haven't implemented anything yet

## Wrap up
abc
