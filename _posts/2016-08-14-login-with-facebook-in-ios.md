---
layout: post
title:  'How to integrate login with Facebook in iOS'
categories: ios development
tags: ios swift facebook login
image: login-with-facebook-in-ios/login-facebook-cover.png
---

Nowadays, there are very few apps that do not authenticate their users. We all wanna know who our users are, how they interact with other users and what makes them engaged the most.

Therefore it's very common that when a user opens the app, the first thing they're gonna see is the Login screen. Usually, there are 2 options we can provide:

* login via email.
* login via social networks: Facebook, Twitter, Google+.

In this post, we're gonna discuss about the Facebook login option.

## We will build a simple app that:

* Has 2 screens: Login screen and Home screen.
* There is a `Login with Facebook` button on the Login screen. When users tap on that button, they will be taken to Safari or Facebook iOS app (if installed) to do the authentication.
* After successfully authenticated, they will move to the Home screen, which only has a welcome label and a Logout button. When they tap on the Logout button, they will be right back at the Login screen again.

Sounds simple and easy, right?

Here a gif that sums up the whole process:

![](/images/login-with-facebook-in-ios/login-facebook.gif)

## Understand the Facebook authentication process.

Imagine Facebook as a market.

You go to the market and register for a booth to sell things.

When users come to your booth and they like what they see, you may ask them to provide some of their personal information. You may also ask them about their friends, their interests, their recent activities... as long as they are happy to tell you.

With those information, you are able to do a lot of things such as:

* Make a list of users for a lucky draw at the end of the show.
* Send user a congratulation email on their birthday.
* Send new offer and sale.
* Make analytics about which users are interested in which items in your booth.
* ect...

The act of `asking users for permissions to use their information` and `having them to accept` is called `authentication`. They trust you enough to give those information and even let you do something on their behalf. (post to their Timeline, invite their friends...)

## So what do we need to prepare for Facebook authentication?

Actually, there are only 2 things that we need:

1. A booth:
  * this refers to as a `Facebook app`.
  * I'm **not** talking about the `Facebook iOS app` on your phone. I'm talking about the `Facebook app` you registered on Facebook Developers website.
  * A `Facebook app` is like an identity of your business on Facebook. All authentications will happen through that.
2. A way for user to come to our booth:
  * this refers to as `platform`.
  * It can be a website, a game, an app... In our case, it'll be an iOS app.

## Here's a summary of what we're gonna do:

