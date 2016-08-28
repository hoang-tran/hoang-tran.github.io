---
layout: post
title:  'A simple guide to Terminal for iOS developers'
categories: ios development
tags: terminal ios developer command line
---

When you read tutorials online, there will come times when they tell you to open up the `Terminal` and type in some commands. Those are the scary moments that may confuse or even stop you from following along. You might wonder why it has to be that complicated.

Can we have any other way that does not involve the use of `Terminal`?

Well, I guess we don't.

As an iOS developer, you may not be comfortable using the `Terminal`. Some people don't even know that such thing does exist at all. We're too familiar with GUI (Graphical User Interface) tools.

* We code in Xcode.
* We use SourceTree for source version control.
* We browse files in Finder.

Hardly do we know that all the above tasks can also be accomplished using `Terminal`.

* We can code in Terminal.
* We can do all kinds of git operations in Terminal.
* We can browse/create/delete/copy/move files and directories in your Mac system, using Terminal.

## How to open Terminal?

1. Open `Finder`
2. Go to *Applications* directory
3. Go to *Utilities*, open `Terminal`.

![how to open terminal on mac](/images/terminal-for-ios-developers/open-terminal.jpg)

You can drag it down to the dock for quick access next time.

![add terminal app to dock](/images/terminal-for-ios-developers/terminal-drag.jpg)

## What does Terminal look like?

It looks like this:

![terminal look](/images/terminal-for-ios-developers/terminal-look.png)

There are 4 parts that we should pay attention to:

![terminal look explained](/images/terminal-for-ios-developers/terminal-look-explained.jpg)

1. Your computer name.
2. This tells which directory you are in at the moment. By default, when you first open the `Terminal`, it will bring you to the Home directory.
3. Your current user name for this computer. (because you may have multiple users in the same computer)
4. This is where you write your command. The `$` sign indicates that everything after it is going to be a command.

## Let's run some commands:

First, we go to the *Documents* directory by typing in `cd Documents` and hit `Enter`.

![terminal cd](/images/terminal-for-ios-developers/terminal-test-command-1.jpg)

Then we wanna see what's inside the *Documents* directory by typing `ls` and hit `Enter`.

![terminal ls](/images/terminal-for-ios-developers/terminal-test-command-2.jpg)

## Common commands for daily use:

### 1. Change directory:

The syntax:

{% highlight sh %}
cd path/to/directory
{% endhighlight %}

Examples:

{% highlight sh %}
# 1.
cd

# 2.
cd ~

# 3.
cd ~/Desktop

# 4.
cd ~/Documents

# 5.
cd ..

# 6.
cd -
{% endhighlight %}

1. If you don't provide any argument to the `cd` command, it will go to Home directory by default.
2. Go to Home directory.
3. Go to *Desktop*.
4. Go to *Documents*.
5. Go to parent directory. If you're at *~/Documents/Books*:
  * `cd ..` will bring you to *~/Documents*.
  * `cd ..` again will lead to home.
6. Go back. Let's say you are at *~/Downloads*:
  * You go to *Desktop* by running `cd ~/Desktop`.
  * Now when you run `cd -`, it will bring you back to *~/Downloads*.
  * `cd -` again will go to *Desktop* again.

### 2. List files:

The syntax:

{% highlight sh %}
ls path/to/directory
{% endhighlight %}

Examples:

{% highlight sh %}
# 1.
ls

# 2.
ls -a

# 3.
ls ~/Documents
{% endhighlight %}

1. List files in the current directory.
2. List files in the current directory, including hidden files.
3. List files in the *Documents* directory.

### 3. Open files and directories:

The syntax:

{% highlight sh %}
open path/to/directory

# or

open path/to/file
{% endhighlight %}

When you use the `open` command, it will choose the most appropriate program to do the task:

* If you open a directory, it will use `Finder`.
* If you open a `txt` file, `TextEdit` will be used. (or `SublimeText`/`Atom` if that's your default text editor)
* If you open a `.xcodeproj` file, it will launch `Xcode`.
* ect...

Examples:

{% highlight sh %}
# 1.
open ~/Documents

# 2.
open .

# 3.
open Podfile

# 4.
open MyAwesomeProject.xcworkspace

# 5.
open -a SomeApplicationNameHere path/to/file

# 6.
open -a Xcode Podfile

# 7.
open -a Sublime\ Text Podfile
{% endhighlight %}

1. Open the *Documents* directory in `Finder`.
2. Open the current directory in `Finder`.
3. Open *Podfile* using `EditText`.
4. Open *MyAwesomeProject.xcworkspace* in `Xcode`.
5. Open file using a specified application.
* If application name contains space characters, you have to escape it by adding a `\ ` character before the space:
  * `Sublime Text` -> `Sublime\ Text`
  * `Google Chrome` -> `Google\ Chrome`
  * ect...
6. Open *Podfile* in `Xcode`.
7. Open *Podfile* in `Sublime Text`.

## 4. Create new directory:

The syntax:

{% highlight sh %}
mkdir directory_name
{% endhighlight %}

Examples:

{% highlight sh %}
# 1.
mkdir Books

# 2.
mkdir my\ awesome\ movies
{% endhighlight %}

1. Create a new directory named `Books` in the current directory. If you're at *~/Documents*, `mkdir Books` will create a directory at path *~/Documents/Books*.
2. Create a new directory named `my awesome movies` in the current directory.

## 5. Create new file:

The syntax:

{% highlight sh %}
touch file_name
{% endhighlight %}

Examples:

{% highlight sh %}
# 1.
touch Podfile

# 2.
touch .gitignore
{% endhighlight %}

1. Create a new file named `Podfile` in the current directory. If *Podfile* already exists, then `touch Podfile` will almost does nothing. (it only changes the modification time and access times of the file, but that's not something you should care for now)
2. Create a new file named `.gitignore` in the current directory.

## 6. Remove files and directories:

The syntax for removing files:

{% highlight sh %}
rm path/to/file
{% endhighlight %}

The syntax for removing directories:

{% highlight sh %}
rm -r path/to/directory
{% endhighlight %}

Note that for directory, we use 1 additional option: `-r`, which will help remove all sub-directories and files inside. Without this option:

* You can only remove empty directory.
* If directory is not empty, you will receive an error that says `rm: directory_name: is a directory`. (where `directory_name` is the directory you want to remove)

Examples:

{% highlight sh %}
# 1.
rm Podfile

# 2.
rm ~/Downloads/abc.zip

# 3.
rm -r build

# 4.
rm -r ~/Documents/Books
{% endhighlight %}

1. Remove file `Podfile` in the current directory.
2. Remove file `abc.zip` at directory *~/Downloads*
3. Remove directory named `build` in the current directory.
4. Remove directory named `Books` at directory *~/Documents*

## 7. Rename files and directories:

The syntax:

{% highlight sh %}
mv old_file_name new_file_name

# or

mv old_directory_name new_directory_name
{% endhighlight %}

Examples:

{% highlight sh %}
# 1.
mv Podfile Podfile2

# 2.
mv .gitignora .gitignore

# 3.
mv Classes Class
{% endhighlight %}

1. Rename file from `Podfile` to `Podfile2`.
2. Rename file from `.gitignora` to `.gitignore` to fix typo.
3. Rename directory from `Classes` to `Class`.

## 8. Copy/move files and directories:

Nope.

You should not do this task in `Terminal`.

Use `open .` to launch `Finder` and do things there instead.

## Wrap up
