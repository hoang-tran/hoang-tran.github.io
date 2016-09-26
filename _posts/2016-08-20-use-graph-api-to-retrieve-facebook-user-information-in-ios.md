---
layout: post
title:  'How to use Graph API to retrieve Facebook user information in iOS'
meta_description: 'Learn how to use the Facebook Graph API in iOS to fetch various user information including: public profile, email address, friend list and user avatar.'
description: 'Learn how to use the Facebook Graph API in iOS to fetch various user information including: public profile, email address, friend list and user avatar.'
summary: 'Learn how to use the Facebook Graph API in iOS to fetch various user information including: public profile, email address, friend list and user avatar.'
categories: ios development
tags: ios swift facebook graph api
image: use-graph-api-to-retrieve-facebook-user-information-in-ios/cover.jpg
related: [
  'How to integrate login with Facebook in iOS'
]
---

## What is Graph API?

It's just a collection of HTTP-based APIs to get data in and out of Facebook.

## Why would you want to use Graph API?

Because:

* You want to obtain the users's information to better understand them.
* Graph API is simple to use. It's already integrated inside the Facebook iOS SDK.
* It's easy to customize your requests to get the desired returned values.
* There are a lot of information that you can access using the Graph API.

## How to make a Graph API request?

Before you can make any Graph API requests, you need to authenticate with Facebook first. You can have a look at the [previous post](/ios/development/2016/08/14/login-with-facebook-in-ios) for step-by-step guide.

A Graph API request looks like this:

{% highlight swift %}
FBSDKGraphRequest(graphPath: String!, parameters: [NSObject : AnyObject]!)
{% endhighlight %}

Just like any other HTTP requests, it takes 2 parameters:

* graphPath: a url String to specify where the request should go.
* parameters: a dictionary to specify what to include in the request.

Here is an example of initializing a Graph API request that points to the `me` path with no parameters:

{% highlight swift %}
let graphRequest = FBSDKGraphRequest(graphPath: "me", parameters: nil)
{% endhighlight %}

And to send it:

{% highlight swift %}
graphRequest.startWithCompletionHandler { (connection, result, error) in

}
{% endhighlight %}

Here's typical completion handler:

{% highlight swift %}
graphRequest.startWithCompletionHandler { (connection, result, error) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    print(result)
  }
}
{% endhighlight %}

In case the request succeeds, you'll receive a `result` dictionary in JSON format which contains all the user information in it.

To avoid a lot of ugly code for parsing that dictionary, I highly recommend that we use the [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) pod to make the process more smoothly. Here's [why](https://github.com/SwiftyJSON/SwiftyJSON#why-is-the-typical-json-handling-in-swift-not-good).

## Integrate SwiftyJSON to your project

First, add the `SwiftyJSON` pod to your Podfile:

{% highlight ruby %}
target 'TestFacebookLogin' do
  use_frameworks!

  pod 'FBSDKCoreKit'
  pod 'FBSDKLoginKit'
  pod 'SwiftyJSON'
end
{% endhighlight %}

Run `pod install` on your terminal.

Import it where you want to use: (Ex: HomeViewController.swift)

{% highlight swift %}
import SwiftyJSON
{% endhighlight %}

## Alright! Let's putting it all together

### To get user public profile:

The `public_profile` permission allow us to access the following information:

* id
* name
* first_name
* last_name
* age_range
* link
* gender
* locale
* picture
* timezone
* updated_time
* verified

This is the request that fetch all the above information:

{% highlight swift %}
let graphPath = "me"

let parameters = ["fields": "id, name, first_name, last_name, age_range, link, gender, locale, timezone, picture, updated_time, verified"]

let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    let json = JSON(result)
    print(json)
  }
}

let graphRequest = FBSDKGraphRequest(graphPath: graphPath, parameters: parameters)

graphRequest.startWithCompletionHandler(completionHandler)
{% endhighlight %}

The `print(json)` will give you this output:

{% highlight abc %}
{
  "locale" : "en_US",
  "last_name" : "Tran",
  "verified" : true,
  "age_range" : {
    "min" : 21
  },
  "picture" : {
    "data" : {
      "url" : "https:\/\/scontent.xx.fbcdn.net\/v\/t1.0-1\/p50x50\/12654253_1282228235124954_3627839379317864707_n.jpg?oh=c5895f2a3bb1d1a6161fd4837bc1e97d&oe=58509005",
      "is_silhouette" : false
    }
  },
  "updated_time" : "2016-08-01T17:36:06+0000",
  "email" : "hoangtx.master@gmail.com",
  "timezone" : 7,
  "gender" : "male",
  "name" : "Hoang Tran",
  "first_name" : "Hoang",
  "id" : "1430456420302134",
  "link" : "https:\/\/www.facebook.com\/app_scoped_user_id\/1430456420302134\/"
}
{% endhighlight %}

Here's how we can easily access each individual information inside the JSON:

{% highlight swift %}
let json = JSON(result)
print(json["name"].stringValue)
print(json["first_name"].stringValue)
print(json["last_name"].stringValue)
print(json["age_range"]["min"].numberValue)
print(json["link"].stringValue)
print(json["gender"].stringValue)
print(json["locale"].stringValue)
print(json["timezone"].numberValue)
print(json["updated_time"].stringValue)
print(json["verified"].boolValue)
print(json["picture"]["data"]["url"].stringValue)
{% endhighlight %}

