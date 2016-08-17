---
layout: post
title:  'How to use Graph API to retrieve Facebook user information'
categories: ios development
tags: ios swift facebook graph api
---

## What is Graph API?

## How to make a Graph API request?

{% highlight swift %}
let graphPath = "me"

let parameters = ["fields": ""]

let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    print(result)
  }
}

FBSDKGraphRequest(graphPath: graphPath, parameters: parameters).startWithCompletionHandler(completionHandler)
{% endhighlight %}

## How to parse the response?

## Putting it all together

### Get user public profile:

The public profile includes the following information:

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

{% highlight swift %}
let graphPath = "me"

let parameters = ["fields": "id, name, age_range, link, gender, locale, timezone, picture, updated_time, verified"]

let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    let json = JSON(result)
    print("json: \(json)")
    print("name: \(json["name"].stringValue)")
    print("age_min: \(json["age_range"]["min"].numberValue)")
    print("link: \(json["link"].stringValue)")
    print("gender: \(json["gender"].stringValue)")
    print("locale: \(json["locale"].stringValue)")
    print("timezone: \(json["timezone"].numberValue)")
    print("updated_time: \(json["updated_time"].stringValue)")
    print("verified: \(json["verified"].boolValue)")
    print("picture_url: \(json["picture"]["data"]["url"].stringValue)")
  }
}

FBSDKGraphRequest(graphPath: graphPath, parameters: parameters).startWithCompletionHandler(completionHandler)
{% endhighlight %}

### Get user email:

To get this, you need to ask for the `email` permission. Open `LoginViewController` and add `email` to the permission array:

{% highlight swift %}
let loginManager = FBSDKLoginManager()
let permissions = ["public_profile", "email"]
let handler = ...
loginManager.logInWithReadPermissions(permissions, fromViewController: self, handler: handler)
{% endhighlight %}

Then you make a Graph API request just like when you get public profile:

{% highlight swift %}
let graphPath = "me"

let parameters = ["fields": "email"]

let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    let json = JSON(result)
    print("email: \(json["email"].stringValue)")
  }
}

FBSDKGraphRequest(graphPath: graphPath, parameters: parameters).startWithCompletionHandler(completionHandler)
{% endhighlight %}

Or you can chain all fields together and fetch at the same time:

{% highlight swift %}
let parameters = ["fields": "id, email, name, age_range, link, gender, locale, timezone, picture, updated_time, verified"]
{% endhighlight %}

### Get user avatar with different sizes:

{% highlight swift %}
let path = "\(FBSDKAccessToken.currentAccessToken().userID)/picture"

let params = [
  "type": "large",
  "fields": "url",
  "redirect": "false"
]

let completionHandler = { (connection: FBSDKGraphRequestConnection!, result: AnyObject!, error: NSError?) in
  if let error = error {
    print(error.localizedDescription)
  } else {
    let json = JSON(result)
    print("picture_url: \(json["picture"]["data"]["url"].stringValue)")
  }
}

FBSDKGraphRequest(graphPath: path, parameters: params).startWithCompletionHandler(completionHandler)
{% endhighlight %}

The type can be one of these:

* large
* square
* album
* normal
* small

### Get user friend list

To get this, you need to ask for another permission, which is `user_friends`:

{% highlight swift %}
let loginManager = FBSDKLoginManager()
let permissions = ["public_profile", "email", "user_friends"]
let handler = ...
loginManager.logInWithReadPermissions(permissions, fromViewController: self, handler: handler)
{% endhighlight %}