1. [Setup your booth (Facebook App)](#setup-your-booth-facebook-app)
  * [Step 1: Create a new Facebook app.](#step-1-create-a-new-facebook-app)
  * [Step 2: Add iOS platform to your Facebook app.](#step-2-add-ios-platform-to-your-facebook-app)
2. [Integrate to iOS project.](#lets-integrate-it-into-to-your-ios-project)
  * [Step 1: Import Facebook SDK.](#step-1-import-facebook-sdk)
  * [Step 2: Configure the Info.plist file.](#step-2-configure-the-infoplist-file)
  * [Step 3: Wire it up in AppDelegate.](#step-3-wire-it-up-in-appdelegate)
  * [Step 4: Call login action in ViewController.](#step-4-call-login-action-in-viewcontroller)
  * [Step 5: Handle autologin.](#step-5-handle-autologin)
  * [Step 6: Provide a way for user to logout](#step-6-provide-a-way-for-user-to-logout)

Alright, without further ado, let's jump right in and make it happen.

## Setup your booth (Facebook App)

### Step 1: Create a new Facebook app.

Visit [Facebook Developers website](https://developers.facebook.com/apps) and click on `Add a New App`.

![](/images/login-with-facebook-in-ios/facebook-developers-apps.jpg)

Fill in your app information:

![](/images/login-with-facebook-in-ios/create-new-app-id.png)

* Display Name: this is what the user will see when the authentication happens. You can put any text here, including spaces and special characters. Ex: `My Awesome App`, `Super News Reader`, etc...
* Is this a test version of another app? **No**.
* Contact Email: put your email address here.
* Categories: Choose the category that best describe your app. Ex: if you're building a news reader app, choose `News`.

Then hit `Create App ID`.

Facebook will challenge you with a simple quiz to detect whether you're a real human. Ex: select all the photos which show a flower.

![](/images/login-with-facebook-in-ios/security-check.jpg)

Select all the correct photos and hit `Submit` (scroll down if you don't see the button). You will then be redirected to the app dashboard.

### Step 2: Add iOS platform to your Facebook app.

Click on `Settings`.

![](/images/login-with-facebook-in-ios/app-dashboard-product.jpg)

Click on `Add Platform`.

![](/images/login-with-facebook-in-ios/add-platform.jpg)

Tap on the Apple logo.

![](/images/login-with-facebook-in-ios/choose-platform.jpg)

This will generate a form for iOS platform.

![](/images/login-with-facebook-in-ios/ios-platform.png)

Now we need the bundle ID to fill in this form. Let's open your iOS project and follow these 3 steps:

1. Tap on the project icon on the left sidebar.
2. Choose `General` tab.
3. Copy the `Bundle Identifier` field.

![](/images/login-with-facebook-in-ios/bundle-id.jpg)

Go back to the iOS form on the web and paste in the bundle ID you just copied.

Turn on `Single Sign On`.

Then click `Save Changes`.

![](/images/login-with-facebook-in-ios/ios-platform-filled.jpg)

Now your Facebook app is fully configured.

## Let's integrate it into to your iOS project

### Step 1: Import Facebook SDK.

The easiest way is to use `cocoapods`. If you haven't installed `cocoapods` yet, please read [this guide at step 2](/ios/testing/2016/07/24/how-to-setup-testing-for-new-ios-project/#step-2-setup-cocoapods).

Add `FBSDKCoreKit` and `FBSDKLoginKit` pods to your Podfile:

{% highlight ruby %}
# replace 'TestFacebookLogin' with your own target name
target 'TestFacebookLogin' do
  use_frameworks!

  pod 'FBSDKCoreKit'
  pod 'FBSDKLoginKit'
end
{% endhighlight %}

Open `Terminal` and run this command:

{% highlight sh %}
pod install
{% endhighlight %}

### Step 2: Configure the Info.plist file.

Open the generated `.xcworkspace`.

Right-click on the `Info.plist`.

Choose to open as `Source Code`.

![](/images/login-with-facebook-in-ios/info-plist-open.png)

You will see the content of `Info.plist` in XML format:

![](/images/login-with-facebook-in-ios/info-plist-before.jpg)

Go back to your Facebook app settings on the web.

Click on `Quick Start`

![](/images/login-with-facebook-in-ios/ios-platform-quick-start.jpg)

Scroll down to the `Configure your info.plist` part.

Facebook already generated 2 blocks of XML code for us with all the necessary information prefilled.

![](/images/login-with-facebook-in-ios/configure-plist.jpg)

Copy the first block to your `Info.plist` (under the `<dict>` tag).

Then copy the second block just below the first one.

If you do it correctly, your `Info.plist` will look like this:

![](/images/login-with-facebook-in-ios/info-plist-after.jpg)

### Step 3: Wire it up in AppDelegate.

Open `AppDelegate.swift`.

Import Facebook SDK module at the top of the file.

{% highlight swift %}
import FBSDKCoreKit
{% endhighlight %}

Add the following lines:

{% highlight swift %}
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {

  //1.
  FBSDKApplicationDelegate.sharedInstance().application(application, didFinishLaunchingWithOptions:launchOptions)

  return true
}

func application(application: UIApplication, openURL url: NSURL, sourceApplication: String?, annotation: AnyObject) -> Bool {

  //2.
  let handled = FBSDKApplicationDelegate.sharedInstance().application(application, openURL: url, sourceApplication: sourceApplication, annotation: annotation)

  return handled
}
{% endhighlight %}

What do these 2 lines do?

1. This is when the app first launch, we want to call the `FBSDKApplicationDelegate` to initialize the Facebook SDK.
2. This is after the app switch to Safari (or the Facebook iOS app), finish the authentication and then switch back to our app, it will store the `accessToken` that Facebook passes back to us as a proof of successful authentication.

### Step 4: Call login action in ViewController.

In your `LoginViewController`, when user tap on the `Login With Facebook` button, you will make the call to Facebook:

{% highlight swift %}
@IBAction func onTapLoginWithFacebook(sender: AnyObject) {
  //1.
  let loginManager = FBSDKLoginManager()

  //2.
  let permissions = ["public_profile"]

  //3.
  let handler = { (result: FBSDKLoginManagerLoginResult!, error: NSError?) in
    if let error = error {
      //3.1
      print("error = \(error.localizedDescription)")
    } else if result.isCancelled {
      //3.2
      print("user tapped on Cancel button")
    } else {
      //3.3
      print("authenticate successfully")
      self.goToHomeViewController()
    }
  }

  //4.
  loginManager.logInWithReadPermissions(permissions, fromViewController: self, handler: handler)
}
{% endhighlight %}

This is what the code above does:

1. Instantiate a `loginManager`. This object is used to make the call to authenticate with Facebook.
2. Define a list of permissions to request. For simplicity, we only ask for the public profile of our users.
3. Define a closure to handle callback when the Facebook authentication finished. There are 3 cases to handle here:
  * 3.1: There is an error occured while authenticating. You can look into the `error` variable to see what went wrong.
  * 3.2: User tap `Cancel` while authenticating. Usually you don't want to do anything here.
  * 3.3: The authentication succeeded. We should redirect them to the `Home View Controller`.
4. Actually make the call to authenticate user via the `loginManager` object. We also pass all variables defined above into this method call.

After this step, the users will be brought to Safari (or Facebook iOS app) to review the permissions. If they're happy with it, they would grant our app access to those information.

Then the user will be switched back to our app and into the callback `application:openURL:sourceApplication:annotation` in the `AppDelegate` we just defined above.

### Step 5: Handle autologin.

After you have been successfully authenticated, you are provided an `accessToken`. The next time users open the app, you want to log them in automatically.

Facebook SDK does provide the `FBSDKAccessToken.currentAccessToken()` method which returns the `accessToken` obtained from previous authentication. If it is not nil, we know that we don't need to login anymore.

{% highlight swift %}
class LoginViewController: UIViewController {

  //...

  override func viewDidLoad() {
    super.viewDidLoad()

    if FBSDKAccessToken.currentAccessToken() != nil {
      //we have been authenticated before
      //let's bypass the Login screen for now
      goToHomeViewController()
    }
  }

  //...

}
{% endhighlight %}

### Step 6: Provide a way for user to logout.

This can be done very easily:

{% highlight swift %}
class HomeViewController : UIViewController {

  //...

  @IBAction func onTapLogoutButton(sender: AnyObject) {
    //log user out
    let loginManager = FBSDKLoginManager()
    loginManager.logOut()

    //go back to the Login screen
    self.navigationController?.popViewControllerAnimated(true)
  }

  //...

}
{% endhighlight %}

## Wrap up

The full source code of this demo app can be found at <https://github.com/hoang-tran/TestFacebookLogin>.

Today we went over the process of authenticating our app with Facebook users. We went from creating a `Facebook app` on Facebook developers website to integrate the whole flow into our iOS app. The process is simple in theory yet requires a lot of steps.

On the next post, we will learn how to take advantage of the Facebook `accessToken` once we have it. We're gonna explore how to retrieve the user information and what we can do with it.

Stay tune, guys 😎
