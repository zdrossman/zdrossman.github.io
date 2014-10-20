---
layout: post
title: "Integrating Parse and Core Data"
date: 2014-07-10 08:39:20 -0400
comments: true
categories: 
---

I'm using this week's blog post to spur conversation about a topic in class. So if you are a reader who is not in my class, you may want to skip this one. Or comment. That would be great too.

Integration of Parse with Core Data seems to come up fairly often in conversation (what parties do I go to?). 

So, I wanted to hypothesize here a bit on how I might do this. And in a future post, elaborate on how I actually did do it (if different / or more nuanced from the way I describe the process here)

I believe the most straightforward (I did not say most elegant) way to integrate core data and Parse is the following:


First a few prerequisites:

A) Do NOT use PFObject subclassing for your local objects.

B) Implement boilerplate core data code (with or without fetched results controller to your liking.)

C) Create a DataManager object that is a singleton.


Then, get started:

1) Write a category to each NSManagedObject that effectively takes the place of a custom initializer; this custom initializer will take each key/value pair from the PFObject and assign it to the appropriate attribute of the NSManagedObject. Like I said, this isn't the most elegant way of doing things. I have seen NSHipster articles and Github repos that suggest using NSValueTransformer, but I'm not ready to metaprogram my process.

2) Write a method in DataManager that converts each NSManagedObject type back into a PFObject. (There will be one method for each type of NSManagedObject, like in step #1.)

3) Write a method in DataManager that creates a new NSManagedObject based on a PFObject and PFObject's class.

4) Call #3 from your ViewController; and then #1 from here as well.

5) When ready to save to Parse, call #2 from your ViewController.