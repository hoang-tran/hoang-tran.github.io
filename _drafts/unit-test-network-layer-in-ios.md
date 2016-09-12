---
layout: post
title:  'How to unit test your Network Layer in iOS'
categories: ios testing
meta_description: Learn how to design, code and unit test your RESTful network layer in iOS using Mockingjay.
description: Learn how to design, code and unit test your RESTful network layer in iOS using Mockingjay.
summary: Learn how to design, code and unit test your RESTful network layer in iOS using Mockingjay.
tags: ios network unittest rest api
image: unit-test-network-layer-in-ios/mockingjay.jpg
---

Network communication is a crucial part in every iOS apps.

Whenever the users interact with a UI element in the app, there usually is one (or multiple) network request sent to an API server to get fresh data and present back to the users.

Let's look at an example network usage of a typical News Reader app:

* Request to authenticate users.
* Request to get list of news sources: Engadget, Mashable, Medium,...
* Request to get list of latest articles when users click on a news source.
* Request for more details when users click on an article.
* Request to mark all articles as read when users click on the corresponding button.

**Network requests are everywhere**. We should make sure that not only do we understand networking clearly but also can implement it very well.

Today we're gonna learn how to build a simple network layer to fetch data from a RESTful API server. The process includes **designing**, **coding** and **unit testing** altogether.

Here's the table of contents:

