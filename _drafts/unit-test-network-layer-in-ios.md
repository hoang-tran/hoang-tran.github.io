---
layout: post
title:  'How to unit test your Network Layer in iOS'
categories: ios testing
tags: ios network unittest rest api
---

## Declare the task to work on:

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

If you click on the link <https://api.github.com/users/hoang-tran> in your browser. It will return a json response:

![browser json response](/images/unit-test-network-layer-in-ios/browser-json-response.jpg)

# 1. Implement a simple network layer:

## 1.1. Design the protocol:

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

## 1.2. Some example implementations of the protocol:

### 1.2.1. NSURLSession:

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

### 1.2.2. AFNetworking:

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

### 1.2.3. Alamofire:

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

## 1.3. Put it in use

{% highlight swift %}
NativeApiClient.requestUserWithUsername("hoang-tran", onSuccess: { userData in

  print(userData)

}, onError: { error in

  print(error.localizedDescription)

})
{% endhighlight %}

# 2. Unit test the network layer we just wrote:

## 2.1. Before you start:

Make sure you're familiar with:

* [How to write unit tests in iOS Part 1: XCTestCase](/ios/testing/2016/07/31/how-to-write-unit-tests-in-ios-p1-xctestcase).
* [How to write unit tests in iOS Part 2: Behavior-driven Development (BDD)](/ios/testing/2016/08/07/how-to-write-unit-tests-in-ios-p2-behavior-driven-development-bdd).
* [Write better unit test assertions with Nimble](/ios/testing/2016/08/09/write-better-unit-test-assertion-with-nimble).

## 2.2. How are we gonna test a network request?

## 2.3. The implementation:

### Step : Setup project for unit testing

Open Podfile and add the following pods to your test target:

* **Quick**: a behavior-driven development (BDD) framework for Swift.
* **Nimble**: write test assertion that reads just like English.
* **Mockingjay**: stub network request in Swift.

{% highlight ruby %}
platform :ios, '9.0'

target 'TestNetworkLayer' do
  use_frameworks!

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

### Step : Create a new spec file for NativeApiClient

Create a new file called `NativeApiClientSpec.swift` in your test target.

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

### Step: Write a test that make real network request

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

### Step 2: Create stub response files

Open Terminal and go to the **TestNetworkLayerTests** directory

{% highlight sh %}
cd TestNetworkLayerTests
{% endhighlight %}

Create a new directory called **Fixtures** and then go to it:

{% highlight sh %}
mkdir Fixtures

cd Fixtures
{% endhighlight %}

Create a new json file called **GetUserSuccess.json**, then open it for edit:

{% highlight sh %}
touch GetUserSuccess.json

open GetUserSuccess.json
{% endhighlight %}

Paste the json content from Postman to the **GetUserSuccess.json** file:

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

### Step : Stub the network request

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

## Wrap up