### Get user email:

To get this, you need to ask for the `email` permission. Open `LoginViewController` and add `email` to the permission array:

{% highlight swift %}
let loginManager = FBSDKLoginManager()
let permissions = ["public_profile", "email"]
let handler = ...
loginManager.logInWithReadPermissions(permissions, fromViewController: self, handler: handler)
{% endhighlight %}

After changing permission, you have to logout and authenticate again.

Then you can make a Graph API request just like what you did with public profile:

{% highlight swift %}
let graphPath = "me"
let parameters = ["fields": "email"]
let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    let json = JSON(result)
    print(json["email"].stringValue)
  }
}
let graphRequest = FBSDKGraphRequest(graphPath: graphPath, parameters: parameters)
graphRequest.startWithCompletionHandler(completionHandler)
{% endhighlight %}

Or you can chain all fields together and fetch at the same time:

{% highlight swift %}
let parameters = ["fields": "id, email, name, first_name, last_name, age_range, link, gender, locale, timezone, picture, updated_time, verified"]
{% endhighlight %}

### Get user friend list

To get this, you need to ask for the `user_friends` permission:

{% highlight swift %}
let loginManager = FBSDKLoginManager()
let permissions = ["public_profile", "email", "user_friends"]
let handler = ...
loginManager.logInWithReadPermissions(permissions, fromViewController: self, handler: handler)
{% endhighlight %}

Now we'll make a request to `me/friends` instead of `me`.

{% highlight swift %}
let graphPath = "me/friends"
let parameters = ["fields": ""]
let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    let json = JSON(result)
    let friendJSONArray = json["data"].arrayValue
    for friendJSON in friendJSONArray {
      print(friendJSON["name"].stringValue)
      print(friendJSON["id"].intValue)
    }
    let nextPageToken = json["paging"]["cursors"]["after"].stringValue
    let prevPageToken = json["paging"]["cursors"]["before"].stringValue
  }
}
let graphRequest = FBSDKGraphRequest(graphPath: graphPath, parameters: parameters)
graphRequest.startWithCompletionHandler(completionHandler)
{% endhighlight %}

This does not return all friends on the users's account. Only those that already installed your app will be returned. If there are too many friends, Facebook only returns the first 25 results. The remaining friends can be accessed using the `nextPageToken` and `prevPageToken`.

To request for the next 25 friends:

{% highlight swift %}
let graphPath = "me/friends"
let parameters = ["after": nextPageToken]
let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  //...
}
let graphRequest = FBSDKGraphRequest(graphPath: graphPath, parameters: parameters)
graphRequest.startWithCompletionHandler(completionHandler)
{% endhighlight %}

And you can fetch the previous 25 results by using the same method, only change the parameters:

{% highlight swift %}
let parameters = ["before": prevPageToken]
{% endhighlight %}

You can also change the `returned limit` to some bigger number instead of just 25 results:

{% highlight swift %}
let graphPath = "me/friends"
let parameters = ["limit": 100]
{% endhighlight %}

This can be applied to fetching for next/prev page as well:

{% highlight swift %}
let graphPath = "me/friends"
let parameters = [
  "after": nextPageToken,
  "limit": 100
]
{% endhighlight %}

### Get user avatar with different sizes:

Although you can access the `picture` field when fetching public profile, it only contains a very small version of the user's avatar (50x50).

To get bigger sizes, you should make a request to `me/picture` with some custom parameters:

{% highlight swift %}
let graphPath = "me/picture"
let parameters = [
  "type": "large",
  "redirect": "false"
]
let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    let json = JSON(result)
    print(json["data"]["url"].stringValue)
  }
}
let graphRequest = FBSDKGraphRequest(graphPath: graphPath, parameters: parameters)
graphRequest.startWithCompletionHandler(completionHandler)
{% endhighlight %}

There are 4 parameters that we can consider here:

* `type`: this is some preset sizes for the avatar. It can be one of the following:
  * large (200x200)
  * normal (100x100)
  * square (50x50)
  * album (50x50)
  * small (50x50)
* `redirect`:
  * if it is true,  it will return the actual image data.
  * if it is false, it will return a JSON response.
* `width`: the width of the avatar in pixels.
* `height`: the height of the avatar in pixels.

You can specify custom `width` and `height` in the parameters and it will fall down to the closest size that match.

For example: If you set `width` to 300, it will return 320. If set to 400, it'll return 480.

{% highlight swift %}
let parameters = [
  "width": 300,
  "height": 300,
  "redirect": "false"
]
{% endhighlight %}

To get maximum size, just set `width` and `height` to some large numbers:

{% highlight swift %}
let parameters = [
  "width": 2000,
  "height": 2000,
  "redirect": "false"
]
{% endhighlight %}

## Wrap up

Today we learnt about some very basic usage of the Facebook Graph API. We then went on to fetch some pieces of user information including: public profile, email, friend list and avatar.

I think those information is quite enough for your day-to-day iOS development. If you need more advanced features, you can read more on [Facebook Graph API documentation](https://developers.facebook.com/docs/graph-api/using-graph-api) and experiment using the [Graph API Explorer](https://developers.facebook.com/tools/explorer).

Please notice that this is just scratching the surface. Graph API is a powerful set of APIs that leverage the immense amount of information that billions of Facebook users generate over time. There are a whole lot of things that you can do with Graph API that I didn't mention here. Maybe we can save it for a later post if you're still interested.
