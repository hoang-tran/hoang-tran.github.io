---
layout: post
title:  'A simple guide to Terminal for iOS developers'
categories: ios development
tags: terminal ios developer command line
---

## Why should you care?

## Common commands for daily use:

### 1. Change directory - `cd` :

{% highlight sh %}
cd path/to/directory
{% endhighlight %}

Go home:

{% highlight sh %}
cd ~

# or just cd

cd
{% endhighlight %}

Go to *Desktop*:

{% highlight sh %}
cd ~/Desktop
{% endhighlight %}

Go to *Documents*:

{% highlight sh %}
cd ~/Documents
{% endhighlight %}

Go to parent directory:

{% highlight sh %}
cd ..
{% endhighlight %}

If you're at *~/Documents/Books*:

* `cd ..` will bring you back to *~/Documents*
* `cd ..` again will lead to home (*~*)

### 2. List files - `ls` :

List all files in the current directory

{% highlight sh %}
ls
{% endhighlight %}

If you're at home (*~*), `ls` will print out something like this:

{% highlight abc %}
➜  ~ ls
Applications       Downloads          Movies
Desktop            Dropbox            Music
Documents          Library            Pictures
{% endhighlight %}

List all files in the current directory, including hidden files:

{% highlight sh %}
ls -a
{% endhighlight %}

You can list files in other directory by providing a path to the `ls` command:

{% highlight sh %}
# list files in ~/Documents
ls ~/Documents
{% endhighlight %}

Or you can use relative path as well:

{% highlight sh %}
# this will list files in ~/Desktop (suppose you're at home)
ls Desktop
{% endhighlight %}

### 3. Open files and directories - `open`:

Open a directory in `Finder`:

{% highlight sh %}
open path/to/directory
{% endhighlight %}

Open directory *~/Documents* in `Finder`:

{% highlight sh %}
open ~/Documents
{% endhighlight %}

Open current directory in `Finder`:

{% highlight sh %}
open .
{% endhighlight %}

Open a file:

{% highlight sh %}
open path/to/file
{% endhighlight %}

Open a *readme.txt* file at *~/Documents*:

{% highlight sh %}
open ~/Documents/readme.txt
{% endhighlight %}

Open an Xcode project:

{% highlight sh %}
open MyAwesomeProject.xcodeproj
{% endhighlight %}

This command works the same as when you double click on the file in `Finder`. It will choose the most appropriate program to open it. For example:

* `.txt` file will be opened using `TextEdit`.
* `.xcodeproj` or `.xcworkspace` will be opened using `Xcode`.
* `.zip` will be handled by `Archive Utilities`.

## Learn to read the manual

## Some contexts for using Terminal

## Wrap up
