---
layout: post
title:  'How to design and unit test your Network Layer in iOS'
categories: ios development
tags: ios network unittest rest api
---

## Step 1: Test the API with Postman

### Test GET request with no params

### Test GET request with params

### Test POST request with params

## Step 2: Design the interface

### The parameters

### The return values when succeeds

### The errors when fails

## Step 3: Implement with your favourite networking framework

### Using NSURLRequest:

### Using AFNetworking:

### Using Alamofire:

## Step 4: Handle errors

### Network errors:

### API errors:

## Step 5: Write unit tests

### The approach:

### Step 5.1: Install pod OHHTTPStubs

### Step 5.2: Create stub response files

### Step 5.3: Actually write unit test

## Step 6: Put it in use

### When first enter the ViewController

### When pull to refresh on TableView

### When hit a button

## Wrap up
