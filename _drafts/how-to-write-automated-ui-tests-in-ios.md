---
layout: post
title:  'How to write automated UI tests in iOS'
categories: ios development
tags: ios ui tests automated kif
---

# Our sample app:

It's a simple note-taking app with the following features:

* Login.
* View list of notes.
* Create new note.
* Update existing note.
* Delete a note.

Here's a gif that runs through all functionalities of the app:

![simple note taking app in ios](/images/how-to-write-automated-ui-tests-in-ios/note-app.gif)

However, our job today is not about how to build such app. What we're gonna do instead is learning how to write automated UI tests for it. That's the exciting part.

# But what does automated UI tests look like anyway?

Let's have a look at this video:

<iframe width="840" height="472" src="https://www.youtube.com/embed/HEGl3Lj4SLE" frameborder="0" allowfullscreen></iframe>

As soon as the test suite is launched, it simulates all kinds of user interactions:

* filling in a text field.
* tapping a button.
* swiping around.

It travels through each screen and test all scenarios in sequence:

* at Login screen, it tests scenario when:
  * entering correct username and password.
  * password field is empty.
  * username field is empty.
  * entering incorrect username or password.
* at Note screen, it tests scenario when:
  * adding a new note.
  * deleting notes.
  * editting an existing note.

**It looks funny**.

Firing up the UI tests and watch it flies through everything is a fun experience.

So the next time someone asks you to show your app, just run the UI tests. He'll be amazed and impressed at the same time. 😜

# Why should we write automated UI tests?

# Understand UI testing:

## Accessibility Label:

### How to set it?

1. From code:

2. From Storyboard:

### How to inspect it?

# What project to practice UI testing?

# Let's do it!

## Step 1: Import KIF using cocoapods

## Step 2: Create KIF helper for Swift

## Step 3: Test the first screen

## Step 4: Move step definitions out into extension

## Step 5: Test the second screen

## Step 6: Move common steps into base class

# Some common scenarios you may encounter:

## Inspect properties of a control:

## Working with TabBar:

## Swiping around:

## Dealing with alert and action sheet:

## Pull to refresh:

## Working with textfields:

## Working with UITableView and UICollectionView:

## Wait for sometime:

## Working with photos and gallery:

## Dealing with network requests:

## Dealing with permissions:

# Wrap up
