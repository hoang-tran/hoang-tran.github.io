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
* We browse files in Finder.
* We use SourceTree for source version control.

Hardly do we know that all the above tasks can also be accomplished using `Terminal`.

* We can code in Terminal.
* We can browse/create/delete/copy/move files and directories in your Mac system, using Terminal.
* We can do all kinds of git operations in Terminal.

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

## What is Terminal anyway?

It's a place where you write **command** in **a line**.

Each line is called a `command-line` and it has 4 parts:

![terminal look explained](/images/terminal-for-ios-developers/terminal-look-explained.jpg)

1. Your computer name.
2. This tells which directory you are in at the moment. By default, when you first open the `Terminal`, it will bring you to the Home directory.
3. Your current user name for this computer. (because you may have multiple users in the same computer)
4. This is where you write your command. The `$` sign indicates that everything after it is going to be a command.

For most of the time, we will only need to care about 2 and 4, which is **where we are now** and **what command we're gonna run next**.

## Let's run some commands:

First, we go to the *Documents* directory by typing in `cd Documents` and hit `Enter`.

![terminal cd](/images/terminal-for-ios-developers/terminal-test-command-1.jpg)

Another line will appear and wait for our next command. Note that we are now in the *Documents* directory.

Let's see what's inside it. Type `ls` and hit `Enter`.

![terminal ls](/images/terminal-for-ios-developers/terminal-test-command-2.jpg)

After experimenting with some more commands, our terminal will get messy. It's quite hard to read things:

![terminal messy](/images/terminal-for-ios-developers/terminal-full.png)

## Let's make Terminal less ugly

We're gonna install a framework called [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh) (pronounced as *oh my zee shell*).

Type this in the `Terminal` and hit `Enter`.

{% highlight sh %}
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
{% endhighlight %}

It will run for a couple of seconds and then prompt for your password. Just type the password in and hit `Enter`.

![terminal oh my zsh password](/images/terminal-for-ios-developers/terminal-zsh-password.png)

Wait for it to finish the installation process.

![terminal oh my zsh done](/images/terminal-for-ios-developers/terminal-zsh-done.png)

Now with `Oh My Zsh` installed, we have this colorful terminal:

![terminal oh my zsh demo](/images/terminal-for-ios-developers/terminal-zsh-demo.png)

As you can see, the unnecessary informations (computer name, user name, `$` sign) have been stripped away. Directory name now has different color so it's easier to read the terminal content.

## Some basic things you should know before using Terminal:

* The Home directory:
  * It has a symbol of `~`.
  * It is located at */Users/your_user_name*
  * If you open it with `Finder`, it's the one with the Home icon.

![Finder Home directory](/images/terminal-for-ios-developers/home.jpg)

* Maximize the Terminal window for best readability.
* When your Terminal gets messy (it will), `Cmd + K` will clear out the screen.
* Press the `Up` arrow to reuse the previous command, press again to go through the command history.
* Press `Down` arrow will go in the opposite direction.
* When we talk about `path`, we're talking about 2 types of paths here:
  * **absolute path**:
    * is a path that starts from your root directory.
    * begins with `~/` (since we usually work with files from the Home directory so it's easier to say this way)
    * Ex: `~/Documents` , `~/Downloads/Movies/hello.zip` , `~/Documents/Music/US-UK/we-dont-talk-anymore.mp3`
  * **relative path**:
    * is a path that is relative to the current directory.
    * Ex: `Books` , `Movies/hello.zip` , `Documents/Music/US-UK/we-dont-talk-anymore.mp3`

Example: If you want to go to *~/Documents/Books* and you're currently at *~/Documents*, you can use absolute path by running this command `cd ~/Documents/Books`. Or you can use relative path for shorter command `cd Books`.

## Let's learn some commands that we're gonna use very frequently:

### 1. List files:

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

### 2. Change directory:

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

### 3. Print out path to current directory:

The syntax:

{% highlight sh %}
pwd
{% endhighlight %}

`pwd` stands for **p**rint **w**orking **d**irectory. This will print out the absolute path to your current directory, which is where you are in now.

Examples:

{% highlight sh %}
# Let's say you're at ~/Documents/Books
# You run
pwd

# Then this would be printed out
/Users/hoang.tran/Documents/Books
{% endhighlight %}

### 4. Open files and directories:

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

### 5. Create new directory:

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

### 6. Create new file:

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

### 7. Remove files and directories:

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

### 8. Rename files and directories:

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

### 9. Copy/move files and directories:

Nope.

If you're just getting started, you should not do this task in `Terminal`.

Use `open .` to launch `Finder` and do things there instead.

## Putting it all together

Here is my typical iOS development workflow.

Let's say I just created a new project called *MyAwesomeProject* at path *~/Documents/code/MyAwesomeProject*.

Then I open the Terminal and go to that directory.

{% highlight sh %}
cd ~/Documents/code/MyAwesomeProject
{% endhighlight %}

Initialize git for this project.

{% highlight sh %}
git init
{% endhighlight %}

There're some files I don't want to track in git so I create a *.gitignore* file.

{% highlight sh %}
# 1. create an empty file named `.gitignore`
touch .gitignore

# 2. open `.gitignore` file in your default text editor
open .gitignore
{% endhighlight %}

Copy the content from [my gist](https://gist.github.com/hoang-tran/d1908e31282e5c1c99a1f1ec5bf2227d) and paste it in. The *.gitignore* file now looks something like this:

{% highlight sh %}
# OS X
.DS_Store

# Xcode
build/
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata/
*.xccheckout
profile
*.moved-aside
DerivedData
*.hmap
*.ipa

# Bundler
.bundle

Carthage
Pods/
{% endhighlight %}

Save the file.

Initialize cocoapods.

{% highlight sh %}
# 1. Create the Podfile
pod init

# 2. Install pods, the --verbose option will help to print out the installation progress
pod install --verbose
{% endhighlight %}

Run `ls` to check if all the neccessary files are generated. It should print out something like this to the terminal:

{% highlight sh %}
MyAwesomeProject             MyAwesomeProject.xcworkspace Podfile.lock
MyAwesomeProject.xcodeproj   Podfile                      Pods
{% endhighlight %}

Open the *MyAwesomeProject.xcworkspace*.

{% highlight sh %}
open MyAwesomeProject.xcworkspace
{% endhighlight %}

Wait a little for it to launch Xcode. Then press `Cmd + R` to run the project and make sure that everything works fine.

Then I stage all files to git.

{% highlight sh %}
git add .
{% endhighlight %}

Make my first commit.

{% highlight sh %}
git commit -m "initial commit"
{% endhighlight %}

Back to Xcode and start to develop my awesome apps.

As you can see, many of my development tasks are performed in the Terminal. It's extremely fast. You don't even need to reach for the mouse. Your hands are always on the keyboard the whole time.

Here is the list of tasks that I do using GUI tools:

* Write code, design user interfaces in Xcode.
* See diff, stage changes and commit to git using GitX. (or SourceTree)

Tasks I do in terminal:

* Working with files and directories.
* Working with cocoapods.
* All of the remaining git operations: checkout, branch, stash, fetch, pull, push, merge, rebase, revert, reset, cherry-pick, ect...

## Wrap up