* [1. Design the protocol](#design-the-protocol)
* [2. Implement the protocol](#implement-the-protocol)
  * [NSURLSession](#nsurlsession)
  * [AFNetworking](#afnetworking)
  * [Alamofire](#alamofire)
* [3. Use the protocol](#use-the-protocol)
* [4. Write unit test for the protocol](#write-unit-test-for-the-protocol)
  * [What to test?](#what-to-test)
  * [How to test?](#how-to-test)
  * [Prepare to test](#prepare-to-test)
  * [Start writing test](#start-writing-test)
    * [Step 1: Setup project for unit testing](#step-1-setup-project-for-unit-testing)
    * [Step 2: Create a new spec file](#step-2-create-a-new-spec-file)
    * [Step 3: Write a test that makes real network request](#step-3-write-a-test-that-makes-real-network-request)
    * [Step 4: Create stubbed response file](#step-4-create-stubbed-response-file)
    * [Step 5: Stub the network request](#step-5-stub-the-network-request)
    * [Step 6: Expect the protocol to parse json correctly](#step-6-expect-the-protocol-to-parse-json-correctly)
    * [Step 7: Test the error case](#step-7-test-the-error-case)

## What RESTful API should we work on?

Let's just pick an easy one.

> Implement GitHub API to get a user's information

The syntax:

{% highlight abc %}
GET https://api.github.com/users/:username
{% endhighlight %}

Examples:

{% highlight abc %}
GET https://api.github.com/users/hoang-tran

GET https://api.github.com/users/orta

GET https://api.github.com/users/chriseidhof
{% endhighlight %}

If you click on the link <https://api.github.com/users/hoang-tran> in your browser. It will return a response in json format:

![browser json response](/images/unit-test-network-layer-in-ios/browser-json-response.jpg)

Simple and sweet, right? Let's head over to the implementation in our iOS app.

# 1. Design the protocol:

Create a new file called **GitHubApiClient.swift** in your main target.

Add a protocol named **GitHubApiClient**:

{% highlight swift %}
protocol GitHubApiClient {
}
{% endhighlight %}

As you can see, the API requires us to pass in **username** as an argument. We will reflect that in our protocol:

{% highlight swift %}
protocol GitHubApiClient {
  static func requestUserWithUsername(username: String)
}
{% endhighlight %}

Now we need a way to handle callbacks when the request finishes. If it succeeds, we want to receive the user data in a nice format.

{% highlight swift %}
static func requestUserWithUsername(username: String,
                                   onSuccess: (GitHubUserData) -> Void)
{% endhighlight %}

The **GitHubUserData** is just a struct we created to store the data obtained from the json response.

{% highlight swift %}
struct GitHubUserData {
  var name: String
  var bio: String
  var email: String
  var numberOfFollowers: Int
  var numberOfFollowing: Int
  // ...
}
{% endhighlight %}

When the request fails, we want to get the error.

{% highlight swift %}
static func requestUserWithUsername(username: String,
                                   onSuccess: (GitHubUserData) -> Void,
                                     onError: (NSError) -> Void)
{% endhighlight %}

Let's alias the 2 callbacks to something more readable:

{% highlight swift %}
typealias GitHubGetUserCallback = (GitHubUserData) -> Void

typealias ErrorCallback = (NSError) -> Void
{% endhighlight %}

Now, our protocol will look like this:

{% highlight swift %}
typealias GitHubGetUserCallback = (GitHubUserData) -> Void
typealias ErrorCallback = (NSError) -> Void

protocol GitHubApiClient {
  static func requestUserWithUsername(username: String,
                                     onSuccess: GitHubGetUserCallback,
                                       onError: ErrorCallback)
}
{% endhighlight %}

There's time when we don't want to specify any callbacks at all. Let's change both of them to **optionals**:

{% highlight swift %}
protocol GitHubApiClient {
  static func requestUserWithUsername(username: String,
                                     onSuccess: GitHubGetUserCallback?,
                                       onError: ErrorCallback?)
}
{% endhighlight %}

Now we are done with the protocol design. Let's move on to the coding part.

# 2. Implement the protocol:

There are many ways to implement network requests in iOS. The 3 most popular ones are: **NSURLSession**, **AFNetworking** and **Alamofire**.

### NSURLSession:

This is supported natively in iOS 7.0 and above.

{% highlight swift %}
import Foundation
import SwiftyJSON

class NativeApiClient: GitHubApiClient {

  static func requestUserWithUsername(username: String,
                                     onSuccess: GitHubGetUserCallback? = nil,
                                       onError: ErrorCallback? = nil) {
    let urlString = "https://api.github.com/users/\(username)"
    let url = NSURL(string: urlString)!

    let defaultSession = NSURLSession(configuration: NSURLSessionConfiguration.defaultSessionConfiguration())
    let dataTask = defaultSession.dataTaskWithURL(url) { data, response, error in
      if let error = error {
        onError?(error)
      } else if let data = data {
        let json = JSON(data: data)
        onSuccess?(GitHubUserData(json: json))
      }
    }
    dataTask.resume()
  }

}
{% endhighlight %}

### AFNetworking:

Remember to add `pod 'AFNetworking'` to your Podfile.

{% highlight swift %}
import AFNetworking
import SwiftyJSON

class AFNetworkingApiClient: GitHubApiClient {

  static func requestUserWithUsername(username: String,
                                     onSuccess: GitHubGetUserCallback? = nil,
                                       onError: ErrorCallback? = nil) {
    let urlString = "https://api.github.com/users/\(username)"
    let url = NSURL(string: urlString)!

    let manager = AFURLSessionManager(sessionConfiguration: NSURLSessionConfiguration.defaultSessionConfiguration())
    let request = NSURLRequest(URL: url)
    let dataTask = manager.dataTaskWithRequest(request) { response, data, error in
      if let error = error {
        onError?(error)
      } else if let data = data {
        let json = JSON(data)
        onSuccess?(GitHubUserData(json: json))
      }
    }
    dataTask.resume()
  }

}
{% endhighlight %}

### Alamofire:

Remember to add `pod 'Alamofire'` to your Podfile.

{% highlight swift %}
import Alamofire
import SwiftyJSON

class AlamofireApiClient: GitHubApiClient {

  static func requestUserWithUsername(username: String,
                                     onSuccess: GitHubGetUserCallback? = nil,
                                       onError: ErrorCallback? = nil) {
    let urlString = "https://api.github.com/users/\(username)"

    Alamofire.request(.GET, urlString)
      .validate()
      .responseJSON { response in
        switch response.result {
        case .Success:
          if let data = response.result.value {
            let json = JSON(data)
            onSuccess?(GitHubUserData(json: json))
          }
        case .Failure(let error):
          onError?(error)
        }
    }
  }

}
{% endhighlight %}

# 3. Use the protocol:

This is fairly straightforward. Just put in:

* parameter for **username**.
* closure for **onSuccess** callback.
* closure for **onError** callback.

{% highlight swift %}
NativeApiClient.requestUserWithUsername("hoang-tran", onSuccess: { userData in

  print(userData)

}, onError: { error in

  print(error.localizedDescription)

})
{% endhighlight %}

Or we can remove the callback we don't care about.

{% highlight swift %}
NativeApiClient.requestUserWithUsername("hoang-tran", onSuccess: { userData in

  print(userData)

})
{% endhighlight %}

Or

{% highlight swift %}
NativeApiClient.requestUserWithUsername("hoang-tran", onError: { error in

  print(error.localizedDescription)

})
{% endhighlight %}

Or remove everything.

{% highlight swift %}
NativeApiClient.requestUserWithUsername("hoang-tran")
{% endhighlight %}

# 4. Write unit test for the protocol:

## What to test?

When talking about network request, there's a couple of things we need to consider:

* Does the request go to the correct url?
* When the request succeeds, does it parse json properly and return the correct model?
* When the request fails, does it return the corresponding error?

## How to test?

We will use a HTTP stubbing framework called [Mockingjay](https://github.com/kylef/Mockingjay).

It allows you to say this:

{% highlight abc %}
Stub request with url A and return json file B.
{% endhighlight %}

Or:

{% highlight abc %}
Stub request with url A and return error.
{% endhighlight %}

### What does Stub mean anyway?

This is the syntax of a stub:

{% highlight abc %}
Stub event A and do custom action B
{% endhighlight %}

What it means:

> When event **A** happens, don't do it the old way. Do custom action **B** instead.

So the line:

{% highlight abc %}
Stub request with url A and return json file B.
{% endhighlight %}

Will translate to as:

> Whenever there's a request that goes to url **A**, do not send it over to network.
>
> Instead, use the file **B** in our project's local directory as the response.

For example:

{% highlight abc %}
Stub request with url https://api.github.com/users/hoang-tran
and return the file GetUserSuccess.json
{% endhighlight %}

This reads as:

> If there's a request that goes to url **https://api.github.com/users/hoang-tran**,
> do not send it over to network.
>
> Instead, use the file **GetUserSuccess.json** as the response

So to write unit test for a network request, we will need to do things in the following order:

1. Stub the request and return with our custom json file.
2. Actually make the network request using the protocol.
3. Describe our expectations for the returned values in both cases: **onSuccess** and **onError**.

## Prepare to test:

Before we start, make sure you're familiar with:

* [How to write unit tests in iOS Part 1: XCTestCase](/ios/testing/2016/07/31/how-to-write-unit-tests-in-ios-p1-xctestcase).
* [How to write unit tests in iOS Part 2: Behavior-driven Development (BDD)](/ios/testing/2016/08/07/how-to-write-unit-tests-in-ios-p2-behavior-driven-development-bdd).
* [Write better unit test assertions with Nimble](/ios/testing/2016/08/09/write-better-unit-test-assertion-with-nimble).

##  Start writing test:

### Step 1: Setup project for unit testing

Open Podfile and add the following pods to your test target:

* **Quick**: a behavior-driven development (BDD) framework for Swift.
* **Nimble**: write test assertion that reads just like English.
* **Mockingjay**: stub network request in Swift.

{% highlight ruby %}
platform :ios, '9.0'

target 'TestNetworkLayer' do
  use_frameworks!

  #...

  target 'TestNetworkLayerTests' do
    inherit! :search_paths
    pod 'Quick'
    pod 'Nimble'
    pod 'Mockingjay'
  end
end
{% endhighlight %}

Type this command into your Terminal to install all 3 pods.

{% highlight sh %}
pod install
{% endhighlight %}

Open the generated *.xcworkspace* file.

{% highlight sh %}
open TestNetworkLayer.xcworkspace
{% endhighlight %}

### Step 2: Create a new spec file

Create a new file called **NativeApiClientSpec.swift** in your test target.

{% highlight swift %}
import Quick
import Nimble

class NativeApiClientSpec : QuickSpec {
  override func spec() {
    super.spec()

    describe("first test") {
      it("should pass") {
        expect(1).to(equal(1))
      }
    }
  }
}
{% endhighlight %}

Press **Cmd + U** in Xcode and make sure the test pass.

### Step 3: Write a test that makes real network request

We will write the success case first:

{% highlight swift %}
override func spec() {
  super.spec()

  describe("requestUserWithName") {
    context("success") {
      it("returns GitHubUserData") {
      }
    }
  }
}
{% endhighlight %}

Make a real network request in the **it** closure:

{% highlight swift %}
it("returns GitHubUserData") {
  NativeApiClient.requestUserWithName("hoang-tran", onSuccess: { userData in

  })
}
{% endhighlight %}

We need to store the **userData** when the request succeeds:

{% highlight swift %}
it("returns GitHubUserData") {
  var returnedUserData: GitHubUserData?

  NativeApiClient.requestUserWithName("hoang-tran", onSuccess: { userData in

    returnedUserData = userData

  })
}
{% endhighlight %}

Then we expect the **returnedUserData** to have value:

{% highlight swift %}
it("returns GitHubUserData") {
  var returnedUserData: GitHubUserData?

  NativeApiClient.requestUserWithName("hoang-tran", onSuccess: { userData in

    returnedUserData = userData

  })

  expect(returnedUserData).toEventuallyNot(beNil())
}
{% endhighlight %}

The **toEventuallyNot** means that it will continuously check the **returnedUserData** to see if it has value (not nil).

After a couple of seconds, if the **returnedUserData** has value , the test passes, otherwise it fails.

However, when you run the test (**Cmd + U**), it will fail.

Because currently we're making real network request and it might take time to finish. The **toEventually** does not wait long enough for the request to set value for the **returnedUserData** and thus causes the failure.

The default wait time is 1 second. If we change it to a larger number, the test shall pass.

{% highlight swift %}
//...

expect(returnedUserData).toEventuallyNot(beNil(), timeout: 20)
{% endhighlight %}

But we don't wanna do that. Our goal is to **NOT** make real network request at all.

### Step 4: Create stubbed response file

Open Terminal and go to the **TestNetworkLayerTests** directory (assuming your project name is **TestNetworkLayer**)

{% highlight sh %}
cd TestNetworkLayerTests
{% endhighlight %}

Create a new directory called **Fixtures** and then go to it:

{% highlight sh %}
mkdir Fixtures

cd Fixtures
{% endhighlight %}

Note that **Fixtures** is just a common name to refer to those stubbed response files. You can name the directory to anything.

Create a new json file called **GetUserSuccess.json**, then open it for edit:

{% highlight sh %}
touch GetUserSuccess.json

open GetUserSuccess.json
{% endhighlight %}

Open this link <https://api.github.com/users/hoang-tran> in your browser.

Paste the json content to the **GetUserSuccess.json** file:

{% highlight json %}
{
  "login": "hoang-tran",
  "id": 6714157,
  "avatar_url": "https://avatars.githubusercontent.com/u/6714157?v=3",
  "gravatar_id": "",
  "url": "https://api.github.com/users/hoang-tran",
  "html_url": "https://github.com/hoang-tran",
  "followers_url": "https://api.github.com/users/hoang-tran/followers",
  "following_url": "https://api.github.com/users/hoang-tran/following{/other_user}",
  "gists_url": "https://api.github.com/users/hoang-tran/gists{/gist_id}",
  "starred_url": "https://api.github.com/users/hoang-tran/starred{/owner}{/repo}",
  "subscriptions_url": "https://api.github.com/users/hoang-tran/subscriptions",
  "organizations_url": "https://api.github.com/users/hoang-tran/orgs",
  "repos_url": "https://api.github.com/users/hoang-tran/repos",
  "events_url": "https://api.github.com/users/hoang-tran/events{/privacy}",
  "received_events_url": "https://api.github.com/users/hoang-tran/received_events",
  "type": "User",
  "site_admin": false,
  "name": "Hoang Tran",
  "company": "East Agile Vietnam",
  "blog": "http://hoangtran.me/",
  "location": "Ho Chi Minh city, VietNam",
  "email": "hoangtx.master@gmail.com",
  "hireable": null,
  "bio": "Hi, I'm Hoang. I'm a Swift lover. I blog weekly about iOS development/testing/deployment at http://hoangtran.me/\r\nAnd I happen to love GitHub so much 😜 ",
  "public_repos": 15,
  "public_gists": 23,
  "followers": 16,
  "following": 93,
  "created_at": "2014-02-18T09:42:07Z",
  "updated_at": "2016-08-30T03:04:04Z"
}
{% endhighlight %}

Open the current directory in Finder:

{% highlight sh %}
open .
{% endhighlight %}

Drag the **Fixtures** directory into your test target:

![Test Fixtures](/images/unit-test-network-layer-in-ios/drag-folder-to-xcode.jpg)

Xcode will ask you for confirmation. Make sure you added the directory to the test target instead of your main target.

Hit **Finish**:

![xcode add directory confirmation](/images/unit-test-network-layer-in-ios/add-folder-confirmation.jpg)

### Step 5: Stub the network request

Back to **NativeApiClientSpec.swift**, we're gonna stub the request so that it will use our **GetUserSuccess.json** file as the response instead of making real network request.

{% highlight swift %}
it("returns GitHubUserData") {
  var returnedUserData: GitHubUserData?

  // 1.
  let path = NSBundle(forClass: self.dynamicType).pathForResource("GetUserSuccess", ofType: "json")!

  // 2.
  let data = NSData(contentsOfFile: path)!

  // 3.
  self.stub(uri("https://api.github.com/users/hoang-tran"), builder: jsonData(data))

  NativeApiClient.requestUserWithName("hoang-tran", onSuccess: { userData in
    returnedUserData = userData
  })
  expect(returnedUserData).toEventuallyNot(beNil())
}
{% endhighlight %}

We did a couple of steps here:

1. Get the path of the **GetUserSuccess.json** file.
2. Read its content into a NSData object.
3. Stub the url so that everytime a request goes to <https://api.github.com/users/hoang-tran>, it will return the fake json data without having to reach for network.

With everything stubbed, hit **Cmd + U** again.

This time, it should pass.

To make sure that it does not make any real network request, turn off your internet connection and run your tests again. It should still pass.

### Step 6: Expect the protocol to parse json correctly

We should also write some expectations for the **returnedUserData** to make sure that the json response is parsed properly into the **GitHubUserData** struct.

{% highlight swift %}
//...

expect(returnedUserData).toEventuallyNot(beNil())
expect(returnedUserData?.name) == "Hoang Tran"
expect(returnedUserData?.email) == "hoangtx.master@gmail.com"
expect(returnedUserData?.numberOfFollowers) == 120
expect(returnedUserData?.numberOfFollowing) == 133
{% endhighlight %}

### Step 7: Test the error case

First declare the error context.

{% highlight swift %}
override func spec() {
  super.spec()

  describe("requestUserWithName") {
    context("success") {
      it("returns GitHubUserData") {
        //...
      }
    }

    context("error") {
      it("returns error") {

      }
    }
  }
}
{% endhighlight %}

Stub the request to return your custom error:

{% highlight swift %}
let error = NSError(domain: "Describe your error here", code: 404, userInfo: nil)

self.stub(uri("https://api.github.com/users/hoang-tran"), builder: failure(error))
{% endhighlight %}

The whole test case:

{% highlight swift %}
it("returns error") {
  var returnedError: NSError?

  let error = NSError(domain: "Describe your error here", code: 404, userInfo: nil)

  self.stub(uri("https://api.github.com/users/hoang-tran"), builder: failure(error))

  NativeApiClient.requestUserWithName("hoang-tran", onError: { error in
    returnedError = error
  })

  expect(returnedError).toEventuallyNot(beNil())
}
{% endhighlight %}

Run the test again and make sure it's green.

## Wrap up

Wow! I didn't expect this to turn into such a looooong post. Let's end it here.

You can find the full code sample at <https://github.com/hoang-tran/TestNetworkLayer>.

I'm very happy to see your comments. Please let me know:

* How do you design your network layer?
* What networking framework do you use? And why?
* Have you ever written unit tests for your network layer? If yes, how do you do that?
